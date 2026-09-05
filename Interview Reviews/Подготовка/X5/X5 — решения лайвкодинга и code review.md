---
tags: [interview-prep, x5, live-coding, code-review, answers, classic-ml, ab-testing]
company: X5
type: live-coding-solutions
status: active
tasks_total: 10
tasks_solved: 10
cssclasses: [wide-page]
---

# ✅ X5 — решения лайвкодинга и code review

> [!warning] Спойлеры
> Это разборы к [[Interview Reviews/Подготовка/X5/Лайвкодинг и code review X5|карточке задач]]. Открывай раздел только после самостоятельной попытки.

## 🧠 Live coding

### X5-LC1. LogLoss и градиенты логистической регрессии

Модель даёт `p = sigmoid(Xw + b)`. Средний LogLoss с L2-штрафом:

$$
\Large L = -\frac{1}{n}\sum_{i=1}^{n}\left[y_i\log p_i + (1-y_i)\log(1-p_i)\right] + \frac{\lambda}{2}\lVert w\rVert_2^2.
$$

`n` - число объектов, `y_i` - истинная метка, `p_i` - вероятность положительного класса, `w` - веса, `b` - bias, `lambda` - сила регуляризации. Bias обычно не штрафуют: он задаёт базовую вероятность класса, а не сложность границы.

```python
import numpy as np


def sigmoid(z: np.ndarray) -> np.ndarray:
    positive = z >= 0
    out = np.empty_like(z, dtype=float)
    out[positive] = 1 / (1 + np.exp(-z[positive]))
    exp_z = np.exp(z[~positive])
    out[~positive] = exp_z / (1 + exp_z)
    return out


def logistic_loss_and_gradients(X, y, w, b, l2=0.0):
    y = np.asarray(y, dtype=float)
    n = len(y)
    logits = X @ w + b
    p = sigmoid(logits)
    eps = 1e-12
    data_loss = -np.mean(y * np.log(p + eps) + (1 - y) * np.log(1 - p + eps))
    loss = data_loss + 0.5 * l2 * np.dot(w, w)
    error = p - y
    dw = X.T @ error / n + l2 * w
    db = error.mean()
    return loss, dw, db
```

Градиент-check: для `w[j]` посчитать `(L(w[j] + eps) - L(w[j] - eps)) / (2 * eps)` и сравнить с `dw[j]`.

### X5-LC2. Метрики, порог и ROC-AUC

Не надо передавать бинарные labels в ROC-AUC: метрике нужны непрерывные `scores`. Пороги перебираем группами одинаковых score, чтобы tie не зависел от порядка объектов.

```python
def metrics(y_true, labels):
    tp = sum(y == 1 and p == 1 for y, p in zip(y_true, labels))
    fp = sum(y == 0 and p == 1 for y, p in zip(y_true, labels))
    fn = sum(y == 1 and p == 0 for y, p in zip(y_true, labels))
    precision = tp / (tp + fp) if tp + fp else 0.0
    recall = tp / (tp + fn) if tp + fn else 0.0
    f1 = 2 * precision * recall / (precision + recall) if precision + recall else 0.0
    return precision, recall, f1


def roc_auc(y_true, scores):
    pos, neg = sum(y_true), len(y_true) - sum(y_true)
    if pos == 0 or neg == 0:
        raise ValueError("ROC-AUC requires both classes")
    ordered = sorted(zip(scores, y_true), reverse=True)
    tp = fp = 0
    points = [(0.0, 0.0)]
    i = 0
    while i < len(ordered):
        score = ordered[i][0]
        while i < len(ordered) and ordered[i][0] == score:
            tp += ordered[i][1]
            fp += 1 - ordered[i][1]
            i += 1
        points.append((fp / neg, tp / pos))
    return sum((x2 - x1) * (y1 + y2) / 2 for (x1, y1), (x2, y2) in zip(points, points[1:]))


def evaluate_thresholds(y_true, scores):
    candidates = sorted(set(scores), reverse=True)
    best = None
    for threshold in candidates:
        labels = [int(s >= threshold) for s in scores]
        precision, recall, f1 = metrics(y_true, labels)
        row = {"threshold": threshold, "precision": precision, "recall": recall, "f1": f1}
        if best is None or row["f1"] > best["f1"]:
            best = row
    best["roc_auc"] = roc_auc(y_true, scores)
    return best
```

### X5-LC3. Time split без leakage

Честный split требует: `train_end < validation_start`, зазор между ними и исключение пользователей, присутствующих в validation, из train. Ниже базовый вариант; на практике точки валидации выбирают по бизнес-окну, а не равномерно «для красоты».

```python
import pandas as pd


def time_group_splits(df, n_splits, validation_days, gap_days):
    data = df.copy()
    data["event_dt"] = pd.to_datetime(data["event_dt"])
    dates = pd.date_range(
        data["event_dt"].min() + pd.Timedelta(days=gap_days + validation_days),
        data["event_dt"].max() - pd.Timedelta(days=validation_days),
        periods=n_splits,
    )
    for valid_start in dates:
        valid_end = valid_start + pd.Timedelta(days=validation_days)
        train = data[data["event_dt"] < valid_start - pd.Timedelta(days=gap_days)]
        valid = data[(data["event_dt"] >= valid_start) & (data["event_dt"] < valid_end)]
        valid_users = set(valid["user_id"])
        train = train[~train["user_id"].isin(valid_users)]
        if not train.empty and not valid.empty:
            yield train, valid
```

Если после group-purge валидация мала, не «доклеиваем» будущие данные: увеличиваем период, уменьшаем число фолдов или используем group-level split с явно согласованной единицей эксперимента.

### X5-LC4. Лучший decision stump

Gini измеряет неоднородность листа:

$$
\Large G = 1 - \sum_{k} p_k^2.
$$

Для бинарного класса это `1 - p_0^2 - p_1^2`; score split - взвешенная сумма Gini левого и правого листа.

```python
def gini(n_pos, n_total):
    if n_total == 0:
        return 0.0
    p = n_pos / n_total
    return 2 * p * (1 - p)


def best_stump(x, y):
    pairs = sorted(zip(x, y))
    total_pos = sum(y)
    left_n = left_pos = 0
    best = None
    for i in range(len(pairs) - 1):
        value, label = pairs[i]
        left_n += 1
        left_pos += label
        next_value = pairs[i + 1][0]
        if value == next_value:
            continue
        right_n = len(pairs) - left_n
        right_pos = total_pos - left_pos
        score = (left_n * gini(left_pos, left_n) + right_n * gini(right_pos, right_n)) / len(pairs)
        if best is None or score < best["gini"]:
            best = {
                "threshold": (value + next_value) / 2,
                "left_label": int(left_pos * 2 >= left_n),
                "right_label": int(right_pos * 2 >= right_n),
                "gini": score,
            }
    if best is None:
        raise ValueError("Need at least two distinct feature values")
    return best
```

Сортировка даёт `O(n log n)`, один проход по границам - `O(n)`.

### X5-LC5. Таргет следующего дерева в GB

Для бинарного LogLoss при raw-score `F`:

$$
\Large p_i = \sigma(F_i), \qquad -\frac{\partial L}{\partial F_i} = y_i - p_i.
$$

`y_i - p_i` - антиградиент, он и служит target следующего дерева. В XGBoost дополнительно используются вторые производные `h_i = p_i(1-p_i)` для ньютоновского шага.

```python
import numpy as np


def boosting_targets(y, raw_scores):
    y = np.asarray(y, dtype=float)
    raw_scores = np.asarray(raw_scores, dtype=float)
    p = sigmoid(raw_scores)
    negative_gradient = y - p
    hessian = p * (1 - p)
    return p, negative_gradient, hessian
```

Следующее дерево не должно учить `y`: текущая модель уже объяснила часть сигнала. Дерево учит именно оставшуюся ошибку в пространстве выбранного loss.

### X5-LC6. Bootstrap для разницы средних

```python
import random


def bootstrap_delta(control, treatment, n_boot=10_000, seed=42):
    if not control or not treatment:
        raise ValueError("Both groups must be non-empty")
    rng = random.Random(seed)
    observed = sum(treatment) / len(treatment) - sum(control) / len(control)
    deltas = []
    for _ in range(n_boot):
        c = [rng.choice(control) for _ in control]
        t = [rng.choice(treatment) for _ in treatment]
        deltas.append(sum(t) / len(t) - sum(c) / len(c))
    deltas.sort()
    return {
        "delta": observed,
        "ci_95": (deltas[int(0.025 * n_boot)], deltas[int(0.975 * n_boot)]),
    }
```

Если CI пересекает ноль, данных недостаточно, чтобы заявить эффект выбранного уровня доверия. Это не доказательство отсутствия эффекта; нужно также смотреть power и минимально значимый для бизнеса эффект.

## 🔎 Code review

### X5-CR1. `loss_and_grad`

Проблемы: overflow в `exp`, `log(0)`, сумма вместо среднего, L2 на bias, несогласованный коэффициент регуляризации и отсутствие проверки форм. Исправление - использовать реализацию из X5-LC1: средний loss, `0.5 * l2 * ||w||^2`, `l2 * w` в градиенте и устойчивый sigmoid.

### X5-CR2. Leakage в preprocessing

`median`, target encoding и scaler обучены на всём `df` до split. Это даёт validation информацию о будущем. Безопасный порядок: split -> `fit` статистик на train -> `transform` train/valid. Для target encoding на train нужны out-of-fold значения; для valid берётся mapping, обученный на полном train, с smoothing и fallback в global mean.

### X5-CR3. ROC-AUC по labels

ROC-AUC по `labels` теряет ранжирующую информацию и превращается в одну рабочую точку. Нужны raw probabilities/scores, `zero_division` для Precision/Recall, PR-AUC при сильном дисбалансе, calibration/reliability curve и метрики на важных сегментах. Порог `0.5` выбирают только по стоимости ошибок и validation, а не по привычке.

### X5-CR4. CV для кредитных заявок

`KFold(shuffle=True)` смешивает прошлое и будущее, а один клиент может попасть в разные фолды. Нужен rolling/expanding time split, при необходимости temporal gap и group purge по `client_id`; все feature-statistics и tuning выполняются внутри каждого train fold. Финальный holdout должен быть самым поздним периодом.
