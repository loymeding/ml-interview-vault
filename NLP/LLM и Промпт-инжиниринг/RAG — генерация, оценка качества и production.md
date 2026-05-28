---
tags: [nlp, llm, rag, generation, evaluation, ragas, mrr, gpu, inference]
тип: теория и практика
уровень: middle+
сложность: средняя
статус: готово
готовность: 90
создано: 2026-05-28
источники:
  - "Sber/Yandex interview notes"
предпосылки:
  - "RAG — retrieval, hybrid search, RRF и embeddings"
связано:
  - "Инференс и производительность DL"
  - "Decoding и sampling в LLM — Temperature, Top-p, Top-k, Beam Search"
сравнить-с: []
---

# RAG — генерация, оценка качества и production

> [!abstract] Суть
> После retrieval начинается не менее важная часть RAG: выбрать генеративную модель, упаковать контекст, не выйти за memory/latency budget и измерить качество не только финального ответа, но и каждого этапа retrieval pipeline.

## Ответ

### 1. Что такое Augmented Generation

В RAG генерация "augmented", потому что LLM отвечает не только из параметрической памяти, но и из внешнего контекста:

```text
user query
  -> retrieved chunks
  -> prompt with context
  -> grounded answer with citations
```

Если база маленькая и контекст короткий, иногда можно положить всю базу знаний прямо в prompt. Это не классический retrieval, а скорее context stuffing / cached context. Подход может быть нормальным для десятков страниц, но плохо масштабируется: растет latency, стоимость, риск потерять нужный факт в длинном контексте и сложность обновления.

### 2. Как выбирать LLM для генерации в RAG

Смотреть нужно на:
- качество ответов на доменных вопросах;
- русский/мультиязычность;
- способность следовать инструкциям и ссылаться на контекст;
- длину контекста;
- latency;
- throughput при параллельных запросах;
- стоимость;
- лицензию;
- поддержку quantization, tensor parallelism, vLLM/TensorRT-LLM/SGLang;
- устойчивость к "не знаю" при недостаточном контексте.

Почему часто выбирали Mistral/Mixtral-подобные модели:
- хорошее качество при умеренном размере;
- open-weight варианты;
- приемлемая latency;
- удобство локального деплоя;
- сильная экосистема fine-tuning/quantization.

Для `1 x A100 80GB` и 10-20 параллельных запросов правильнее выбирать не "самую большую модель", а модель, которая выдерживает latency и KV-cache. Часто практичнее качественная 7B-32B модель или 70B в 4-bit/8-bit с аккуратным batching, чем слишком большая модель, которая съедает всю память и не держит параллелизм.

### 3. Поместится ли 120B на A100 80GB

Грубая память только под веса:

| Формат весов | Память для 120B |
|---|---:|
| FP16/BF16 | около 240 GB |
| INT8 | около 120 GB |
| 4-bit | около 60 GB + overhead |

В 4-bit веса могут "почти поместиться" в 80 GB, но это не означает production-ready inference:
- нужны scale/zero-point/metadata;
- нужен KV-cache;
- нужен runtime overhead;
- нужна память под batch;
- длинный контекст и 10-20 параллельных запросов быстро съедят остаток.

Поэтому 120B на одной A100 80GB — это скорее эксперимент с жесткой квантизацией и коротким контекстом, а не комфортный production для параллельных запросов.

### 4. Что влияет на GPU-память при inference LLM

Основные компоненты:

$$\Large M_{\text{total}} \approx M_{\text{weights}} + M_{\text{KV-cache}} + M_{\text{activations}} + M_{\text{runtime}}$$

где:
- $M_{\text{weights}}$ — память под веса модели;
- $M_{\text{KV-cache}}$ — кэш ключей и значений attention для уже обработанных токенов;
- $M_{\text{activations}}$ — промежуточные активации, на inference обычно меньше, чем при training;
- $M_{\text{runtime}}$ — CUDA kernels, allocator, serving framework, fragmentation.

Для decoder-only модели KV-cache растет с:
- числом слоев;
- числом запросов в батче;
- длиной prompt + generated tokens;
- hidden size / числом KV-heads;
- dtype KV-cache.

Упрощенно:

$$\Large M_{\text{KV}} \propto 2 \cdot L \cdot B \cdot S \cdot H_{\text{kv}} \cdot bytes$$

где:
- $2$ — key и value;
- $L$ — число слоев;
- $B$ — batch/concurrent sequences;
- $S$ — длина контекста;
- $H_{\text{kv}}$ — размерность KV-представлений;
- $bytes$ — байты на число (`fp16` = 2, `int8` = 1 и т.д.).

### 5. Как попытаться уместить большую модель

Варианты:
- weight quantization: 8-bit, 4-bit, GPTQ/AWQ/NF4;
- KV-cache quantization;
- CPU/NVMe offloading;
- tensor parallelism на несколько GPU;
- pipeline parallelism;
- MoE-модель с меньшим active parameter count;
- shorter context;
- smaller batch/concurrency;
- speculative decoding с маленькой draft-моделью;
- paged attention/paged KV-cache;
- выбрать меньшую модель и усилить RAG/reranking.

Главный trade-off: чем сильнее ужимаем модель, тем выше риск потери качества, роста latency или сложности эксплуатации.

### 6. Как итерировать RAG

Итерация обычно идет не "покрутили один параметр", а по слоям:

1. Собрать небольшой evaluation set: вопросы, релевантные документы, ожидаемые ответы.
2. Проверить ingestion: не потерялись ли данные.
3. Проверить chunking: размер, overlap, parent-child.
4. Проверить embedding model.
5. Проверить hybrid search: BM25/vector ratio или RRF.
6. Проверить `top_k`, metadata filters, reranker.
7. Проверить prompt: формат цитат, запрет фантазирования, "не знаю".
8. Проверить decoding: temperature, max tokens.
9. Проверить latency/cost.

### 7. Метрики RAG

Retrieval-метрики:
- `Recall@k`: попал ли релевантный chunk в top-k;
- `Precision@k`: какая доля top-k релевантна;
- `MRR`: насколько высоко первый релевантный результат;
- `nDCG`: учитывает порядок и graded relevance;
- `Hit Rate@k`: есть ли хотя бы один релевантный chunk.

Формула MRR:

$$\Large MRR = \frac{1}{|Q|}\sum_{q \in Q}\frac{1}{rank_q}$$

где:
- $Q$ — набор вопросов;
- $rank_q$ — позиция первого релевантного результата для вопроса $q$;
- если релевантного результата нет, вклад обычно считают равным 0.

Generation-метрики:
- faithfulness/groundedness: ответ следует из контекста;
- answer correctness: ответ совпадает с эталоном;
- context precision: retrieved context не зашумлен;
- context recall: retrieved context содержит нужные факты;
- citation accuracy;
- hallucination rate;
- refusal quality: модель честно говорит "нет данных".

Production-метрики:
- latency p50/p95/p99;
- cost per request;
- tokens in/out;
- retrieval timeout rate;
- empty retrieval rate;
- доля ответов без цитат;
- user feedback;
- drift по темам запросов.

### 8. Если нет Golden Dataset

Варианты:
- собрать seed set из реальных логов и разметить вручную 50-200 вопросов;
- использовать weak labels: клики, выбранные документы, FAQ mapping;
- синтетически сгенерировать вопросы по документам и проверить, что retrieval возвращает исходный документ;
- делать pairwise сравнение двух RAG-конфигураций через human/LLM judge;
- анализировать production feedback;
- отдельно оценивать retrieval на "document -> generated queries".

### 9. Как использовать RAGAS без Golden Dataset

RAGAS-подобный подход можно натянуть частично:
- `faithfulness`: LLM-judge проверяет, выводится ли ответ из контекста;
- `context relevance`: judge оценивает, относится ли контекст к вопросу;
- `answer relevance`: judge оценивает, отвечает ли ответ на вопрос;
- synthetic QA: генерируем вопрос из chunk и знаем source chunk как pseudo-gold.

Ограничение: без настоящего golden set такая оценка проверяет в основном самосогласованность pipeline, а не реальную полезность для бизнеса. Поэтому ее надо калибровать ручной разметкой хотя бы на небольшой выборке.

## Короткий пример

Если RAG отвечает по регламентам банка, сначала полезнее поднять `Recall@10` retrieval и качество reranker, чем сразу менять генеративную LLM. Если нужный пункт регламента не попал в контекст, даже сильная модель будет либо молчать, либо галлюцинировать.

## Типичные ошибки

- Оценивать только финальный ответ и не смотреть, был ли найден нужный chunk.
- Увеличивать `top_k` без reranking и забивать prompt шумом.
- Ставить большую LLM, хотя проблема была в OCR/chunking/retrieval.
- Игнорировать KV-cache при оценке, поместится ли модель на GPU.
- Считать synthetic/RAGAS оценку полноценной заменой golden dataset.

## Проверка себя

- Почему 120B в 4-bit не означает, что модель удобно обслуживать на A100 80GB?
- Чем `MRR` отличается от `Recall@k`?
- Какие параметры RAG стоит тюнить кроме chunk size?
- Как оценивать RAG без эталонных ответов?

## Связано

- [[NLP/LLM и Промпт-инжиниринг/RAG — документы, OCR и чанкинг]]
- [[NLP/LLM и Промпт-инжиниринг/RAG — retrieval, hybrid search, RRF и embeddings]]
- [[Deep Learning/Обучение/Инференс и производительность DL]]

## Источники

- Sber/Yandex interview notes

---
[[🗺️ Индекс|Назад к разделу]]
