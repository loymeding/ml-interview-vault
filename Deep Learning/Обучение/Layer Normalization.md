---
tags: [deep-learning, normalization, layernorm, rmsnorm, transformer, training-stability]
тип: теория
уровень: middle
сложность: средняя
статус: готово
готовность: 90
создано: 2026-06-01
источники:
  - "Layer Normalization — Ba, Kiros, Hinton"
  - "Batch Normalization"
  - "Архитектура Transformer — энкодер, декодер и слои"
предпосылки:
  - "Метод обратного распространения ошибки (Backpropagation)"
  - "Batch Normalization"
связано:
  - "Batch Normalization"
  - "Архитектура Transformer — энкодер, декодер и слои"
  - "Эволюция и архитектура LLaMA"
сравнить-с:
  - "Batch Normalization"
---

# Layer Normalization

> [!abstract] Суть
> **Layer Normalization (LayerNorm)** нормирует признаки **внутри одного объекта**. В Transformer это обычно значит: для каждого токена отдельно берём его hidden vector, считаем среднее и дисперсию по hidden dimension, приводим масштаб к стабильному виду и затем применяем обучаемые `scale` и `shift`. В отличие от BatchNorm, LayerNorm не зависит от размера батча и одинаково работает на train и inference.

---

## 1. Зачем нужна нормализация

Во время обучения нейросети распределения активаций постоянно меняются: веса обновились → выход слоя изменил масштаб → следующему слою снова нужно подстраиваться. Если активации становятся слишком большими или слишком маленькими, обучение становится нестабильным:

- градиенты могут взрываться или затухать;
- оптимизатору сложнее выбрать стабильный learning rate;
- глубокие residual-сети хуже передают сигнал;
- при mixed precision легче поймать `NaN` или underflow/overflow.

Нормализация стабилизирует масштаб hidden states. Это не “магическая таблетка”, но она расширяет область гиперпараметров, при которых модель вообще обучается.

---

## 2. Что именно нормирует LayerNorm

Пусть у нас есть один объект или один токен с hidden vector:

$$\Large x = [x_1, x_2, \ldots, x_d]$$

где:
- $x$ — вектор признаков одного объекта/токена;
- $d$ — размер hidden dimension, например `d_model=768` у BERT-base.

LayerNorm считает среднее по признакам этого же вектора:

$$\Large \mu = \frac{1}{d}\sum_{i=1}^{d}x_i$$

Затем дисперсию:

$$\Large \sigma^2 = \frac{1}{d}\sum_{i=1}^{d}(x_i-\mu)^2$$

Нормализует каждый компонент:

$$\Large \hat{x}_i = \frac{x_i-\mu}{\sqrt{\sigma^2+\epsilon}}$$

И применяет обучаемые параметры масштаба и сдвига:

$$\Large y_i = \gamma_i\hat{x}_i + \beta_i$$

где:
- $\mu$ — среднее по признакам одного вектора;
- $\sigma^2$ — дисперсия по признакам одного вектора;
- $\epsilon$ — маленькая константа для численной стабильности;
- $\hat{x}_i$ — нормализованное значение;
- $\gamma_i$ — обучаемый scale для признака $i$;
- $\beta_i$ — обучаемый shift для признака $i$;
- $y_i$ — выход LayerNorm.

> [!important] Главный смысл
> LayerNorm не сравнивает объект с другими объектами батча. Он нормирует каждый объект сам по себе.

---

## 3. Пошаговый числовой пример

Пусть hidden vector одного токена:

$$\Large x = [2,\;4,\;6,\;8]$$

### Шаг 1. Среднее

$$\Large \mu = \frac{2+4+6+8}{4} = 5$$

### Шаг 2. Отклонения от среднего

$$\Large x-\mu = [-3,\;-1,\;1,\;3]$$

### Шаг 3. Дисперсия

$$\Large \sigma^2 = \frac{(-3)^2+(-1)^2+1^2+3^2}{4} = \frac{20}{4}=5$$

### Шаг 4. Стандартное отклонение

Если временно игнорировать $\epsilon$:

$$\Large \sigma = \sqrt{5} \approx 2.236$$

### Шаг 5. Нормализованный вектор

$$\Large \hat{x}
=
\left[
\frac{-3}{2.236},
\frac{-1}{2.236},
\frac{1}{2.236},
\frac{3}{2.236}
\right]
\approx
[-1.34,\;-0.45,\;0.45,\;1.34]
$$

У $\hat{x}$ среднее примерно `0`, дисперсия примерно `1`.

### Шаг 6. Scale & shift

Если $\gamma=[1,1,1,1]$ и $\beta=[0,0,0,0]$, выход равен нормализованному вектору.

Если модель выучит:

$$\Large \gamma=[2,2,2,2], \qquad \beta=[0.5,0.5,0.5,0.5]$$

то:

$$\Large y = 2\hat{x}+0.5 \approx [-2.18,\;-0.39,\;1.39,\;3.18]$$

**Зачем это нужно:** нормализация стабилизирует масштаб, но $\gamma$ и $\beta$ позволяют модели вернуть удобный масштаб и сдвиг, если это полезно для задачи.

---

## 4. Как это выглядит в Transformer

В Transformer hidden tensor обычно имеет форму:

$$\Large X \in \mathbb{R}^{B \times T \times d_{\text{model}}}$$

где:
- $B$ — batch size;
- $T$ — длина последовательности;
- $d_{\text{model}}$ — размер hidden vector каждого токена.

LayerNorm применяется отдельно к каждому `X[b, t, :]`:

```text
batch b=0
token t=0: LayerNorm по hidden dimension
token t=1: LayerNorm по hidden dimension
token t=2: LayerNorm по hidden dimension
...
```

То есть LayerNorm:
- не зависит от других примеров в batch;
- не зависит от других позиций напрямую;
- не требует running mean/running variance;
- одинаково работает при `batch_size=1` и `batch_size=512`;
- хорошо подходит для autoregressive inference.

---

## 5. LayerNorm vs BatchNorm

| Метод | По чему считает статистики | Зависит от batch size | Train vs inference | Типичные области |
|---|---|---:|---|---|
| **BatchNorm** | по батчу для каждого признака/канала | да | на inference использует running stats | CNN, большие батчи |
| **LayerNorm** | по признакам одного объекта/токена | нет | одинаковая логика train/inference | Transformers, RNN, LLM |

### Почему BatchNorm плох для Transformer

В NLP и LLM часто есть:
- переменная длина последовательностей;
- маленький batch size из-за больших моделей;
- autoregressive inference по одному запросу;
- разные распределения токенов на разных позициях;
- padding.

BatchNorm зависит от статистики батча, поэтому поведение при train и inference отличается. LayerNorm считает статистики внутри каждого токена, поэтому стабильнее для последовательностей.

### Где BatchNorm всё ещё хорош

BatchNorm отлично работает в CNN, когда:
- batch size достаточно большой;
- изображения одного домена;
- каналы имеют стабильные статистики;
- важна дополнительная регуляризация через шум batch statistics.

---

## 6. Где ставят LayerNorm: Post-LN и Pre-LN

В Transformer есть подслои:
- attention;
- feed-forward network.

И есть residual connection:

$$\Large x + \text{Sublayer}(x)$$

### Post-LN

Оригинальный Transformer использовал Post-LN:

$$\Large x_{l+1} = \text{LayerNorm}(x_l + \text{Sublayer}(x_l))$$

Плюсы:
- классическая схема оригинальной статьи;
- иногда даёт хорошее качество при аккуратном warmup и настройке.

Минусы:
- в очень глубоких сетях градиент проходит через много LayerNorm;
- обучение может быть нестабильнее.

### Pre-LN

Многие современные GPT/LLM используют Pre-LN:

$$\Large x_{l+1} = x_l + \text{Sublayer}(\text{LayerNorm}(x_l))$$

Плюсы:
- residual stream остаётся более прямым каналом;
- градиенты легче проходят через глубокую сеть;
- проще обучать очень глубокие Transformer.

Минусы:
- часто нужен final normalization перед выходными logits;
- динамика качества может отличаться от Post-LN.

> [!tip] Интервьюерская формулировка
> Post-LN нормализует уже смешанный residual output. Pre-LN нормализует вход в подслой, а residual stream остаётся почти прямой магистралью для сигнала и градиента.

Подробнее: [[NLP/Модели и архитектуры/Архитектура Transformer — энкодер, декодер и слои]]

---

## 7. Варианты и близкие подходы

### RMSNorm

RMSNorm похож на LayerNorm, но не вычитает среднее. Он контролирует масштаб через root mean square:

$$\Large \text{RMS}(x) = \sqrt{\frac{1}{d}\sum_{i=1}^{d}x_i^2+\epsilon}$$

$$\Large \text{RMSNorm}(x)_i = \frac{x_i}{\text{RMS}(x)}\gamma_i$$

Плюсы:
- дешевле LayerNorm;
- проще;
- хорошо работает в больших decoder-only LLM;
- используется в LLaMA-подобных архитектурах.

Минус:
- не центрирует вектор, то есть не убирает среднее.

Подробнее: [[NLP/LLM и Промпт-инжиниринг/Эволюция и архитектура LLaMA]]

### ScaleNorm

ScaleNorm нормирует весь вектор по его L2-норме и умножает на один обучаемый scalar:

$$\Large \text{ScaleNorm}(x) = g \cdot \frac{x}{\|x\|_2}$$

где:
- $g$ — обучаемый scalar;
- $\|x\|_2$ — L2-норма вектора.

Идея похожа: контролировать масштаб hidden state. Используется реже, чем LayerNorm/RMSNorm.

### GroupNorm

GroupNorm чаще встречается в computer vision. Он делит каналы одного объекта на группы и нормирует внутри групп:
- не зависит от batch size;
- полезен, когда изображения большие и batch маленький;
- часто применяется в segmentation/detection.

### InstanceNorm

InstanceNorm нормирует каждый объект и канал отдельно, часто по spatial dimensions. Исторически популярен в style transfer.

---

## 8. Когда использовать LayerNorm

LayerNorm обычно выбирают, когда:
- модель работает с последовательностями;
- batch size маленький или нестабильный;
- inference может быть по одному объекту;
- архитектура residual/Transformer-like;
- важна одинаковая логика train и inference;
- BatchNorm даёт плохие running statistics.

Типичные случаи:
- Transformer encoder/decoder;
- BERT/GPT-like модели;
- RNN/LSTM/GRU;
- speech/time-series модели;
- MLP с маленькими батчами;
- RL, где батчи нерепрезентативны.

---

## 9. PyTorch-пример

```python
import torch
import torch.nn as nn

x = torch.tensor([[2.0, 4.0, 6.0, 8.0]])  # shape: [batch=1, features=4]

ln = nn.LayerNorm(normalized_shape=4, elementwise_affine=True)

with torch.no_grad():
    ln.weight.fill_(1.0)  # gamma
    ln.bias.fill_(0.0)    # beta

y = ln(x)
print(y)
# tensor([[-1.3416, -0.4472,  0.4472,  1.3416]])
```

Для Transformer:

```python
x = torch.randn(batch_size, seq_len, d_model)
ln = nn.LayerNorm(d_model)
y = ln(x)  # нормализация по последней оси: d_model
```

Важно: `normalized_shape=d_model` означает, что статистики считаются по последнему измерению. Для тензора `[B, T, d_model]` это как раз hidden vector каждого токена.

---

## 10. Типичные ошибки

- **Путать LayerNorm и BatchNorm:** LayerNorm нормирует признаки одного объекта; BatchNorm нормирует признак по батчу.
- **Думать, что LayerNorm использует running statistics:** нет, у него нет `running_mean` и `running_var`, в отличие от BatchNorm.
- **Считать, что LayerNorm смешивает токены:** нет, он нормирует hidden vector каждого токена отдельно. Токены смешиваются в attention.
- **Убирать $\gamma$ и $\beta$ без причины:** affine-параметры позволяют модели восстановить нужный масштаб и сдвиг.
- **Не понимать Pre-LN vs Post-LN:** это не мелкая перестановка; она влияет на стабильность глубоких Transformer.
- **Ожидать регуляризации как у BatchNorm:** LayerNorm не шумит статистиками батча, поэтому регуляризационный эффект обычно слабее.
- **Ставить нормализацию “куда попало”:** в residual architectures порядок `Norm → Sublayer → Add` или `Sublayer → Add → Norm` меняет динамику обучения.

---

## 11. Каверзные вопросы

> [!question] Почему LayerNorm хорош при batch size = 1?
> Потому что он считает статистики внутри одного объекта/токена, а не по batch dimension. Даже если в батче один пример, у токена всё равно есть hidden dimension, по которой можно посчитать mean и variance.

> [!question] Почему LayerNorm используют в Transformer?
> Transformer работает с последовательностями переменной длины, часто обучается на маленьких effective batch и используется на inference по одному запросу. LayerNorm не зависит от batch statistics и стабилизирует hidden state каждого токена отдельно.

> [!question] Можно ли заменить LayerNorm на BatchNorm в Transformer?
> Технически можно экспериментировать, но обычно это ухудшает стабильность: BatchNorm зависит от batch statistics, padding, длины последовательности и режима train/inference. Поэтому стандарт — LayerNorm или RMSNorm.

> [!question] Чем RMSNorm отличается от LayerNorm?
> RMSNorm не вычитает среднее, а только делит на RMS-масштаб. Он дешевле и часто достаточно стабилен для LLM, но не центрирует активации.

---

## 12. Короткий ответ для собеседования

> [!quote]
> LayerNorm нормирует hidden vector одного объекта или одного токена по его feature dimension: считает среднее и дисперсию внутри этого вектора, приводит масштаб к стабильному виду и затем применяет обучаемые $\gamma$ и $\beta$. В Transformer это удобно, потому что нормализация не зависит от batch size, padding и длины последовательности. Поэтому LayerNorm стабильно работает и на train, и на inference, включая `batch_size=1`. В глубоких Transformer часто используют Pre-LN, где нормировка стоит перед attention/FFN-блоком, чтобы residual stream оставался прямым каналом для градиента.

---

## Проверка себя

- По какой оси LayerNorm считает среднее и дисперсию в тензоре `[B, T, d_model]`?
- Почему LayerNorm не требует `model.eval()`-специфичных running statistics?
- Чем LayerNorm отличается от BatchNorm на inference?
- Почему Pre-LN стабильнее для глубоких Transformer?
- Чем RMSNorm отличается от LayerNorm?

---

## Связано

- [[Deep Learning/Обучение/Batch Normalization]]
- [[NLP/Модели и архитектуры/Архитектура Transformer — энкодер, декодер и слои]]
- [[NLP/LLM и Промпт-инжиниринг/Эволюция и архитектура LLaMA]]
- [[NLP/LLM и Промпт-инжиниринг/GPT-2 — Zero-Shot Learning и начало эры промптов]]
- [[NLP/🧭 Карта NLP и LLM]]

---

## Источники

- [Layer Normalization — Ba, Kiros, Hinton](https://arxiv.org/abs/1607.06450)
- [Attention Is All You Need — Vaswani et al.](https://arxiv.org/abs/1706.03762)
- [Root Mean Square Layer Normalization — Zhang, Sennrich](https://arxiv.org/abs/1910.07467)

---

[[Deep Learning/🏠 Главная|Назад к Deep Learning]]
