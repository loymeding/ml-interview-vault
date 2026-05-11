---
tags: [deep-learning, оптимизация, sgd, momentum, adam, rmsprop, lr-scheduler, pytorch]
тип: теория
уровень: middle
сложность: средняя
статус: готово
готовность: 95
создано: 2026-04-27
источники:
  - "Интенсив Основы глубокого обучения 4 Радослав Нейчев MADE"
  - "Семинар. PyTorch. Оптимизаторы"
  - "Алгоритмы градиентного спуска"
предпосылки:
  - "Градиентный спуск"
  - "Метод обратного распространения ошибки (Backpropagation)"
связано:
  - "Регуляризация в DL — Dropout"
  - "Batch Normalization"
сравнить-с: []
---

# Продвинутые методы оптимизации (от SGD до AdamW)

> [!abstract] Суть
> Чистый SGD плохо справляется со сложными поверхностями (седловыми точками и узкими «оврагами»). Momentum накапливает историю шагов, работая как тяжёлый катящийся шар. RMSprop адаптирует learning rate для каждого параметра. Adam объединяет оба подхода и является золотым стандартом. Для финальной сходимости применяется динамическое расписание шага (LR Schedulers).

## Ответ

### 1. Проблема классического SGD

Базовый SGD: $\theta = \theta - \eta \nabla L$. На «оврагах» он осциллирует (прыгает от стенки к стенке), медленно продвигаясь к минимуму. Легко застревает в седловых точках.

### 2. Momentum (Метод инерции)

$$\Large v_t = \beta v_{t-1} + (1 - \beta)\nabla L$$
$$\Large \theta = \theta - \eta v_t$$

где:
- $v_t$ — накопленный «импульс» (скользящее среднее прошлых градиентов)
- $\beta$ — коэффициент инерции (обычно $0.9$)
- $\eta$ — learning rate

**Эффект:** по осям с осцилляциями импульс гасит себя; по устойчивому направлению — накапливается.

### 3. RMSprop

Адаптирует learning rate для каждого параметра: делит шаг на корень из среднего квадрата прошлых градиентов. Это дешёвая аппроксимация методов второго порядка (матрицы Гессе).

### 4. Adam (Adaptive Moment Estimation)

Объединяет Momentum (первый момент $m_t$) и RMSprop (второй момент $v_t$). Стандарт индустрии.

### 5. Learning Rate Schedulers

Если $\eta$ константен, модель бесконечно осциллирует вокруг минимума. Schedulers плавно снижают $\eta$ (StepLR, CosineAnnealing).

### 6. Mixed Precision Training (AMP)

Матричные умножения в **FP16** выполняются на современных GPU в 2–3 раза быстрее при вдвое меньшем расходе VRAM. Проблема: FP16 имеет узкий динамический диапазон — маленькие градиенты при Backward Pass просто округляются до нуля (Gradient Underflow) и веса перестают обновляться.

**Идея Automatic Mixed Precision (AMP):**
- Тяжёлые операции (Forward + Backward) → в **FP16**.
- Мастер-веса оптимизатора → в **FP32** (чтобы не терять крошечные шаги обновления).
- **Loss Scaling:** перед `.backward()` лосс умножают на большое число ($2^{16}$), градиенты масштабируются вместе с ним и попадают в безопасную зону FP16; перед шагом оптимизатора делят обратно.

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()  # Динамически управляет масштабом лосса

for inputs, labels in dataloader:
    optimizer.zero_grad()
    with autocast():                       # Forward в FP16
        outputs = model(inputs)
        loss = criterion(outputs, labels)
    scaler.scale(loss).backward()          # Backward с масштабированием
    scaler.step(optimizer)                 # Шаг весов (в FP32)
    scaler.update()                        # Обновляет scale-фактор
```

## Формула / Схема

**Momentum:**

$$\Large v_t = \beta v_{t-1} + (1-\beta)g_t, \qquad \theta \leftarrow \theta - \eta v_t$$

**Adam:**

$$\Large \theta \leftarrow \theta - \frac{\eta \hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

где:
- $\hat{m}_t$ — скорректированное скользящее среднее градиента (первый момент)
- $\hat{v}_t$ — скорректированное скользящее среднее квадрата градиента (второй момент)
- $\epsilon$ — малая константа для стабильности ($\sim 10^{-8}$)

**Weight Decay** в PyTorch: $\theta = (1 - \lambda)\theta - \eta \nabla L$.

## Короткий пример

```python
import torch.optim as optim

optimizer = optim.Adam(model.parameters(), lr=1e-3, weight_decay=1e-4)
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)

for epoch in range(100):
    for batch in dataloader:
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
    scheduler.step()  # Один раз в ЭПОХУ, не в батче!
```

## Типичные ошибки

- **`scheduler.step()` внутри цикла по батчам:** LR обнулится за первую эпоху.
- **Забыть про VRAM для Adam:** Adam хранит два момента для каждого параметра → потребление памяти в **3 раза** больше, чем у SGD.

## Каверзные вопросы

> [!question] Модель на SGD застряла на loss $0.5$. Поможет ли замена на Adam?
> Не факт. Adam быстрее на старте, но часто сходится к менее обобщающим «острым» оптимумам. Коллеге сначала стоит применить LR Scheduler для SGD.

## Проверка себя

- За счёт чего Momentum «выскакивает» из локальных ям?
- Какую проблему решает деление на $\sqrt{\hat{v}_t}$ в Adam?
- Как добавить L2-регуляризацию в PyTorch?

## Предпосылки

- [[Градиентный спуск]]
- [[Метод обратного распространения ошибки (Backpropagation)]]

## Связано

- [[Регуляризация в DL — Dropout]]
- [[Batch Normalization]]

## Источники

- Интенсив Основы глубокого обучения 4 Радослав Нейчев MADE
- Семинар. PyTorch. Оптимизаторы
- Алгоритмы градиентного спуска

---
[[🗺️ Индекс|Назад к разделу]]