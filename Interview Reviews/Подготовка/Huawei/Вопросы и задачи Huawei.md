---
tags: [interview-prep, huawei, multimodal, vision-language-speech, questions]
company: Huawei
type: raw-questions-and-tasks
status: active
created: 2026-07-13
questions_total: 83
tasks_total: 10
cssclasses: [wide-page]
---

# 🧾 Huawei CBG AI - вопросы и задачи

> [!success] Готовые ответы
> [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI|Развёрнутые ответы VLM, Document AI, Speech и Edge AI]] закрывают все вопросы ниже. Здесь остаются исходный список и будущие практические задачи.

> [!tip] Практика
> Подготовлены 6 задач live coding и 4 задачи code review по Document AI, speech, VLM и edge: [[Interview Reviews/Подготовка/Huawei/Лайвкодинг и code review Huawei|Huawei — лайвкодинг и code review]].

## 🧷 Карта вопросов и ответов

- 1-12. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#1. 🧠 VLM и Omni-модели: как соединить зрение, речь и язык|VLM, CLIP, ViT, connectors и alignment]]
- 13-22. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#2. 👁️ Document AI, OCR, handwriting и SAM|Document AI, OCR, layout, handwriting и SAM]]
- 23-32. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#3. 🎙️ Whisper, Wav2Vec 2.0, TTS и video/audio QA|Whisper, Wav2Vec 2.0, TTS и video/audio QA]]
- 33-42. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#4. 🧪 Мультимодальные данные: pipeline, синхронизация и privacy|Мультимодальные данные, pipeline, синхронизация и privacy]]
- 43-52. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#5. 🔬 Fine-tuning, robustness и честная оценка|Fine-tuning, evaluation, robustness и safety]]
- 53-66. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#6. ⚙️ Edge optimization: quantization, pruning, distillation и ONNX|Edge optimization, quantization, pruning, distillation и ONNX]]
- 67-74. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#7. 🏗️ Cloud/Edge, SDK, rollout и observability|Cloud/Edge, SDK, rollout и observability]]
- 75-77. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#8. 🧭 Research и инженерная зрелость|Выбор foundation model и research-подход]]
- 78. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#10. Как персонализировать рассказ о невыкаченной модели|Шаблон личного production-кейса]]
- 79-83. [[Interview Reviews/Подготовка/Huawei/Huawei — развёрнутые ответы VLM и Edge AI#9. 🧩 System design: как рассуждать о практической задаче|System design и разбор ограничений устройства]]

> [!tip] Как пользоваться
> Это не абстрактный список по deep learning, а вопросы, выведенные из конкретной вакансии. У каждого блока есть прикладной контекст: AI-ассистент, документы, аудио/видео и ограниченные ресурсы consumer devices.

---

## 🧭 Карта блоков

| Блок | Суть | Приоритет |
|---|---|---|
| 1. VLM и Omni-модели | как объединить изображение, текст и речь | 🔥 |
| 2. Computer Vision и документы | OCR, layout, handwriting, segment-anything | 🔥 |
| 3. Speech и audio/video | ASR, TTS, Wav2Vec, синхронизация | 🔥 |
| 4. Мультимодальные данные | качество, разметка, пайплайны | 🔥 |
| 5. Адаптация и evaluation | fine-tuning, hallucinations, бенчмарки | 🔥 |
| 6. Edge optimization | latency, память, quantization, ONNX | 🔥 |
| 7. Production и SDK | Cloud/Edge, API, наблюдаемость | 🟡 |
| 8. Инженерный и research-подход | выбор модели, эксперименты, опыт | 🟡 |
| 9. System design / coding | проектирование реальных сценариев | 🔥 |

---

## 1. 🧠 VLM, LLM и Omni-модели

1. Что называют мультимодальной или Omni-моделью и чем она отличается от text-only LLM?
2. Как типичная VLM соединяет image encoder и языковую модель: какие роли у vision encoder, projector/adapter и LLM?
3. Почему нельзя просто передать пиксели в LLM? Что именно должен извлечь vision encoder?
4. Как устроен ViT и почему изображения разбивают на patches? Как размер patch влияет на качество и compute?
5. Что такое CLIP и какую задачу решает контрастивное обучение image-text пар?
6. Как с помощью CLIP получать zero-shot классификацию изображений?
7. Что такое cross-attention и чем он отличается от self-attention в мультимодальной модели?
8. Какие варианты коннектора между vision encoder и LLM знаешь: linear projector, MLP, Q-Former, cross-attention? Когда выбрать каждый?
9. Чем архитектурно и practically отличаются encoder-decoder, decoder-only VLM и unified omni-model?
10. Как обучать VLM: pre-training, alignment, instruction tuning? Какие данные нужны на каждом этапе?
11. Почему VLM может галлюцинировать объекты на изображении? Какие причины на стороне данных, модели и decoding?
12. Как сравнивать InternVL, Qwen-VL, MiniCPM-o и DeepSeek-VL для продуктовой задачи, а не по одному benchmark score?

---

## 2. 👁️ Computer Vision, document understanding и handwriting

13. Как спроектировать систему document QA для PDF/фото документа: какие стадии будут от изображения до ответа?
14. Когда достаточно OCR + text RAG, а когда нужна end-to-end VLM?
15. Какие типичные ошибки возникают у OCR на мобильной фотографии документа и как их уменьшить до распознавания?
16. Как оценивать качество OCR: CER, WER, field-level accuracy? Когда какая метрика важнее?
17. Как извлечь структуру документа: заголовки, абзацы, таблицы, подписи, порядок чтения?
18. Почему таблицы и multi-column layout особенно сложны для document understanding?
19. Как бы ты решал распознавание рукописного текста? Какие данные и метрики понадобятся?
20. Что делает SAM и для каких задач сегментации он подходит? Где SAM не заменяет специализированную модель?
21. Чем ConvNeXt концептуально отличается от классической CNN и ViT? Когда CNN всё ещё разумнее ViT?
22. Как подготовить document benchmark, чтобы модель не выучила шаблоны документов из train?

---

## 3. 🎙️ Speech, TTS и video/audio understanding

23. Как работает Whisper на высоком уровне: входные представления, encoder-decoder и декодирование текста?
24. Чем Wav2Vec 2.0 отличается от supervised ASR и зачем ему self-supervised pre-training?
25. Как выбирать между Whisper, Wav2Vec 2.0 и небольшой task-specific ASR-моделью для on-device сценария?
26. Какие метрики нужны для ASR: WER, CER, latency, real-time factor? Как интерпретировать WER?
27. Какие факторы сильнее всего ухудшают распознавание речи на устройстве: шум, акцент, far-field, overlap, язык, код-свитчинг?
28. Что такое voice activity detection и зачем она нужна до ASR?
29. Как устроен TTS-пайплайн и какие характеристики определяют его качество кроме естественности голоса?
30. Как оценивать TTS: MOS, intelligibility, speaker similarity, latency? Почему одной MOS недостаточно?
31. Как синхронизировать аудио, видео, субтитры и текстовую аннотацию в мультимодальном датасете?
32. Как бы ты построил поиск и QA по видео: кадры, ASR-транскрипт, temporal chunks, embeddings, reranking?

---

## 4. 🧪 Мультимодальные датасеты и data pipelines

33. Как построить pipeline сбора мультимодальных данных из изображений, PDF, аудио и видео?
34. Какие метаданные обязательно хранить для каждого объекта и как обеспечить его воспроизводимость?
35. Как проверять синхронизацию image-text, audio-text и video-audio пар? Какие виды рассинхронизации бывают?
36. Как обнаруживать дубликаты и почти-дубликаты изображений, аудио и текстов?
37. Как устроить разбиение train/validation/test, чтобы не было leakage между версиями одного документа, видео или пользователем?
38. Как организовать human annotation для image-caption, document QA или audio-comment задач и как измерить согласие разметчиков?
39. Что делать с шумными, противоречивыми и токсичными мультимодальными примерами?
40. Как решать проблему дисбаланса по языкам, устройствам, документам и редким классам?
41. Почему licensing, consent и персональные данные особенно важны для consumer-device мультимодальных данных?
42. Как масштабировать preprocessing и хранение мультимодальных данных в distributed pipeline?

---

## 5. 🔬 Fine-tuning, alignment и evaluation

43. Когда нужен full fine-tuning, а когда LoRA/QLoRA или adapter tuning для VLM?
44. Какие слои VLM ты бы адаптировал в первую очередь при доменном сдвиге в документах или речи?
45. Как избежать catastrophic forgetting при дообучении foundation model на узком домене?
46. Как составить offline evaluation для document QA: retrieval, visual grounding, корректность ответа, citation/faithfulness?
47. Как оценить модель видео- или аудиоанализа, если правильный ответ зависит от времени и контекста?
48. Какие бенчмарки и срезы данных использовал бы для VLM, OCR, ASR и TTS? Как не переоптимизироваться под benchmark?
49. Что такое ablation study и какие абляции полезны в проекте с VLM?
50. Как отличить реальный прогресс модели от шума эксперимента?
51. Какие safety-риски есть у мультимодального ассистента: privacy, hallucination, harmful content, prompt injection через изображение/документ?
52. Как проектировать human evaluation так, чтобы она была воспроизводимой и полезной для решения о релизе?

---

## 6. ⚙️ Edge inference, оптимизация и ONNX Runtime

53. Из чего складывается latency VLM на смартфоне: preprocessing, vision encoder, prefill, decode, memory transfer, postprocessing?
54. Чем throughput отличается от latency и почему для интерактивного ассистента p95/p99 часто важнее среднего?
55. Какие виды квантования знаешь: dynamic/static, PTQ/QAT, int8/int4, weight-only и activation quantization?
56. Как выбрать между PTQ и QAT для модели, которая должна работать на edge-устройстве?
57. Почему int4-квантование может заметно ухудшить VLM/ASR, хотя int8 почти не меняет качество? Как это проверять?
58. Какие варианты pruning существуют: unstructured, structured, head/channel pruning? Почему structured pruning обычно полезнее для реального ускорения?
59. Что такое knowledge distillation и как использовать teacher VLM для обучения маленькой student-модели?
60. Что даёт экспорт в ONNX и какую роль выполняет ONNX Runtime?
61. Какие проблемы возникают при экспорте Transformer/VLM в ONNX: dynamic shapes, unsupported ops, KV-cache, numerical parity?
62. Как профилировать модель на реальном мобильном устройстве, а не только на GPU-сервере?
63. Как сравнить CPU, GPU и NPU/NNAPI backend для конкретной модели?
64. Как снизить энергопотребление и нагрев устройства, не оптимизируя только latency?
65. Что такое memory bandwidth bottleneck и почему FLOPS не всегда предсказывают скорость inference?
66. Какие компромиссы между размером модели, контекстом, quality, latency, RAM и battery ты бы заложил в product requirements?

---

## 7. 🏗️ Production, Cloud/Edge и Mobile SDK

67. Как разделить обработку между устройством и облаком для AI-ассистента, учитывая privacy, задержку и качество?
68. Какой fallback-путь нужен, если локальная модель не уверена, устройство перегрето или нет сети?
69. Как оформить API/контракт между Mobile SDK, inference engine и backend?
70. Какие части мультимодального пайплайна разумно писать на Python, а какие - на C++?
71. Как версионировать модель, tokenizer, processor и preprocessing, чтобы обновление SDK не ломало inference?
72. Какие метрики и логи нужны после релиза мультимодальной функции на устройства?
73. Как мониторить деградацию качества, если большую часть пользовательских данных нельзя собирать из-за privacy?
74. Как организовать постепенный rollout новой модели и быстрый rollback?

---

## 8. 🧭 Инженерный и research-подход

75. Как бы ты выбрал foundation model для нового document assistant: какие критерии проверишь до fine-tuning?
76. Как спланировать эксперимент, чтобы за две недели доказать ценность VLM для продукта?
77. Какие результаты и артефакты должен оставить после себя исследовательский эксперимент, чтобы команда могла его воспроизвести?
78. Расскажи о случае, когда качество модели выросло, но решение всё равно нельзя было выкатывать в production. Какие ограничения могли остановить запуск?
79. Как объяснить product manager разницу между offline benchmark score и пользовательской ценностью?

---

## 9. 🧩 System design и практические задачи

80. Спроектируй on-device ассистента, который отвечает на вопросы по сфотографированному договору: архитектура, модели, кэш, privacy и метрики.
81. Спроектируй video/audio summarization для смартфона с ограничением на latency, память и battery.
82. Как бы ты построил pipeline обновления мультимодального датасета и дообучения модели без остановки production?
83. Есть VLM, которая хорошо работает на A100, но не помещается в mobile memory budget. Как последовательно найти и устранить проблему?

---

## ✅ Следующий шаг

> [!success] Очередь ответов
> Сначала стоит подготовить ответы к блокам 1, 3, 4 и 6: они почти буквально повторяют требования вакансии. Практический слой уже вынесен в [[Interview Reviews/Подготовка/Huawei/Лайвкодинг и code review Huawei|Huawei — лайвкодинг и code review]]; после появления тестового задания его можно дополнить реальными задачами.
