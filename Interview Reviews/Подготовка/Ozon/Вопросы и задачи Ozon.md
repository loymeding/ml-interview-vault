---
tags: [interview-prep, ozon, rag, llm, nlp, ml, questions]
company: Ozon
type: raw-questions-and-tasks
status: active
created: 2026-07-13
questions_total: 77
tasks_total: 0
cssclasses: [wide-page]
---

# 🧾 Ozon - вопросы и задачи

> [!success] Готовые ответы
> [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML|Развёрнутые ответы по RAG, LLM, ML, агентам и оценке]] закрывают все вопросы ниже. Здесь остаются исходный список и будущие практические задачи.

## 🧷 Карта вопросов и ответов

- 1-17. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#1. 🧩 RAG: от поиска к честному ответу|RAG, retrieval, golden set, hallucinations, PII и structured output]]
- 18-30. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#2. 📏 Классификация и clustering: метрика следует за решением|Классификация, F1/F-beta, ROC-AUC и clustering]]
- 31-39. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#3. 🔎 Векторизация и embeddings|Векторизация, Word2Vec/FastText, similarity и triplet loss]]
- 40-49. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#4. 🧠 Transformer и decoding LLM|Transformer, RoPE, BERT/GPT/T5 и decoding]]
- 50-59. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#5. ⚙️ Fine-tuning и быстрый inference|LoRA, QLoRA, quantization, KV-cache и speculative decoding]]
- 60-66. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#6. 🤖 Агенты: когда нужен выбор действий|Агенты, ReAct, workflow, memory и выбор LLM]]
- 67-73. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#7. 🧪 Ассессоры, agreement и LLM-as-a-Judge|Внешние ассессоры, agreement, aggregation и LLM-as-a-Judge]]
- 74-77. [[Interview Reviews/Подготовка/Ozon/Ozon — развёрнутые ответы RAG, LLM и ML#8. 🏗️ Инфраструктурный блиц: как связать компоненты|Redis, БД, Kafka, gRPC, serving и личный опыт]]

> [!tip] Как собран банк
> Два исходных списка объединены и очищены от прямых повторов. Вопросы специально оставлены прикладными: отвечая, связывай определение с решением в RAG/ML-сервисе, метрикой и ограничениями production.

---

## 🧭 Карта блоков

| Блок | Темы | Приоритет |
|---|---|---|
| 1. RAG | retrieval, генерация, PII, hallucinations, guardrails | 🔥 |
| 2. Классификация и clustering | loss, метрики, F1, ROC-AUC, k-means, DBSCAN | 🔥 |
| 3. Embeddings | BoW, TF-IDF, Word2Vec, FastText, triplet loss | 🔥 |
| 4. Transformers и decoding | attention, RoPE, T5/BERT/GPT, sampling | 🔥 |
| 5. Fine-tuning и inference | LoRA, QLoRA, quantization, KV-cache, ONNX | 🔥 |
| 6. Агенты | ReAct, workflow, memory, model choice | 🔥 |
| 7. Evaluation и ассессоры | golden data, agreement, LLM-as-a-Judge | 🔥 |
| 8. Инфраструктурный блиц | Redis, Kafka, vLLM, Triton, multi-GPU | 🟡 |

---

## 1. 🧩 RAG: архитектура, качество и безопасность

1. Что такое RAG, из каких компонентов состоит архитектура и на каком этапе появляется генерация с контекстом?
2. Как выбрать embedding-модель для RAG: какие offline-метрики, latency, языки, размерность и лицензии учитывать?
3. Как выбрать генеративную LLM для RAG: качество, context window, tool calling, latency, цена, deployment и safety?
4. Как построить golden dataset для retrieval и почему его нельзя собирать только из простых вопросов?
5. Как оценивать retrieval: Recall@k, Precision@k, MRR, nDCG? Что каждая метрика говорит о проблеме?
6. Чем MRR отличается от Precision по смыслу и когда MRR может быть высоким при плохом coverage?
7. Как оценивать генерацию RAG отдельно от retrieval: faithfulness, answer relevance, completeness, citation correctness и human evaluation?
8. Какие способы улучшения RAG знаешь: данные, chunking, hybrid search, reranking, query transformation, prompting и model selection?
9. Какие метрики указывают, какой именно компонент RAG нужно исправлять?
10. Как предобрабатывать запрос пользователя: rewrite, multi-query, HyDE, decomposition, filters и metadata extraction?
11. Что такое BM25 и чем lexical search отличается от семантического vector search?
12. Как объединять BM25 и vector retrieval: weighted score, RRF, reranking? Когда выбрать каждый способ?
13. Пользователи жалуются на hallucinations RAG. Как провести диагностику причин и какой порядок исправлений выбрать?
14. Как добиться, чтобы RAG не отвечал на запрещённые темы/слова, не раскрывал PII и не обходил policy через документы?
15. Как шифровать или иначе защищать PII, если запрос нужно передать во внешнюю LLM?
16. Что такое few-shot в RAG: куда передаются примеры и чем это отличается от дообучения?
17. Что такое structured output и как добиться валидного JSON/схемы в RAG-пайплайне?

---

## 2. 📏 Классификация и кластеризация

18. В чём суть задачи классификации и какие алгоритмы подходят для бинарного, мультиклассового и multilabel случая?
19. Какие loss-функции используются в классификации: binary cross-entropy, categorical cross-entropy, focal loss? Когда они нужны?
20. Какие метрики классификации знаешь: Accuracy, Precision, Recall, F1, ROC-AUC, PR-AUC, LogLoss? Как выбрать среди них?
21. Почему F1 использует гармоническое среднее Precision и Recall, а не арифметическое?
22. Как через F-beta учесть, что Precision важнее Recall? Каким должен быть $\beta$?
23. Нужно предсказывать торнадо: что важнее, Precision или Recall, и почему ответ зависит от бизнес-действия после прогноза?
24. Три классификатора имеют ROC-AUC 0.1, 0.5 и 0.7. Как интерпретировать их и какой выбрать?
25. Как агрегировать метрики в мультиклассовой классификации: micro, macro, weighted average? Что выберешь при редких классах?
26. Что такое кластеризация и чем она отличается от классификации?
27. Какие методы кластеризации знаешь: k-means, hierarchical, DBSCAN/HDBSCAN, GMM, spectral clustering? В чём их предположения?
28. Сравни k-means и DBSCAN: сильные/слабые стороны и критерии выбора.
29. Как выбрать число кластеров в k-means: elbow, silhouette, stability, business interpretability?
30. Как оценивать кластеризацию без разметки и с разметкой: silhouette, Davies-Bouldin, inertia, ARI, NMI?

---

## 3. 🔎 Векторизация и embeddings

31. Назови методы векторизации текста от простых к современным: one-hot, BoW, TF-IDF, static embeddings, contextual embeddings, sentence embeddings.
32. Как устроен Bag-of-Words и в чём его основные ограничения?
33. Чем TF-IDF улучшает BoW и что означает каждая часть формулы?
34. Как работает Word2Vec: что предсказывают CBOW и Skip-gram, какой objective используется?
35. Чем FastText отличается от Word2Vec и почему он устойчивее к редким словам и опечаткам?
36. Как измерять близость векторов: cosine similarity, dot product, Euclidean/Manhattan distance? Когда какая мера уместна?
37. Какие значения принимает cosine similarity и что означают 1, 0 и -1?
38. Как обучают sentence embeddings для retrieval: contrastive loss, in-batch negatives, hard negatives?
39. Что такое triplet loss, как устроены anchor/positive/negative и зачем нужен margin?

---

## 4. 🧠 Transformer и генерация LLM

40. Расскажи архитектуру Transformer: encoder, decoder, self-attention, cross-attention, feed-forward, residual connection и LayerNorm.
41. Что такое Q, K и V, как вычисляется scaled dot-product attention и почему делят на $\sqrt{d_k}$?
42. Какие виды attention бывают: self, cross, causal/masked, multi-head? Когда используется каждый?
43. Какая вычислительная и memory-сложность полного attention по длине последовательности?
44. Зачем Transformer учитывает позиции токенов и какие бывают positional encodings: learned/sinusoidal, relative, RoPE, ALiBi?
45. Расскажи подробнее про RoPE: что вращается, почему attention зависит от относительного сдвига и как растягивают контекст?
46. В чём архитектурная и практическая разница между BERT, GPT и T5?
47. Какие параметры генерации LLM знаешь и как они меняют качество, разнообразие, latency и safety ответа?
48. Как температура изменяет softmax-распределение? Какие практические диапазоны и риски очень малой/большой температуры?
49. Что такое top-k, top-p/nucleus sampling и beam search? Почему beam search редко выбирают для чатовых LLM?

---

## 5. ⚙️ Дообучение и ускорение inference

50. Какие методы адаптации LLM знаешь: full fine-tuning, adapters, prompt/prefix tuning, LoRA, QLoRA? Как выбрать?
51. Как устроена LoRA: что означает низкоранговая дельта весов и почему она экономит память?
52. Как LoRA ведёт себя на inference: что значит merge adapter с базовыми весами и когда merge не делают?
53. Как выбирать rank $r$, scaling $\alpha$, dropout и target modules LoRA?
54. Чем QLoRA отличается от LoRA: что квантуется, что остаётся обучаемым и зачем нужны NF4/double quantization?
55. Что такое квантование: PTQ/QAT, static/dynamic, int8/int4, weight-only/activation? Как проверять качество после него?
56. Что такое knowledge distillation и почему «мягкие» вероятности teacher дают student больше информации, чем hard labels?
57. Что такое KV-cache, какую работу он экономит и почему он становится memory bottleneck?
58. Что такое speculative decoding и при каких условиях он реально ускоряет генерацию?
59. Какие ещё способы ускорения inference знаешь: continuous batching, paged attention, FlashAttention, prefix caching, kernel fusion, pruning, ONNX/TensorRT/vLLM/Triton?

---

## 6. 🤖 Агенты и workflow

60. Что такое LLM-агент: из каких частей состоит и как он принимает решение о следующем действии?
61. Что такое ReAct и почему чередование reasoning/action/observation полезно?
62. Чем workflow отличается от агентной системы и когда детерминированный workflow лучше?
63. Какие архитектуры агентов знаешь кроме ReAct: Plan-and-Execute, router, supervisor, multi-agent, reflection? Какие у них риски?
64. Можно ли реализовать агента без LangGraph? Что фреймворк даёт, а что остаётся ответственностью инженера?
65. Как выбрать LLM для агентов: сложность задач, качество tool calling, structured output, context, latency, стоимость, безопасность и open/closed weights?
66. Какая память бывает у агентов: short-term context, episodic/long-term memory, vector store, state store? Как избежать накопления нерелевантного контекста?

---

## 7. 🧪 Оценка, ассессоры и LLM-as-a-Judge

67. Как подготовить выборку диалогов для оценки классификатора внешними ассессорами: sampling, стратификация, PII, инструкция и quality control?
68. Есть бинарная разметка с перекрытием 3 и метаданные ассессоров. Как оценить качество разметки и обнаружить систематические проблемы?
69. Как агрегировать разметку трёх ассессоров: majority vote, weighted vote, Dawid-Skene? Когда простого большинства недостаточно?
70. Как измерить inter-annotator agreement: Cohen's/Fleiss' kappa, Krippendorff's alpha? Почему agreement нельзя трактовать без base rate?
71. Что такое LLM-as-a-Judge: основной принцип, сильные и слабые стороны?
72. Как доказать, что LLM-as-a-Judge можно доверять в конкретном сценарии, и почему она не отменяет human evaluation?
73. Как prompt, rubric, порядок вариантов и bias судьи влияют на оценку? Как калибровать судью под людей?

---

## 8. 🏗️ Инфраструктурный блиц и практический опыт

74. Для чего в ML/LLM-системе могут понадобиться Redis, Postgres и ClickHouse? Как бы ты распределил между ними кэш, транзакционные данные, логи и аналитику?
75. Как Kafka обеспечивает доставку сообщений и зачем она в data/LLM pipeline? Какие особенности at-least-once нужно учесть?
76. Что такое gRPC и почему он может быть удобнее REST для внутренних высоконагруженных сервисов?
77. Что стоит уверенно рассказать про Docker, Git/Git Flow, Triton Inference Server, vLLM, ONNX, fine-tuning BERT/LLM, multi-GPU training и постановку ТЗ ассессорам?

---

## ✅ Следующий шаг

> [!success] Приоритет ответов
> Сначала подготовить ответы к RAG (1-17), LLM/inference (40-59) и ассессорам (67-73): именно там в источнике было больше всего практических уточнений и кейсов.
