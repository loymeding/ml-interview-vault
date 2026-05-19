---
topic: Bootstrap, Bagging
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/Bootstrap, Bagging|Bootstrap, Bagging]]"
generated: 2026-05-19
source: "Classic Machine Learning/Interview Questions/Вопросы для МОК-собеседования.md"
---

## Q001
type: fact
difficulty: middle
key_concepts: [выборка с возвращением, bootstrap sample, out-of-bag объекты, оценка неопределенности]
optional_concepts: [примерно 63.2% уникальных объектов]
source_refs:
  - "Classic Machine Learning/Модели/Ансамблевые методы — Bagging, Boosting, Stacking, Blending#2. Bagging (Bootstrap Aggregating)|Ответ"
text: Определение бутстрапа, выборка с возвращением?
follow_up:
  - Почему в bootstrap-выборке часть объектов повторяется?
  - Какая доля объектов в среднем остаётся out-of-bag?

## Q002
type: compare
difficulty: middle
key_concepts: [bias, variance, bootstrap estimator, разброс оценок]
optional_concepts: [смещение bootstrap-оценки]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Формальный вывод: бэггинг снижает дисперсию в $k$ раз|Ответ"
text: Не-смещённость vs увеличение дисперсии?
follow_up:
  - Почему bootstrap может увеличить разброс отдельной обучающей выборки?
  - Что именно снижает bagging: bias или variance?

## Q003
type: fact
difficulty: middle
key_concepts: [усреднение независимых моделей, дисперсия ансамбля, корреляция ошибок, эффективность bagging]
optional_concepts: [rho в формуле коррелированных моделей]
source_refs:
  - "Classic Machine Learning/Модели/Ансамблевые методы — Bagging, Boosting, Stacking, Blending#Формула / Схема|Ответ"
text: Усреднение n моделей: ошибка σ²/n при некоррелированных предсказаниях?
follow_up:
  - Что будет, если ошибки моделей сильно коррелированы?
  - Почему Random Subspaces уменьшают корреляцию деревьев?

## Q004
type: fact
difficulty: middle
key_concepts: [OOB объекты, оценка качества без отдельной validation, bootstrap samples, агрегация предсказаний]
optional_concepts: [OOB для feature importance]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Out-of-Bag (OOB) оценка|Ответ"
text: Out-of-Bag оценка качества?
follow_up:
  - Почему OOB можно рассматривать как встроенную валидацию?
  - Когда OOB-оценка может быть шумной?

## Q005
type: fact
difficulty: middle
key_concepts: [cross-validation, OOB validation, стоимость обучения, размер обучающих подвыборок]
optional_concepts: [стратификация]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Out-of-Bag (OOB) оценка|Ответ"
text: Сравнение с k-fold CV?
follow_up:
  - Когда k-fold надёжнее OOB?
  - Почему OOB дешевле для bagging-моделей?

## Q006
type: fact
difficulty: middle
key_concepts: [bias модели, variance reduction, слабая базовая модель, underfitting]
optional_concepts: [сложность базового алгоритма]
source_refs:
  - "Classic Machine Learning/Модели/Bagging vs Boosting#Bias-Variance Tradeoff|Ответ"
text: Когда бэггинг не работает (высокий bias)?
follow_up:
  - Почему bagging не исправит систематическую ошибку модели?
  - Как понять, что базовая модель слишком простая?

## Q007
type: fact
difficulty: middle
key_concepts: [Extremely Randomized Trees, случайные пороги, снижение корреляции, bias-variance tradeoff]
optional_concepts: [ExtraTrees]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Extremely Randomized Trees (Extra Trees)|Ответ"
text: Экстремальный случай: полностью случайные деревья?
follow_up:
  - Почему случайные пороги могут повысить bias?
  - За счёт чего ExtraTrees иногда быстрее Random Forest?
