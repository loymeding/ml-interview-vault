---
tags: [nlp, llm, embeddings, retrieval, reranking, mteb, eos, contrastive-learning, infonce]
тип: теория и практика
уровень: senior
сложность: высокая
статус: готово
готовность: 85
создано: 2026-05-28
источники:
  - "Вопросы пользователя по LLM-собеседованию"
предпосылки:
  - "Fine-tuning и PEFT в LLM — BERT, LoRA, QLoRA"
связано:
  - "Продвинутые Seq2Seq (T5, BART) и RAG-системы"
  - "Метрики оценки NLP-моделей (Perplexity, BLEU, ROUGE, BERTScore, GLUE)"
сравнить-с:
  - "Transfer Learning в NLP и семейство BERT"
---

# Эмбеддинги decoder-only LLM — EOS, Anchor Embeddings и MTEB

> [!abstract] Суть
> Decoder-only LLM хорошо генерируют текст, но их стандартный `[EOS]`-эмбеддинг не обязан быть хорошим sentence embedding. Для retrieval и semantic search его нужно специально обучать: например, через реконструкцию query-document пар и contrastive learning.

## Ответ

### 1. Почему `[EOS]` не гарантирует хороший эмбеддинг

В decoder-only LLM токен `[EOS]` появляется в конце последовательности. Кажется, что он "прочитал" весь текст и может быть представлением всего текста. Но во время pretraining `[EOS]` обычно учится прежде всего сигнализировать конец последовательности.

Проблемы:
- `[EOS]` не обучался явно сжимать семантику всего текста;
- next-token prediction не равен задаче retrieval;
- запрос и релевантный документ могут быть лексически разными, но семантически близкими;
- embedding из генеративной модели может отражать стиль и продолжение, а не поисковую релевантность.

Поэтому для MTEB/retrieval decoder-only модель обычно дообучают.

### 2. Anchor Embeddings

Идея anchor embeddings: сделать финальный токен семантическим якорем, куда модель должна сжать смысл текста так, чтобы он был полезен для сопоставления query и document.

Вместо того чтобы просто брать `[EOS]` как есть, модель обучают задачам, которые заставляют этот вектор хранить смысловую связь.

### 3. Bidirectional Reconstruction

Используются две симметричные задачи.

**EBQ2D (Embedding-Based Query-to-Document):**
1. модель получает query;
2. сжимает query в embedding `[EOS]`;
3. по этому embedding через decoder должна восстановить релевантный document.

Смысл: если embedding query плохой, по нему нельзя восстановить документ.

**EBD2Q (Embedding-Based Document-to-Query):**
1. модель получает document;
2. сжимает document в embedding;
3. должна восстановить query, которому этот документ отвечает.

Симметрия важна: модель учится понимать связь "что спрашивают" и "какой документ отвечает".

### 4. Contrastive learning

После реконструкции эмбеддинги дообучают через contrastive learning: похожие пары сближаются, непохожие отталкиваются.

Типичная функция — InfoNCE:

$$\Large L_i = -\log \frac{\exp(\text{sim}(q_i, d_i^+)/\tau)}{\exp(\text{sim}(q_i, d_i^+)/\tau) + \sum_j \exp(\text{sim}(q_i, d_j^-)/\tau)}$$

где:
- $q_i$ — embedding запроса;
- $d_i^+$ — embedding релевантного документа;
- $d_j^-$ — embeddings нерелевантных документов;
- $\text{sim}$ — cosine similarity или dot product;
- $\tau$ — temperature contrastive loss.

### 5. Почему это полезно для retrieval

Retrieval требует, чтобы близкими были не просто похожие строки, а смысловые пары:

```text
query: "как ускорить инференс LLM"
document: "KV-cache, batching, quantization and paged attention reduce latency"
```

Лексическое пересечение может быть слабым, но семантическая связь сильная. Специальное embedding-обучение под query-document matching помогает этому.

### 6. MTEB

MTEB — набор задач для оценки текстовых эмбеддингов: retrieval, reranking, clustering, classification, semantic textual similarity. Высокий score на MTEB означает, что embeddings полезны не только для одной задачи, а для широкого набора сценариев.

Важно: MTEB — не гарантия качества в конкретном проде. Для корпоративного поиска нужно валидировать на своих запросах, документах и негативных примерах.

## Формула / Схема

**Два этапа обучения anchor embeddings:**

```text
Stage I: Bidirectional Reconstruction
query -> EOS embedding -> generate document
document -> EOS embedding -> generate query

Stage II: Contrastive Learning
positive pairs closer, negatives farther
```

## Короткий пример

Для RAG-системы эмбеддинги должны сближать вопрос пользователя и документ, который содержит ответ, даже если они написаны разными словами. Поэтому "просто взять EOS из LLaMA" обычно слабее, чем модель, обученная на query-document contrastive objective.

## Типичные ошибки

- **Использовать generative LLM как embedding model без проверки:** хороший генератор не обязательно хороший retriever.
- **Оценивать embeddings только на cosine examples вручную:** нужен retrieval eval с hard negatives.
- **Считать MTEB финальной истиной:** бенчмарк полезен, но доменный поиск может вести себя иначе.
- **Путать reranking и retrieval:** retrieval быстро достаёт кандидатов, reranker дороже и точнее переупорядочивает top-k.

## Каверзные вопросы

> [!question] Почему bidirectional reconstruction помогает сильнее, чем просто contrastive learning с нуля?
> Реконструкция заставляет embedding хранить информацию, достаточную для восстановления парного текста. Contrastive stage потом полирует уже семантически насыщенное пространство.

> [!question] Почему decoder-only embedding часто берут из последнего токена?
> В causal attention последний токен имеет доступ ко всем предыдущим токенам. Но доступ к контексту не означает, что он обучен быть хорошим sentence embedding.

## Проверка себя

- Почему `[EOS]` может быть плохим embedding-агрегатором?
- Чем retrieval отличается от reranking?
- Зачем нужны hard negatives в contrastive learning?

## Предпосылки

- [[NLP/LLM и Промпт-инжиниринг/Fine-tuning и PEFT в LLM — BERT, LoRA, QLoRA]]

## Связано

- [[NLP/LLM и Промпт-инжиниринг/Продвинутые Seq2Seq (T5, BART) и RAG-системы]]
- [[NLP/LLM и Промпт-инжиниринг/Метрики оценки NLP-моделей (Perplexity, BLEU, ROUGE, BERTScore, GLUE)]]

## Источники

- Вопросы пользователя по LLM-собеседованию

---
[[🗺️ Индекс|Назад к разделу]]
