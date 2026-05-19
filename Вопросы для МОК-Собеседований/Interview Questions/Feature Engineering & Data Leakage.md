---
topic: Feature Engineering & Data Leakage
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/Feature Engineering & Data Leakage|Feature Engineering & Data Leakage]]"
generated: 2026-05-19
source: "Classic Machine Learning/Interview Questions/Вопросы для МОК-собеседования.md"
---

## Q001
type: compare
difficulty: middle
key_concepts: [train-time transformation, test-time transformation, fit vs transform, production consistency]
optional_concepts: [feature store]
source_refs:
  - "Classic Machine Learning/Модели/Валидация — разбивка выборки и переобучение#Скрытые источники утечки|Ответ"
text: train-time vs test-time трансформации?
follow_up:
  - Почему transform на test должен использовать параметры train?
  - Как гарантировать одинаковую обработку в train и production?

## Q002
type: fact
difficulty: middle
key_concepts: [cross-validation, OOB validation, стоимость обучения, размер обучающих подвыборок]
optional_concepts: [стратификация]
source_refs:
  - "Classic Machine Learning/Предобработка/Кодирование категориальных признаков#🧐 Вопросы для интервью|Ответ"
text: Target-encoding с k-fold scheme?
follow_up:
  - Когда k-fold надёжнее OOB?
  - Почему OOB дешевле для bagging-моделей?

## Q003
type: fact
difficulty: middle
key_concepts: [fit scaler on train, data leakage, pipeline, validation split]
optional_concepts: [ColumnTransformer]
source_refs:
  - "Classic Machine Learning/Модели/Валидация — разбивка выборки и переобучение#Скрытые источники утечки|Ответ"
text: Scale/normalize после split?
follow_up:
  - Что именно нельзя делать до split?
  - Как sklearn Pipeline защищает от leakage в cross-validation?

## Q004
type: fact
difficulty: middle
key_concepts: [редкие категории, grouping rare labels, unknown category, стабильность encoding]
optional_concepts: [frequency encoding]
source_refs:
  - "Classic Machine Learning/Предобработка/Кодирование категориальных признаков#1. Базовые методы кодирования|Ответ"
text: Handling rare categories?
follow_up:
  - Что делать с категорией, которой не было на train?
  - Почему редкие категории могут привести к переобучению?

## Q005
type: fact
difficulty: middle
key_concepts: [look-ahead bias, time-based split, lag features, rolling statistics]
optional_concepts: [feature availability time]
source_refs:
  - "Classic Machine Learning/Модели/Валидация — разбивка выборки и переобучение#Data Leakage при работе с временными рядами|Ответ"
text: Date-time фичи и look-ahead bias?
follow_up:
  - Как проверить, что фича была доступна на момент предсказания?
  - Почему случайный split опасен для временных признаков?

## Q006
type: fact
difficulty: middle
key_concepts: [Pipeline, ColumnTransformer, разные трансформации колонок, reproducible preprocessing]
optional_concepts: [custom transformers]
source_refs:
  - "Classic Machine Learning/Модели/Валидация — разбивка выборки и переобучение#Скрытые источники утечки|Ответ"
text: Pipeline + ColumnTransformer?
follow_up:
  - Как Pipeline помогает при cross-validation?
  - Где хранить preprocessing логику для production inference?
