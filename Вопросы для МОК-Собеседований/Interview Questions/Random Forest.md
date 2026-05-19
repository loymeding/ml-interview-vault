---
topic: Random Forest
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/Random Forest|Random Forest]]"
generated: 2026-05-19
source: "Classic Machine Learning/Interview Questions/Вопросы для МОК-собеседования.md"
---

## Q001
type: fact
difficulty: middle
key_concepts: [bagging, случайный поднабор признаков, декорреляция деревьев, variance reduction]
optional_concepts: [max_features]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Метод случайных подпространств (RSM)|Ответ"
text: Bagging + Random Subspaces: роль случайности признаков?
follow_up:
  - Что случится, если на каждом сплите давать дереву все признаки?
  - Почему случайность признаков особенно важна при сильных предикторах?

## Q002
type: fact
difficulty: middle
key_concepts: [число деревьев, max_features, max_depth, variance и bias]
optional_concepts: [min_samples_leaf]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Гиперпараметры и когда их тюнить|Ответ"
text: Параметры: n_estimators, max_features, max_depth?
follow_up:
  - Как увеличение n_estimators влияет на переобучение и время?
  - Какие гиперпараметры сильнее всего контролируют сложность леса?

## Q003
type: compare
difficulty: middle
key_concepts: [bias-variance tradeoff, усреднение деревьев, декорреляция ошибок, стабильность предсказаний]
optional_concepts: [bootstrap]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Формальный вывод: бэггинг снижает дисперсию в $k$ раз|Ответ"
text: Сдвиг и разброс: дерево vs лес?
follow_up:
  - Почему лес обычно имеет меньший variance, чем одно дерево?
  - Может ли Random Forest увеличить bias относительно глубокого дерева?

## Q004
type: fact
difficulty: middle
key_concepts: [entropy, Gini impurity, MSE в регрессии, уменьшение неоднородности]
optional_concepts: [information gain]
source_refs:
  - "Classic Machine Learning/Интерпретируемость/Интерпретация и важность признаков|Ответ"
text: Variable importance: decrease-impurity, permutation, SHAP?
follow_up:
  - Чем Gini отличается от энтропии интуитивно?
  - Какой критерий использовать для регрессионного дерева?

## Q005
type: fact
difficulty: middle
key_concepts: [out-of-bag score, bootstrap, валидация без holdout, ограничения при малых данных]
optional_concepts: [OOB decision function]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Out-of-Bag (OOB) оценка|Ответ"
text: OOB-score и его ограничения?
follow_up:
  - Почему OOB не требует отдельного validation set?
  - Когда OOB-score может быть оптимистичным или нестабильным?

## Q006
type: fact
difficulty: middle
key_concepts: [ExtraTrees, Isolation Forest, случайные сплиты, аномалии через короткие пути]
optional_concepts: [unsupervised anomaly detection]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Extremely Randomized Trees (Extra Trees)|Extra Trees"
  - "Classic Machine Learning/Модели/Isolation Forest|Isolation Forest"
text: Extremely Randomized Trees, Isolation Forest сравнение?
follow_up:
  - Чем цель Isolation Forest отличается от Random Forest?
  - Почему аномалии изолируются меньшим числом сплитов?

## Q007
type: fact
difficulty: middle
key_concepts: [кусочно-постоянные предсказания, среднее в листьях, отсутствие линейной экстраполяции, границы обучающего диапазона]
optional_concepts: [GBDT тоже ограничен деревьями]
source_refs:
  - "Classic Machine Learning/Модели/Решающие деревья#Проблема экстраполяции|Ответ"
text: Почему лес плохо экстраполирует в регрессии?
follow_up:
  - Что предскажет лес за пределами диапазона train таргета?
  - Какая модель лучше подойдёт для линейного тренда вне train диапазона?

## Q008
type: fact
difficulty: middle
key_concepts: [число деревьев, глубина деревьев, число посещаемых узлов, параллелизация]
optional_concepts: [batch inference]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Вычислительная сложность (Время и Память)|Ответ"
text: Время infer O(n_{trees},d)?
follow_up:
  - Как max_depth влияет на latency?
  - Почему лес можно эффективно параллелить на inference?

## Q009
type: fact
difficulty: middle
key_concepts: [корреляция ошибок, bootstrap, random subspaces, max_features]
optional_concepts: [ExtraTrees]
source_refs:
  - "Classic Machine Learning/Модели/Ансамблевые методы — Bagging, Boosting, Stacking, Blending#Каверзные вопросы|Зачем ограничивать число признаков?"
text: Корреляция деревьев и стратегия её уменьшения?
follow_up:
  - Как уменьшить корреляцию деревьев без уменьшения их силы слишком сильно?
  - Почему очень похожие деревья дают мало выигрыша от ансамбля?

## Q010
type: fact
difficulty: middle
key_concepts: [max_depth, min_samples_leaf, переобучение листьев, bias-variance]
optional_concepts: [шумные признаки]
source_refs:
  - "Classic Machine Learning/Модели/Random Forest — RSM и OOB#Гиперпараметры и когда их тюнить|Ответ"
text: Когда стоит ограничивать глубину каждого дерева?
follow_up:
  - В каких данных глубокие деревья особенно опасны?
  - Как ограничение глубины влияет на extrapolation проблему?
