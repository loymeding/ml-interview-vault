---
topic: k-NN
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/k-NN|k-NN]]"
generated: 2026-05-19
source: "Classic Machine Learning/Interview Questions/Вопросы для МОК-собеседования.md"
---

## Q001
type: fact
difficulty: middle
key_concepts: [lazy learning, память O(Nd), стоимость inference, отсутствие явного обучения]
optional_concepts: [индексы ближайших соседей]
source_refs:
  - "Classic Machine Learning/Модели/kNN — метрические методы#Почему kNN вычислительно дорогой при инференсе?|Ответ"
text: Хранение всей обучающей выборки?
follow_up:
  - Почему k-NN почти не тратит время на fit?
  - Что станет bottleneck при миллионах объектов?

## Q002
type: fact
difficulty: middle
key_concepts: [евклидова метрика, манхэттенская метрика, масштаб признаков, нормализация]
optional_concepts: [cosine distance]
source_refs:
  - "Classic Machine Learning/Модели/kNN — метрические методы#Метрика Минковского и параметр P|Метрики"
  - "Classic Machine Learning/Модели/kNN — метрические методы#Нормировка признаков — обязательный стандарт|Скейлинг"
text: Метрики расстояния, скейлинг?
follow_up:
  - Почему один признак с большим масштабом может доминировать?
  - Когда cosine distance лучше Euclidean?

## Q003
type: fact
difficulty: middle
key_concepts: [малый k высокий variance, большой k высокий bias, валидация k, локальность решения]
optional_concepts: [нечётное k]
source_refs:
  - "Classic Machine Learning/Модели/kNN — метрические методы#Bias-Variance Tradeoff в kNN|Ответ"
text: Выбор k: bias-variance?
follow_up:
  - Что будет при k=1?
  - Почему слишком большой k сглаживает границу классов?

## Q004
type: fact
difficulty: middle
key_concepts: [inverse distance weighting, локальная близость, нулевое расстояние, влияние дальних соседей]
optional_concepts: [kernel weighting]
source_refs:
  - "Classic Machine Learning/Модели/kNN — метрические методы#Взвешенный kNN|Ответ"
text: Весовые схемы (inverse distance)?
follow_up:
  - Как обработать соседа с расстоянием 0?
  - Когда взвешенный k-NN лучше обычного голосования?

## Q005
type: fact
difficulty: middle
key_concepts: [KD-tree, Ball-tree, FAISS, approximate nearest neighbors]
optional_concepts: [curse of dimensionality]
source_refs:
  - "Classic Machine Learning/Модели/kNN — метрические методы#Почему kNN вычислительно дорогой при инференсе?|Ответ"
text: KD-tree / Ball-tree / FAISS для ускорения?
follow_up:
  - Почему KD-tree деградирует в высокой размерности?
  - Чем approximate search платит за ускорение?

## Q006
type: fact
difficulty: middle
key_concepts: [концентрация расстояний, разреженность пространства, рост требуемых данных, снижение различимости соседей]
optional_concepts: [feature selection]
source_refs:
  - "Classic Machine Learning/Модели/kNN — метрические методы#Почему kNN вычислительно дорогой при инференсе?|Ответ"
text: Curse of dimensionality?
follow_up:
  - Почему ближайший и дальний сосед становятся похожи по расстоянию?
  - Как PCA или feature selection могут помочь k-NN?

## Q007
type: compare
difficulty: middle
key_concepts: [голосование классов, среднее таргета соседей, взвешивание по расстоянию, локальная аппроксимация]
optional_concepts: [Nadaraya-Watson]
source_refs:
  - "Classic Machine Learning/Модели/kNN — метрические методы#Формула Надарая-Ватсона|Ответ"
text: Regression vs Classification?
follow_up:
  - Как k-NN предсказывает непрерывный таргет?
  - Что меняется в метриках качества для classification и regression?
