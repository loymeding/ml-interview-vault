---
tags: [ансамбли, bagging, boosting, stacking, blending, random-forest, xgboost, meta-learner, oof]
тип: сравнение
уровень: middle
сложность: средняя
статус: готово
готовность: 97
создано: 2026-05-15
источники:
  - "Яндекс Handbook — ансамбли в машинном обучении"
  - "Тренировки по ML — лекция 4"
предпосылки:
  - "Решающие деревья"
  - "Валидация — разбивка выборки и переобучение"
связано:
  - "Bagging vs Boosting"
  - "Random Forest — RSM и OOB"
  - "От AdaBoost к Gradient Boosting — XGBoost, LightGBM, CatBoost"
сравнить-с:
  - "Bagging vs Boosting"
---

# Ансамблевые методы — Bagging, Boosting, Stacking, Blending

> [!abstract] Суть
> Все ансамблевые методы основаны на одной идее: **несколько разных моделей лучше одной хорошей**. Разница — в том, как именно строятся модели и как объединяются их предсказания. **Bagging** усредняет независимые модели (снижает дисперсию), **Boosting** исправляет ошибки последовательно (снижает смещение), **Stacking** учит мета-модель комбинировать ответы базовых через cross-validation, **Blending** делает то же самое быстрее, но грубее — через hold-out.

## Ответ

### 1. Общая идея: почему ансамбли работают?

Представьте, что вы задаёте один и тот же вопрос 100 независимым экспертам. Каждый эксперт ошибается, но по-своему. Когда вы усредняете их ответы — **случайные ошибки компенсируют друг друга**, а системные закономерности усиливаются.

Математически: если $N$ независимых моделей дают ошибку с дисперсией $\sigma^2$, то среднее их предсказаний имеет дисперсию $\sigma^2 / N$.

Три условия эффективного ансамбля:
1. **Разнообразие:** модели делают разные ошибки (не 100 одинаковых экспертов)
2. **Качество:** каждая модель работает лучше случайного угадывания
3. **Независимость:** ошибки слабо коррелируют между моделями

---

### 2. Bagging (Bootstrap Aggregating)

**Идея:** создай много версий одной модели на разных подвыборках данных, объедини усреднением.

**Как работает:**
1. Из $N$ примеров делаем $k$ bootstrap-выборок (случайно с возвращением, каждая тоже размером $N$)
2. На каждой выборке независимо обучаем одну и ту же модель
3. Итоговое предсказание: среднее (регрессия) или мажоритарное голосование (классификация)

```
Датасет (N примеров)
    ├─ Bootstrap 1 → Модель 1 ─┐
    ├─ Bootstrap 2 → Модель 2 ─┼──▶ Average / Majority vote
    └─ Bootstrap k → Модель k ─┘
```

**Главный эффект: снижение дисперсии (variance)**. Каждая модель «шумит», но некоррелированный шум при усреднении затухает. Смещение (bias) при этом практически не меняется — поэтому базовые модели должны быть **сложными** (глубокие деревья, низкий bias).

**Канонический пример — Random Forest:** каждое дерево обучается на своём bootstrap-сэмпле + при каждом сплите рассматривается только случайное подмножество признаков (`max_features`). Второй механизм дополнительно уменьшает корреляцию между деревьями.

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import cross_val_score

X, y = make_classification(n_samples=1000, n_features=20, random_state=42)

rf = RandomForestClassifier(
    n_estimators=200,     # количество деревьев
    max_features="sqrt",  # √p признаков на сплит (классический bagging-трюк)
    max_depth=None,       # глубокие деревья — низкий bias, высокий variance (компенсирует ансамбль)
    bootstrap=True,       # bootstrap-выборки
    oob_score=True,       # Out-Of-Bag оценка без отдельного val
    random_state=42
)
rf.fit(X, y)
print(f"OOB accuracy: {rf.oob_score_:.3f}")
print(f"CV accuracy: {cross_val_score(rf, X, y, cv=5).mean():.3f}")
```

**Когда использовать:** нужна быстрая устойчивая модель; данных немного (OOB даёт бесплатную валидацию); нет времени на тюнинг; высокий variance у одиночной модели.

---

### 3. Boosting

**Идея:** строй модели последовательно — каждая следующая фокусируется на ошибках предыдущих.

**Как работает (концептуально):**
1. Обучаем первую слабую модель $f_1$ на данных
2. Смотрим, где $f_1$ ошиблась — увеличиваем «вес» этих примеров
3. Обучаем $f_2$ на данных с обновлёнными весами (больше внимания трудным примерам)
4. Повторяем $k$ раз
5. Финал: взвешенная сумма всех моделей

```
Данные → f1 → ошибки → f2 → ошибки → f3 → ... → Σ wᵢfᵢ
```

**Главный эффект: снижение смещения (bias)**. Базовые модели намеренно слабые (неглубокие деревья, высокий bias) — каждая вносит маленькое улучшение. Именно поэтому бустинг склонен к переобучению: последовательные ошибки усиливаются.

**Современные реализации:** XGBoost, LightGBM, CatBoost — градиентный бустинг с регуляризацией.

→ Подробнее: [[Classic Machine Learning/Модели/От AdaBoost к Gradient Boosting — XGBoost, LightGBM, CatBoost]]

```python
import lightgbm as lgb
from sklearn.model_selection import cross_val_score

lgb_model = lgb.LGBMClassifier(
    n_estimators=500,
    learning_rate=0.05,  # маленький LR + много деревьев
    max_depth=6,
    subsample=0.8,       # случайность → меньше переобучения
    colsample_bytree=0.8,
    random_state=42
)
print(f"LightGBM CV: {cross_val_score(lgb_model, X, y, cv=5).mean():.3f}")
```

---

### 4. Stacking (Stacked Generalization)

**Идея:** обучи несколько **разных** базовых моделей (уровень 0), затем обучи мета-модель (уровень 1) предсказывать правильный ответ на основе их предсказаний.

Ключевая проблема наивного стекинга: если мета-модель обучается на предсказаниях базовых моделей на тех же данных — базовые модели уже «видели» эти примеры и переобучились. Мета-модель обучится на переоптимистичных предсказаниях → утечка.

**Решение: Out-of-Fold (OOF) предсказания** через cross-validation:

```
Данные ─── разбиваем на K фолдов ───────────────────────────────────┐
                                                                      │
Фолд 1: обучаем M1 на фолдах 2..K → предсказываем фолд 1 → OOF₁    │
Фолд 2: обучаем M1 на фолдах 1,3..K → предсказываем фолд 2 → OOF₂  │
...                                                                   │
Конкатенируем OOF₁..OOFₖ → полный OOF-вектор модели M1              │
То же для M2, M3, ...                                                 │
                                                                      │
OOF-векторы (= признаки для уровня 1) → обучаем мета-модель          │
                                                                      └─── готово
```

Каждое предсказание в OOF-векторе сделано на данных, которые модель **не видела при обучении** → утечки нет.

```python
import numpy as np
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import StratifiedKFold, cross_val_score
from sklearn.datasets import make_classification

X, y = make_classification(n_samples=1000, n_features=20, random_state=42)

# --- Уровень 0: базовые модели ---
base_models = [
    RandomForestClassifier(n_estimators=100, random_state=42),
    GradientBoostingClassifier(n_estimators=100, random_state=42),
]

kf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
oof_preds = np.zeros((len(X), len(base_models)))  # OOF-матрица

for m_idx, model in enumerate(base_models):
    for train_idx, val_idx in kf.split(X, y):
        model.fit(X[train_idx], y[train_idx])
        oof_preds[val_idx, m_idx] = model.predict_proba(X[val_idx])[:, 1]

# --- Уровень 1: мета-модель ---
meta_model = LogisticRegression()
meta_score = cross_val_score(meta_model, oof_preds, y, cv=5).mean()
print(f"Stacking CV: {meta_score:.3f}")

# Удобная готовая реализация
from sklearn.ensemble import StackingClassifier

stack = StackingClassifier(
    estimators=[("rf", base_models[0]), ("gb", base_models[1])],
    final_estimator=LogisticRegression(),
    cv=5,
    passthrough=False  # True — добавить исходные признаки к OOF в мета-слой
)
print(f"sklearn Stacking: {cross_val_score(stack, X, y, cv=5).mean():.3f}")
```

**Советы по стекингу:**
- Базовые модели должны быть **разнообразными** (RF + GB + LR + KNN) — иначе мета-модель ничему не научится
- Мета-модель обычно простая (LogisticRegression, Ridge) — сложная переобучится
- `passthrough=True` — передать исходные признаки в мета-слой — иногда помогает
- В Kaggle часто делают 2-3 уровня стекинга, но на практике 2 уровня — оптимум

---

### 5. Blending

**Идея:** то же, что стекинг, но вместо cross-validation используется фиксированный hold-out сет.

**Как работает:**
1. Делим данные: Train (70%) / Blend (15%) / Test (15%)
2. Базовые модели обучаются на Train
3. Базовые модели предсказывают на Blend → эти предсказания становятся признаками для мета-модели
4. Мета-модель обучается на Blend-предсказаниях
5. Предсказание на Test: сначала базовые модели, потом мета-модель

```python
from sklearn.model_selection import train_test_split

X_train, X_temp, y_train, y_temp = train_test_split(X, y, test_size=0.3, random_state=42)
X_blend, X_test, y_blend, y_test = train_test_split(X_temp, y_temp, test_size=0.5, random_state=42)

blend_preds = np.zeros((len(X_blend), len(base_models)))
test_preds = np.zeros((len(X_test), len(base_models)))

for m_idx, model in enumerate(base_models):
    model.fit(X_train, y_train)
    blend_preds[:, m_idx] = model.predict_proba(X_blend)[:, 1]
    test_preds[:, m_idx] = model.predict_proba(X_test)[:, 1]

meta_model = LogisticRegression()
meta_model.fit(blend_preds, y_blend)
print(f"Blending Test acc: {meta_model.score(test_preds, y_test):.3f}")
```

**Разница со стекингом:**
- Blending **быстрее** (нет K-fold обучения базовых моделей)
- Но **теряет данные** (blend-сет не используется для обучения базовых моделей)
- **Выше риск переобучения мета-модели** (blend-сет маленький)
- В Kaggle часто используется когда обучение одной модели занимает часы

---

### 6. Сравнение всех четырёх методов

| | Bagging | Boosting | Stacking | Blending |
|---|---|---|---|---|
| **Обучение** | Параллельно | Последовательно | OOF (K итераций) | Hold-out |
| **Скорость** | Быстро | Медленнее | Медленно | Быстро |
| **Главный эффект** | ↓ Variance | ↓ Bias | ↓ Оба | ↓ Оба |
| **Базовые модели** | Одинаковые, сложные | Одинаковые, слабые | **Разные** (ключевое!) | **Разные** |
| **Риск переобучения** | Низкий | Высокий | Средний (OOF защищает) | Выше (маленький blend) |
| **Интерпретируемость** | Средняя | Низкая | Очень низкая | Очень низкая |
| **Использование** | RF в продакшне | XGBoost/LGB на таблицах | Kaggle top | Kaggle быстро |

**Когда что выбирать:**
- **Bagging (RF):** быстрый baseline, интерпретируемость важна, мало данных
- **Boosting:** максимальное качество на табличных данных, есть время на тюнинг
- **Stacking:** Kaggle финал, данных много, нужно выжать последний % качества
- **Blending:** стекинг нужен, но обучение слишком долгое; или быстрый эксперимент

## Формула / Схема

**Предсказание Bagging:**

$$\Large \hat{y} = \frac{1}{K} \sum_{k=1}^{K} f_k(\mathbf{x})$$

где:
- $K$ — количество базовых моделей
- $f_k(\mathbf{x})$ — предсказание $k$-й модели (обученной на bootstrap-выборке)

**Дисперсия ансамбля:**

$$\Large \text{Var}\left(\hat{y}\right) = \frac{\sigma^2}{K} + \frac{K-1}{K} \cdot \rho \cdot \sigma^2$$

где:
- $\sigma^2$ — дисперсия одной модели
- $\rho$ — средняя корреляция между моделями
- При $\rho = 0$ (независимые): дисперсия $= \sigma^2/K$ — падает линейно
- При $\rho = 1$ (одинаковые): дисперсия $= \sigma^2$ — ансамбль бесполезен

**Предсказание Stacking:**

$$\Large \hat{y} = g\bigl(f_1^{\text{OOF}}(\mathbf{x}),\; f_2^{\text{OOF}}(\mathbf{x}),\; \ldots,\; f_M^{\text{OOF}}(\mathbf{x})\bigr)$$

где:
- $f_m^{\text{OOF}}(\mathbf{x})$ — OOF-предсказание $m$-й базовой модели
- $g(\cdot)$ — мета-модель (логрегрессия, Ridge и т.п.)

## Типичные ошибки

- **Стекинг без OOF:** тренировать базовые модели на всём трейне, а потом их же предсказания подавать мета-модели — прямая утечка. Базовые модели «знают» все примеры, их предсказания оптимистичны.
- **Одинаковые базовые модели в стекинге:** 3 Random Forest с разными гиперпараметрами почти одинаково ошибаются — мета-модель ничему не научится. Нужна реальная диверсификация: RF + LightGBM + линейная модель + KNN.
- **Сложная мета-модель:** если мета-модель — тот же XGBoost, она переобучится на маленьком наборе OOF-признаков. LogisticRegression / Ridge / простой LinearSVC — оптимально.
- **Использовать бустинг там, где важна скорость инференса:** бустинг — последовательный, 500 деревьев применяются одно за другим. RF с 500 деревьями работает быстрее при инференсе (параллельно).
- **Путать blending и stacking:** в blending blend-сет не входит в обучение базовых моделей, а в stacking OOF охватывает все данные.

## Каверзные вопросы

> [!question] Зачем Random Forest ограничивает число признаков на каждом сплите?
> Цель — уменьшить **корреляцию между деревьями**. Без `max_features` все деревья будут начинать сплит с одного и того же самого важного признака → они будут похожи → корреляция высокая → дисперсия ансамбля почти не падает. Ограничение признаков заставляет деревья «специализироваться» на разных подмножествах — разнообразие растёт, корреляция падает.

> [!question] Почему мета-модель в стекинге обычно линейная?
> OOF-предсказания базовых моделей — это числа от 0 до 1 (вероятности), их всего $M$ штук (по числу базовых моделей). Нелинейная мета-модель легко переобучится на таком маленьком признаковом пространстве. Логрегрессия просто учит **оптимальные веса** для каждой базовой модели — чего, как правило, достаточно.

> [!question] Может ли стекинг дать результат хуже, чем лучшая базовая модель?
> Да — если мета-модель переобучилась. Сигнал: тест CV значительно хуже, чем лучшая базовая модель. Лечится: проще мета-модель (LR вместо XGBoost), больше фолдов в OOF, `passthrough=False`.

## Проверка себя

- Почему bagging снижает variance, а не bias? Объясните интуитивно.
- Что произойдёт с ансамблем bagging, если все базовые модели полностью одинаковые (ρ = 1)?
- Почему в stacking нельзя просто обучить все модели на train, а потом их предсказания дать мета-модели?
- Чем blending хуже stacking с точки зрения риска переобучения?
- Вы хотите сделать stacking из RF + LightGBM + SVM. Какую мета-модель выберете и почему?
- В какой ситуации вы предпочтёте blending stacking?

## Предпосылки

- [[Classic Machine Learning/Модели/Решающие деревья]]
- [[Classic Machine Learning/Модели/Валидация — разбивка выборки и переобучение]]
- [[Classic Machine Learning/Модели/Bagging vs Boosting]]

## Связано

- [[Classic Machine Learning/Модели/Random Forest — RSM и OOB]]
- [[Classic Machine Learning/Модели/От AdaBoost к Gradient Boosting — XGBoost, LightGBM, CatBoost]]
- [[Classic Machine Learning/Модели/Bias-Variance — RF vs GB]]

## Сравнить с

- [[Classic Machine Learning/Модели/Bagging vs Boosting]]
- [[Classic Machine Learning/Модели/Random Forest — RSM и OOB]]

## Источники

- [Яндекс Handbook — ансамбли в машинном обучении](https://education.yandex.ru/handbook/ml/article/ansambli-v-mashinnom-obuchenii)
- [Тренировки по ML — лекция 4](https://www.youtube.com/@Young_and_Yandex)

---
[[🗺️ Индекс|Назад к разделу]]
