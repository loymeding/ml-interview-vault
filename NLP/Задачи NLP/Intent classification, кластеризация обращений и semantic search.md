---
tags: [nlp, intent-classification, text-classification, semantic-search, clustering, embeddings, metrics, production]
тип: теория и практика
уровень: middle-to-senior
сложность: средняя
статус: готово
готовность: 88
создано: 2026-07-09
источники:
  - "Разбор вакансии Ozon Tech: диалоговый ИИ и ML-сервисы"
  - "Текущий чат: подготовка к вопросам по NLP и текстовой классификации"
предпосылки:
  - "Метрики бинарной классификации"
  - "Метрики RAG-систем — retrieval, generation, faithfulness и RAGAS"
связано:
  - "RAG — retrieval, hybrid search, RRF и embeddings"
  - "Sequence Labeling и NER"
  - "Метрики оценки NLP-моделей (Perplexity, BLEU, ROUGE, BERTScore, GLUE)"
---

# Intent classification, кластеризация обращений и semantic search

> [!abstract] Суть
> В диалоговых ML-сервисах текстовая классификация, кластеризация обращений и semantic search решают разные части одной задачи: понять намерение пользователя, найти похожие или релевантные тексты и обнаружить новые темы обращений.

## Ответ

### 1. Intent classification

**Intent classification** — задача определить намерение пользователя:

```text
"Где мой заказ?" -> order_status
"Хочу вернуть товар" -> return_request
"Не пришли деньги" -> refund_status
```

Для чат-ботов это один из базовых компонентов: по intent выбирается сценарий, tool, RAG-источник или эскалация на оператора.

Главные вопросы постановки:

- intents взаимоисключающие или multi-label?
- есть ли fallback / out-of-domain class?
- какие intents рискованные?
- нужна ли иерархия: `возврат -> возврат денег -> срок возврата`;
- как часто появляются новые intents;
- какие ошибки дороже: false positive или false negative.

### 2. Практический pipeline intent classifier

```text
production logs
  -> cleaning + PII removal
  -> annotation guidelines
  -> labeled dataset
  -> train/val/test split by time or users
  -> baseline
  -> stronger model
  -> threshold calibration
  -> error analysis
  -> monitoring
```

Baseline:

```text
TF-IDF + Logistic Regression / Linear SVM / CatBoost
```

Плюсы baseline:

- быстрый;
- дешёвый;
- понятный;
- легко дебажить top features.

Сильнее:

```text
BERT-like encoder + classification head
sentence embeddings + nearest centroid / kNN
LLM few-shot classifier для сложных или редких cases
```

Для массового realtime-классификатора LLM часто дорогая. Её разумно использовать:

- для weak labeling;
- для редких intents;
- как fallback;
- для объяснения ошибок;
- для генерации synthetic examples.

### 3. Метрики при дисбалансе классов

Accuracy почти бесполезна, если один intent занимает 80% трафика.

Смотреть:

- `precision`, `recall`, `F1` по каждому классу;
- `macro F1`, чтобы редкие классы имели вес;
- `weighted F1`, чтобы учитывать частоты;
- confusion matrix;
- coverage при confidence threshold;
- бизнес-стоимость ошибок.

Формулы:

$$\Large Precision = \frac{TP}{TP+FP}$$

$$\Large Recall = \frac{TP}{TP+FN}$$

$$\Large F1 = 2 \cdot \frac{Precision \cdot Recall}{Precision + Recall}$$

$$\Large MacroF1 = \frac{1}{K}\sum_{k=1}^{K}F1_k$$

Если intent критичный, например жалоба или финансовый риск, часто важнее `Recall`: лучше лишний раз отправить на оператора, чем пропустить опасный запрос. Если ошибочная эскалация дорогая, важнее `Precision`.

### 4. Thresholds и fallback

Классификатор не обязан всегда выбирать intent. В production нужен confidence threshold:

```text
if max_proba < threshold:
    ask_clarifying_question()
    or handoff_to_operator()
```

Для multi-class можно делать разные thresholds по intent:

```text
refund_status: threshold = 0.80
order_status: threshold = 0.60
legal_complaint: threshold = 0.90
```

Почему так: цена ошибки по классам разная.

### 5. Error analysis

После обучения важнее всего не "докрутить модель", а понять ошибки.

Разбор:

- confusion matrix: какие intents путаются;
- high-confidence errors: модель уверенно ошибается;
- low-confidence correct: модель знает, но threshold слишком строгий;
- label noise: асессоры размечали неодинаково;
- overlapping intents: классы нужно объединить или сделать иерархию;
- out-of-domain: нужен отдельный fallback;
- temporal drift: появились новые темы обращений.

Типовые действия:

| Симптом | Что делать |
|---|---|
| `return_request` путается с `refund_status` | уточнить guidelines, сделать hierarchy |
| редкий intent имеет низкий recall | собрать данные, class weights, oversampling |
| много OOD попадает в известные intents | добавить rejection/fallback |
| модель уверенно ошибается | искать leakage, неправильные labels, adversarial phrases |
| новые темы в логах | кластеризация обращений + active learning |

### 6. Кластеризация обращений

Кластеризация нужна не для "получить красивый график", а чтобы находить новые темы, боли пользователей и кандидатов для автоматизации.

Pipeline:

```text
texts
  -> embeddings
  -> dimensionality reduction for visualization (UMAP/t-SNE)
  -> clustering (HDBSCAN/KMeans/agglomerative)
  -> cluster labeling
  -> human review
  -> new intents / FAQ / alerts
```

Алгоритмы:

- `KMeans`: если примерно известен `k`, быстрый baseline;
- `HDBSCAN`: хорошо находит кластеры разной формы и шум;
- agglomerative clustering: удобно для иерархий;
- BERTopic-like pipeline: embeddings + clustering + topic labels.

Как интерпретировать кластер:

- ближайшие к центру примеры;
- top n-grams;
- частотность по дням;
- доля негативных обращений;
- доля transfer to operator;
- representative queries.

Нюанс: embedding-кластеры не обязаны совпадать с бизнес-intents. Один бизнес-intent может распасться на несколько языковых кластеров, а один кластер может смешивать разные действия.

### 7. Semantic search

Semantic search ищет не по точным словам, а по смысловой близости embedding.

Схема:

```text
query -> query embedding
documents -> document embeddings
cosine similarity / ANN search
top-k results -> reranker / generator / UI
```

Косинусное сходство:

$$\Large \cos(q,d)=\frac{q \cdot d}{\|q\|\|d\|}$$

Где:
- $q$ — embedding запроса;
- $d$ — embedding документа или chunk.

Semantic search полезен, когда:

- пользователь пишет не теми словами, что в документах;
- много синонимов;
- нужны похожие обращения;
- нужен retrieval для RAG;
- нужно дедуплицировать tickets.

Но он хуже точного поиска, когда:

- важны артикулы, номера заказов, ID;
- запросы короткие и содержат точные коды;
- нужны строгие фильтры по дате/региону/статусу.

Поэтому в production часто используют hybrid search:

```text
BM25 + vector search + reranker
```

### 8. Как оценивать semantic search

Нужен benchmark:

```text
query
relevant_docs/chunks
optional relevance grade
```

Основные метрики:

- `Recall@k`: попали ли нужные документы в top-k;
- `MRR`: насколько высоко первый релевантный результат;
- `nDCG@k`: качество ранжирования с graded relevance;
- `Precision@k`: сколько мусора в top-k;
- `HitRate@k`: есть ли хотя бы один релевантный документ.

Коротко:

$$\Large Recall@k = \frac{\#\text{relevant docs in top-k}}{\#\text{all relevant docs}}$$

$$\Large MRR = \frac{1}{|Q|}\sum_{q \in Q}\frac{1}{rank_q}$$

Для RAG важно оценивать retrieval отдельно от generator:

```text
Если нужный chunk не найден -> проблема retriever/chunking/embedding.
Если найден, но ответ плохой -> проблема prompt/generation/grounding.
```

### 9. Как выбирать embedding model

Выбирать embedding-модель нужно не по leaderboard, а по своему benchmark.

Критерии:

- качество на реальных русскоязычных запросах;
- domain terminology;
- длина документов и query;
- latency;
- размер вектора;
- стоимость индекса;
- возможность локального деплоя;
- устойчивость к опечаткам;
- качество на short queries.

Минимальный эксперимент:

```text
BM25 baseline
open-source multilingual embeddings
domain fine-tuned embeddings
hybrid search
hybrid + reranker
```

Сравнить по `Recall@k`, `MRR`, `nDCG`, latency и стоимости.

## Формула / Схема

Intent classifier:

$$\Large \hat{y}=\arg\max_{c \in C} P(c \mid x)$$

Fallback:

$$\Large \max_{c \in C} P(c \mid x) < \tau \Rightarrow \text{clarify or escalate}$$

Semantic search:

$$\Large score(q,d)=\cos(E(q), E(d))$$

где $E(\cdot)$ — embedding model.

## Короткий пример

Пользователь:

```text
Мне не пришли деньги после возврата
```

Intent classifier:

```json
{
  "intent": "refund_status",
  "confidence": 0.87
}
```

Semantic search ищет документы:

```text
top-1: policy_refund_money_terms
top-2: faq_refund_after_return
top-3: operator_script_refund_delay
```

Если confidence по intent низкий, бот может уточнить:

```text
Вы хотите узнать срок возврата денег или оформить новый возврат?
```

## Типичные ошибки

- **Смотреть только accuracy:** редкие intents исчезают за крупными классами.
- **Не иметь out-of-domain class:** любой мусор принудительно классифицируется в известный intent.
- **Не калибровать thresholds:** модель отвечает уверенно даже там, где надо уточнять.
- **Путать кластеры с intents:** кластеры — исследовательский инструмент, а intents — продуктовая схема действий.
- **Оценивать semantic search только end-to-end ответом LLM:** нужно отдельно понимать, нашёлся ли правильный context.
- **Использовать только dense retrieval для ID/номеров:** точные сущности часто лучше ловит BM25/фильтры.

## Каверзные вопросы

> [!question] Когда лучше классический ML, BERT-like модель или LLM для intent classification?
> Классический ML хорош как быстрый baseline и для простых intents. BERT-like модель лучше, когда важна семантика и разные формулировки. LLM полезна для few-shot, weak labeling и сложных cases, но для массового realtime-классификатора может быть дорогой и менее стабильной.

> [!question] Почему высокий Recall@10 semantic search не гарантирует хороший RAG-ответ?
> Нужный chunk мог попасть в top-10, но оказаться низко, быть зашумлённым или не попасть в итоговый prompt после reranking/compression. Плюс generator может проигнорировать хороший context.

> [!question] Что делать, если два intents постоянно путаются?
> Проверить guidelines и примеры. Возможно, intents пересекаются и нужны hierarchy, объединение классов, уточняющий вопрос или дополнительный feature/tool на следующем шаге.

## Проверка себя

- Почему macro F1 важен при дисбалансе intents?
- Чем semantic search отличается от BM25?
- Когда нужен fallback class?
- Как использовать кластеризацию для поиска новых intents?
- Как отделить ошибку retriever от ошибки generator?

## Предпосылки

- [[Classic Machine Learning/Метрики/Метрики бинарной классификации]]
- [[NLP/LLM и Промпт-инжиниринг/Метрики RAG-систем — retrieval, generation, faithfulness и RAGAS]]

## Связано

- [[NLP/LLM и Промпт-инжиниринг/RAG — retrieval, hybrid search, RRF и embeddings]]
- [[NLP/Модели и архитектуры/Sequence Labeling и NER]]
- [[NLP/LLM и Промпт-инжиниринг/Оценка LLM-ответов — relevance, completeness, factuality и safety]]

## Источники

- Разбор вакансии Ozon Tech: диалоговый ИИ и ML-сервисы.
- Текущий чат: ответы по NLP/classification, semantic search и clustering.

---
[[🗺️ Индекс|Назад к разделу]]
