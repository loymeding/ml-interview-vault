---
topic: Сжатие языковых моделей (Distillation, Quantization, Pruning, LoRA)
card_ref: "[[Сжатие языковых моделей (Distillation, Quantization, Pruning, LoRA)]]"
generated: 2026-05-19
---

## Q001
type: fact
difficulty: junior
key_concepts: [квантизация, FP32, INT8/INT4, экономия памяти]
optional_concepts: [scale и zero-point]
text: Что делает квантизация языковой модели и почему переход с FP32 на INT8 или INT4 уменьшает требования к памяти?
follow_up:
  - Что может пойти не так, если квантизовать модель до INT4 без проверки качества на целевой задаче?

## Q002
type: compare
difficulty: junior
key_concepts: [PTQ, QAT, обучение с имитацией квантизации, качество модели]
optional_concepts: []
text: Чем Post-Training Quantization отличается от Quantization-Aware Training?
follow_up:
  - Если у вас нет доступа к обучающему датасету, какой вариант квантизации вы выберете и какие риски останутся?

## Q003
type: trap
difficulty: junior
key_concepts: [unstructured pruning, sparse веса, отсутствие реального ускорения, аппаратная поддержка]
optional_concepts: [structured pruning]
text: Правда ли, что если мы обнулили 50% весов модели через unstructured pruning, инференс автоматически станет примерно в 2 раза быстрее?
follow_up:
  - Что должно быть выполнено на уровне библиотек или железа, чтобы sparse-модель действительно ускорилась?

## Q004
type: scenario
difficulty: middle
key_concepts: [knowledge distillation, мягкие метки, температура, dark knowledge]
optional_concepts: [KL-дивергенция]
text: Команда обучает маленький student по ответам большого teacher, но использует только argmax-класс teacher как обычную жёсткую метку. Качество student почти не лучше модели, обученной на исходных labels. Что потеряли в таком пайплайне?
follow_up:
  - Как изменится обучение, если teacher выдаёт слишком уверенное распределение вероятностей почти с единицей на одном классе?

## Q005
type: compare
difficulty: middle
key_concepts: [structured pruning, unstructured pruning, скорость инференса, падение качества]
optional_concepts: [удаление голов attention или слоёв]
text: Сравните structured pruning и unstructured pruning: какой метод чаще даёт реальное ускорение, а какой обычно безопаснее по качеству?
follow_up:
  - Если нужно ускорить модель на обычном GPU без специальных sparse kernels, какой тип pruning вы бы рассматривали первым?

## Q006
type: chain
difficulty: middle
key_concepts: [DistilBERT, уменьшение числа слоёв, distillation loss, cosine embedding loss]
optional_concepts: [MLM loss, удаление NSP]
text: Объясните, как DistilBERT получает меньшую архитектуру из BERT-base и почему при обучении ему недостаточно только обычного MLM loss.
follow_up:
  - Что изменится в рисках качества, если вы дистиллируете модель для домена, сильно отличающегося от данных teacher?

## Q007
type: tradeoff
difficulty: senior
key_concepts: [QLoRA, INT4-квантизация, LoRA, ограничение GPU-памяти]
optional_concepts: [замороженные базовые веса, low-rank адаптеры]
text: Вам нужно дообучить 70B LLM под внутренний домен, но доступна только одна A100 80GB. Почему QLoRA может быть практичным выбором и какие компромиссы он вносит?
follow_up:
  - Что вы будете мониторить, если после QLoRA качество на общих задачах ухудшилось, хотя доменная валидация выросла?

## Q008
type: system_design
difficulty: senior
key_concepts: [комбинирование методов сжатия, latency, memory footprint, контроль деградации качества]
optional_concepts: [A/B тест, fallback на большую модель]
text: Нужно задеплоить LLM-сервис с жёстким SLA по latency и ограниченной GPU-памятью. Как бы вы спроектировали стратегию сжатия и проверки качества, комбинируя distillation, quantization, pruning или LoRA?
follow_up:
  - Как изменится дизайн, если распределение пользовательских запросов начало дрейфовать и маленькая модель стала чаще ошибаться на новых типах запросов?
