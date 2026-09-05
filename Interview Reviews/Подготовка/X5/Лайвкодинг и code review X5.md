---
tags: [interview-prep, x5, live-coding, code-review, classic-ml, pandas, ab-testing]
company: X5
type: live-coding-problems
status: active
tasks_total: 10
cssclasses: [wide-page]
---

# 💻 X5 — лайвкодинг и code review

> [!important] Режим тренировки
> В этой карточке только условия. Для каждой задачи сначала проговори подход, напиши код и тесты, а потом открывай [[Interview Reviews/Подготовка/X5/X5 — решения лайвкодинга и code review|разборы и решения]].

## 🎯 Как тренироваться

| Формат | Время | Что проверяет |
|---|---:|---|
| Live coding | 25-35 минут | алгоритм, код, тесты, асимптотика |
| Code review | 10-15 минут | умение увидеть баг, leakage, численную и продуктовую проблему |

---

## 🧠 Live coding: Classic ML и A/B

### X5-LC1. LogLoss и градиенты логистической регрессии

- [ ] Решено самостоятельно

Реализуй функцию `logistic_loss_and_gradients(X, y, w, b, l2) -> (loss, dw, db)` для бинарной классификации.

Требования: численно стабильный sigmoid, средний LogLoss, L2-регуляризация только весов `w`, векторизованные вычисления, корректная обработка `y` из `0/1`.

Добавь тест: сравни аналитический градиент хотя бы одного веса с численным конечным разностным приближением.

### X5-LC2. Метрики по score и лучший порог

- [ ] Решено самостоятельно

Даны `y_true: list[int]` и `scores: list[float]`. Напиши `evaluate_thresholds`, которая перебирает все осмысленные пороги и возвращает лучший порог по F1, а также Precision, Recall, F1 и ROC-AUC.

Нужно явно договориться, как трактуются равные score и что происходит, когда модель не предсказала ни одного положительного объекта.

### X5-LC3. Time split без утечки

- [ ] Решено самостоятельно

Есть таблица с `user_id`, `event_dt`, `target`. Реализуй функцию, которая строит `n_splits` временных train/validation-разбиений с временным зазором `gap_days`.

В train должны быть только события раньше validation. Один и тот же пользователь не должен оказаться в train и validation одного фолда. Опиши, как поступишь, если после исключения пересекающихся пользователей validation стал слишком мал.

### X5-LC4. Лучший decision stump

- [ ] Решено самостоятельно

Для одного числового признака `x` и бинарной метки `y` найди порог и направление правила вида `x <= threshold`, которые минимизируют взвешенную Gini impurity.

Верни порог, предсказание для левой и правой ветвей и итоговый score. Не допускай пустых листьев.

### X5-LC5. Таргет следующего дерева в градиентном бустинге

- [ ] Решено самостоятельно

Для бинарной классификации даны истинные метки `y` и текущие raw-score модели `F`. Реализуй функцию, которая возвращает вероятности, отрицательные градиенты LogLoss по `F` и, если готов, ньютоновские веса второго порядка.

Поясни, что именно будет учить следующее дерево и почему нельзя обучать его напрямую на исходный `y`.

### X5-LC6. Bootstrap для разницы метрик A/B

- [ ] Решено самостоятельно

Даны метрики на уровне пользователя в control и treatment. Реализуй `bootstrap_delta(control, treatment, n_boot, seed)`, возвращающую оценку разницы средних и 95% percentile confidence interval.

Добавь проверки на пустую группу, фиксированный seed и интерпретацию интервала, который пересекает ноль.

---

## 🔎 Code review: Classic ML и данные

### X5-CR1. Численная устойчивость и регуляризация

- [ ] Разобрано самостоятельно

Найди проблемы и предложи исправления:

```python
def loss_and_grad(X, y, w, b, l2=0.0):
    z = X @ w + b
    p = 1 / (1 + np.exp(-z))
    loss = -(y * np.log(p) + (1 - y) * np.log(1 - p)).sum()
    loss += l2 * (w @ w + b ** 2)
    dw = X.T @ (p - y) + 2 * l2 * w
    db = (p - y).sum() + 2 * l2 * b
    return loss, dw, db
```

Обсуди минимум пять аспектов: масштаб loss/градиента, overflow, `log(0)`, регуляризацию bias, согласование коэффициента L2 и граничные формы входов.

### X5-CR2. Leakage в feature engineering

- [ ] Разобрано самостоятельно

```python
def prepare_data(df):
    df = df.copy()
    df["income"] = df["income"].fillna(df["income"].median())
    df["city_freq"] = df.groupby("city")["target"].transform("mean")
    train, valid = train_test_split(df, test_size=0.2, random_state=42)
    scaler = StandardScaler().fit(df[["age", "income"]])
    train[["age", "income"]] = scaler.transform(train[["age", "income"]])
    valid[["age", "income"]] = scaler.transform(valid[["age", "income"]])
    return train, valid
```

Найди все источники утечки и предложи безопасный train-only pipeline, включая способ построения target encoding.

### X5-CR3. Неверный ROC-AUC и нестабильная метрика

- [ ] Разобрано самостоятельно

```python
def report_metrics(y_true, probabilities):
    labels = (probabilities > 0.5).astype(int)
    auc = roc_auc_score(y_true, labels)
    precision = precision_score(y_true, labels)
    recall = recall_score(y_true, labels)
    return {"auc": auc, "precision": precision, "recall": recall}
```

Что здесь корректно, а что нет? Какие параметры и дополнительные срезы нужны для production-отчёта при сильном дисбалансе?

### X5-CR4. Cross-validation для временных данных

- [ ] Разобрано самостоятельно

```python
def cv_score(df, features, target):
    kf = KFold(n_splits=5, shuffle=True, random_state=42)
    scores = []
    for train_idx, valid_idx in kf.split(df):
        model = CatBoostClassifier(verbose=False)
        model.fit(df.iloc[train_idx][features], df.iloc[train_idx][target])
        p = model.predict_proba(df.iloc[valid_idx][features])[:, 1]
        scores.append(roc_auc_score(df.iloc[valid_idx][target], p))
    return np.mean(scores)
```

Представь, что `df` содержит кредитные заявки за несколько лет и несколько записей на клиента. Найди риски валидации и спроектируй более честный вариант.

---

## 🧾 После каждой попытки

1. Запиши решение без подсказок.
2. Назови временную и пространственную сложность.
3. Добавь минимум три крайних случая.
4. Проговори ответ как интервьюеру за 60-90 секунд.
