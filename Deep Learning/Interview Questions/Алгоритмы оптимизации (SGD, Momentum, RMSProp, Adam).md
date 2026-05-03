---
topic: Алгоритмы оптимизации (SGD, Momentum, RMSProp, Adam)
card_ref: "[[Алгоритмы оптимизации (SGD, Momentum, RMSProp, Adam)]]"
generated: 2026-04-29
---

## Q001
type: fact
difficulty: junior
key_concepts: [инерция градиентов, сглаживание осцилляций, гиперпараметр beta]
optional_concepts: [аналогия с тяжёлым шаром]
text: Что такое «момент» (momentum) в контексте оптимизации нейросетей и какую проблему базового SGD он решает?
follow_up:
  - Как изменится поведение оптимизатора, если установить beta=0.99 вместо стандартного 0.9 — в чём риск на практике?

---

## Q002
type: compare
difficulty: junior
key_concepts: [адаптивный шаг, накопленный квадрат градиента, разные скорости обучения для разных параметров]
optional_concepts: [проблема затухающего LR в RMSProp]
text: В чём принципиальное отличие RMSProp от SGD с моментом? Почему адаптивный шаг может быть важнее инерции в некоторых задачах?
follow_up:
  - Что произойдёт с шагом в RMSProp для параметра, чей градиент стабильно близок к нулю на протяжении 1000 шагов?

---

## Q003
type: trap
difficulty: middle
key_concepts: [L2-регуляризация, weight decay, AdamW, некорректное применение Adam]
optional_concepts: [decoupled weight decay, оригинальная статья Loshchilov & Hutter]
text: Коллега добавил L2-регуляризацию к модели, обучаемой с Adam, через параметр `weight_decay` в `optim.Adam`. Он уверен, что это эквивалентно L2-регуляризации в SGD. Он прав?
follow_up:
  - В каком случае разница между Adam и AdamW становится особенно заметной — на маленьких или больших моделях с большим числом эпох?

---

## Q004
type: scenario
difficulty: middle
key_concepts: [острые vs плоские минимумы, обобщающая способность, разрыв train/test loss]
optional_concepts: [SAM — Sharpness-Aware Minimization]
text: Команда обучила большую модель классификации с Adam, train loss упал до нуля, val loss тоже выглядит хорошо на hold-out. Но после деплоя модель работает заметно хуже, чем ожидалось на реальных данных. Лосс на train и val совпадает, переобучения нет. Какова одна из нетривиальных причин, связанных именно с выбором оптимизатора?
follow_up:
  - Как бы вы проверили гипотезу об «остром» минимуме без полного переобучения модели?

---

## Q005
type: chain
difficulty: middle
key_concepts: [bias correction, первый момент, второй момент, инициализация нулями]
optional_concepts: [формула bias-corrected estimates, поведение на первых шагах]
text: Объясните, зачем в Adam нужна коррекция смещения (bias correction) для моментов — и что произойдёт, если её убрать?
follow_up:
  - Представьте, что вы начинаете обучение с очень маленьким batch size (например, 4). Как это повлияет на первый и второй момент в Adam и стоит ли менять гиперпараметры?

---

## Q006
type: compare
difficulty: middle
key_concepts: [обобщающая способность, скорость сходимости, плоские минимумы, CV vs NLP]
optional_concepts: [learning rate scheduler, warm-up]
text: В каких условиях SGD с моментом может оказаться предпочтительнее Adam, несмотря на то что Adam сходится быстрее? Приведите конкретные сценарии.
follow_up:
  - Если заказчик требует результат за 24 часа GPU-времени, как этот ограничение меняет ваш выбор между SGD+momentum и Adam?

---

## Q007
type: tradeoff
difficulty: senior
key_concepts: [скорость сходимости vs качество обобщения, flat minima, LR scheduling, production constraints]
optional_concepts: [cyclical LR, SWA — Stochastic Weight Averaging]
text: Вы деплоите модель компьютерного зрения в продакшн. Эксперименты показали: Adam сходится за 50 эпох с accuracy 91%, SGD+momentum за 120 эпох даёт 93%. Вычислительный бюджет ограничен, но модель будет обновляться каждый квартал. Как вы обоснуете выбор оптимизатора перед бизнесом и ML-командой?
follow_up:
  - Если через квартал появятся новые классы (дообучение), как это меняет ваше решение — и меняет ли вообще?

---

## Q008
type: debug
difficulty: senior
key_concepts: [learning rate warmup, gradient explosion на старте, адаптивный шаг Adam, нестабильность в начале обучения]
optional_concepts: [epsilon в знаменателе Adam, gradient clipping]
text: Инженер обучает Transformer с нуля. Первые 100 шагов loss резко падает, затем loss взрывается (NaN или очень большое значение), причём это воспроизводится стабильно. Batch size нормальный, данные проверены, архитектура стандартная. Оптимизатор — Adam с lr=1e-3, без warmup. Найдите причину и предложите fix.
follow_up:
  - Предположим, вы добавили warmup на 1000 шагов и gradient clipping — loss всё равно нестабилен, но теперь взрыв происходит на шаге ~5000. Куда смотреть дальше?
