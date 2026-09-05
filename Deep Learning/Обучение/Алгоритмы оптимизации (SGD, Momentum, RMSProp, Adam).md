---
tags: [deep-learning, оптимизация, sgd, momentum, adam, adamw, lamb, lans, rmsprop, lr-scheduler, pytorch]
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
  - "Decoupled Weight Decay Regularization — Loshchilov, Hutter"
  - "Large Batch Optimization for Deep Learning: Training BERT in 76 minutes — You et al."
  - "Accelerated Large Batch Optimization of BERT Pretraining in 54 minutes — Zheng et al."
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
> Чистый SGD плохо справляется со сложными поверхностями (седловыми точками и узкими «оврагами»). Momentum накапливает историю шагов, RMSprop адаптирует learning rate для каждого параметра, Adam объединяет оба подхода. В современных Transformer/LLM чаще используют **AdamW**, потому что он корректно отделяет weight decay от адаптивного шага Adam. Для очень больших distributed batch существуют layer-wise оптимизаторы вроде **LAMB** и исследовательские варианты вроде **LANS**.

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

Разные параметры имеют разную кривизну loss-функции, разные масштабы градиентов. Идея RMSprop в том, чтобы задавать отдельный adaptive learning rate для каждого весового параметра.
Для этого мы будем накапливать среднее значение квадратов прошлых градиентов.
Адаптирует learning rate для каждого параметра: делит шаг на корень из среднего квадрата прошлых градиентов. Это дешёвая аппроксимация методов второго порядка (матрицы Гессе).

Если градиенты много итераций большие, то шаг уменьшается, если долго маленькие, то шаг увеличивается.

### 4. Adam (Adaptive Moment Estimation)

Объединяет Momentum (первый момент $m_t$) и RMSprop (второй момент $v_t$). Стандарт индустрии.

Пусть:

$$\Large g_t = \nabla_{\theta}L_t(\theta_t)$$

где:
- $g_t$ — градиент loss по параметрам на шаге $t$
- $\theta_t$ — параметры модели на шаге $t$
- $L_t$ — loss на текущем mini-batch

Adam хранит два экспоненциальных скользящих средних:

$$\Large m_t = \beta_1 m_{t-1} + (1-\beta_1)g_t$$

$$\Large v_t = \beta_2 v_{t-1} + (1-\beta_2)(g_t \odot g_t)$$

где:
- $m_t$ — первый момент, сглаженное направление градиента; это momentum-часть
- $v_t$ — второй момент, сглаженный квадрат градиента; это RMSProp-часть
- $\beta_1$ — скорость забывания для первого момента, часто `0.9`
- $\beta_2$ — скорость забывания для второго момента, часто `0.999`
- $\odot$ — поэлементное умножение

Так как $m_0 = 0$ и $v_0 = 0$, в начале оценки смещены к нулю. Поэтому используют bias correction:

$$\Large \hat{m}_t = \frac{m_t}{1-\beta_1^t}, \qquad \hat{v}_t = \frac{v_t}{1-\beta_2^t}$$

И сам шаг:

$$\Large \theta_{t+1} = \theta_t - \eta_t \frac{\hat{m}_t}{\sqrt{\hat{v}_t}+\epsilon}$$

где:
- $\eta_t$ — текущий learning rate, возможно после scheduler
- $\epsilon$ — маленькая константа для численной стабильности, часто `1e-8`
- $\frac{1}{\sqrt{\hat{v}_t}+\epsilon}$ — адаптивное поэлементное масштабирование шага

Интуиция: если у параметра долго большие градиенты, Adam уменьшает для него эффективный шаг; если градиенты маленькие, шаг относительно увеличивается.

### 4.1 AdamW (Adam with Decoupled Weight Decay)

> [!important] Главное отличие
> **AdamW — это не просто Adam + L2-регуляризация.** В AdamW weight decay применяется отдельно к весам, а не добавляется внутрь градиента перед адаптивным масштабированием.

Если добавить L2-регуляризацию в обычный Adam через градиент, получится:

$$\Large g_t^{L2} = \nabla_{\theta}L_t(\theta_t) + \lambda\theta_t$$

где:
- $g_t^{L2}$ — градиент loss вместе с L2-штрафом
- $\lambda$ — коэффициент регуляризации
- $\theta_t$ — текущие веса

Проблема: Adam потом делит весь $g_t^{L2}$ на $\sqrt{\hat{v}_t}+\epsilon$. Значит, регуляризация тоже становится **адаптивной по координатам**: один вес штрафуется сильнее, другой слабее, потому что у них разные истории градиентов. В SGD L2 и weight decay почти эквивалентны, а в Adam — уже нет.

AdamW отделяет shrinkage весов от направления градиентного шага:

$$\Large u_t = \frac{\hat{m}_t}{\sqrt{\hat{v}_t}+\epsilon}$$

$$\Large \theta_{t+1} = (1-\eta_t\lambda)\theta_t - \eta_t u_t$$

где:
- $u_t$ — Adam-направление, полученное только из градиента loss
- $(1-\eta_t\lambda)\theta_t$ — отдельное уменьшение нормы весов
- $\lambda$ — weight decay
- $\eta_t$ — learning rate

То есть AdamW делает две разные вещи:
1. **Adam-шаг:** идёт в сторону уменьшения loss.
2. **Weight decay:** слегка тянет веса к нулю пропорционально их величине.

**Практическая интерпретация:** AdamW обычно даёт более стабильную регуляризацию и лучшее обобщение, особенно в Transformer, BERT-like моделях, LLM fine-tuning и CV-моделях с Adam-family optimizer.

**Что обычно не decay-ят:**
- bias-параметры
- веса `LayerNorm`, `BatchNorm`, `RMSNorm`
- иногда embedding-таблицы и output head, если есть причина сохранить масштаб представлений

Пример разделения параметров в PyTorch:

```python
import torch

decay_params = []
no_decay_params = []

for name, param in model.named_parameters():
    if not param.requires_grad:
        continue
    if name.endswith("bias") or "norm" in name.lower():
        no_decay_params.append(param)
    else:
        decay_params.append(param)

optimizer = torch.optim.AdamW(
    [
        {"params": decay_params, "weight_decay": 0.01},
        {"params": no_decay_params, "weight_decay": 0.0},
    ],
    lr=2e-5,
    betas=(0.9, 0.999),
    eps=1e-8,
)
```

**Типичные настройки:**
- BERT fine-tuning: `lr=1e-5...5e-5`, `weight_decay=0.01`, warmup `5-10%`
- обучение Transformer/LLM с нуля: `betas=(0.9, 0.95)` или близко к этому, `weight_decay=0.01...0.1`, cosine schedule + warmup
- small tabular/network tasks: часто достаточно AdamW с `lr=1e-3` и небольшим decay

### 4.2 LAMB (Layer-wise Adaptive Moments)

LAMB — это оптимизатор для **large-batch distributed training**. Его смысл: AdamW хорошо адаптирует шаг по координатам, но при огромном batch size разные слои могут получать обновления слишком разного относительного масштаба. LAMB добавляет **layer-wise trust ratio**: каждый слой получает шаг, соразмерный норме его собственных весов.

Разобьём параметры на блоки/слои $\mathcal{G}_b$:

$$\Large \theta_t = [\theta_{t,\mathcal{G}_1}, \theta_{t,\mathcal{G}_2}, \ldots, \theta_{t,\mathcal{G}_B}]$$

где:
- $\mathcal{G}_b$ — блок параметров, например матрица весов слоя
- $B$ — число блоков
- $\theta_{t,\mathcal{G}_b}$ — параметры блока $b$

Сначала LAMB считает Adam-подобные моменты по блоку:

$$\Large m_{t,\mathcal{G}_b} = \beta_1m_{t-1,\mathcal{G}_b} + (1-\beta_1)g_{t,\mathcal{G}_b}$$

$$\Large v_{t,\mathcal{G}_b} = \beta_2v_{t-1,\mathcal{G}_b} + (1-\beta_2)g_{t,\mathcal{G}_b}^2$$

$$\Large r_{t,\mathcal{G}_b} = \frac{\hat{m}_{t,\mathcal{G}_b}}{\sqrt{\hat{v}_{t,\mathcal{G}_b}}+\epsilon}$$

Дальше формируется направление обновления с weight decay:

$$\Large u_{t,\mathcal{G}_b} = r_{t,\mathcal{G}_b} + \lambda\theta_{t,\mathcal{G}_b}$$

И считается trust ratio:

$$\Large \rho_{t,\mathcal{G}_b} = \frac{\phi(\|\theta_{t,\mathcal{G}_b}\|_2)}{\|u_{t,\mathcal{G}_b}\|_2}$$

Итоговый шаг:

$$\Large \theta_{t+1,\mathcal{G}_b} = \theta_{t,\mathcal{G}_b} - \eta_t \rho_{t,\mathcal{G}_b}u_{t,\mathcal{G}_b}$$

где:
- $r_{t,\mathcal{G}_b}$ — Adam-направление для блока
- $u_{t,\mathcal{G}_b}$ — направление с weight decay
- $\rho_{t,\mathcal{G}_b}$ — trust ratio, коэффициент доверия к обновлению слоя
- $\phi(\cdot)$ — функция масштабирования нормы; на практике часто identity, то есть $\phi(x)=x$

**Интуиция на пальцах:** если у слоя норма весов $10$, а норма обновления $0.1$, trust ratio будет большим и шаг усилится. Если у слоя норма весов $0.01$, а обновление $1.0$, trust ratio будет маленьким и шаг притормозится. Так LAMB стабилизирует относительный размер шага по слоям.

**Где используют:**
- pretraining BERT/Transformer на очень больших batch size
- distributed training на сотнях/тысячах GPU/TPU
- ситуации, где AdamW начинает деградировать при агрессивном увеличении batch size

**Где обычно не нужен:**
- обычный fine-tuning BERT/LLM на одной GPU
- LoRA/QLoRA fine-tuning
- небольшие CNN/MLP, где batch size умеренный

Пример с внешней реализацией:

```python
# В обычном torch.optim LAMB нет.
# Часто используют DeepSpeed, NVIDIA Apex, bitsandbytes или pytorch-optimizer.

from pytorch_optimizer import Lamb

optimizer = Lamb(
    model.parameters(),
    lr=1e-3,
    betas=(0.9, 0.999),
    weight_decay=0.01,
)
```

### 4.3 LANS — не Lookahead + Noisy Student

> [!warning] Не путать
> В найденных первоисточниках **LANS** — это не `Lookahead with Noisy Student`. LANS описан как accelerated large-batch optimizer для BERT pretraining: он добавляет к LAMB **per-block gradient normalization** и **Nesterov-style momentum**.
> **Noisy Student** — это semi-supervised/self-training подход: teacher размечает unlabeled data, student обучается на pseudo-labels с шумом. Это не оптимизатор.

LANS начинает с нормализации градиента в каждом блоке:

$$\Large \tilde{g}_{t,\mathcal{G}_b} =
\frac{g_{t,\mathcal{G}_b}}{\|g_{t,\mathcal{G}_b}\|_2 + \epsilon_g}$$

где:
- $\tilde{g}_{t,\mathcal{G}_b}$ — нормализованный градиент блока
- $g_{t,\mathcal{G}_b}$ — обычный mini-batch gradient блока
- $\epsilon_g$ — маленькая константа для защиты от деления на ноль

Дальше моменты считаются уже по нормализованному градиенту:

$$\Large m_{t,\mathcal{G}_b} =
\beta_1m_{t-1,\mathcal{G}_b} + (1-\beta_1)\tilde{g}_{t,\mathcal{G}_b}$$

$$\Large v_{t,\mathcal{G}_b} =
\beta_2v_{t-1,\mathcal{G}_b} + (1-\beta_2)\tilde{g}_{t,\mathcal{G}_b}^{2}$$

Определяются два направления:

$$\Large r_{t,\mathcal{G}_b} =
\frac{\hat{m}_{t,\mathcal{G}_b}}{\sqrt{\hat{v}_{t,\mathcal{G}_b}}+\epsilon}$$

$$\Large c_{t,\mathcal{G}_b} =
\frac{\tilde{g}_{t,\mathcal{G}_b}}{\sqrt{\hat{v}_{t,\mathcal{G}_b}}+\epsilon}$$

где:
- $r_{t,\mathcal{G}_b}$ — Adam/LAMB-like направление с первым моментом
- $c_{t,\mathcal{G}_b}$ — направление по текущему нормализованному градиенту без накопленного первого момента

Упрощённо, LANS строит выпуклую смесь двух нормированных направлений:

$$\Large
d_{t,\mathcal{G}_b}
=
\phi(\|\theta_{t,\mathcal{G}_b}\|_2)
\left[
\beta_1
\frac{r_{t,\mathcal{G}_b}+\lambda\theta_{t,\mathcal{G}_b}}
{\|r_{t,\mathcal{G}_b}+\lambda\theta_{t,\mathcal{G}_b}\|_2}
+
(1-\beta_1)
\frac{c_{t,\mathcal{G}_b}+\lambda\theta_{t,\mathcal{G}_b}}
{\|c_{t,\mathcal{G}_b}+\lambda\theta_{t,\mathcal{G}_b}\|_2}
\right]
$$

$$\Large \theta_{t+1,\mathcal{G}_b} =
\theta_{t,\mathcal{G}_b} - \eta_t d_{t,\mathcal{G}_b}$$

где:
- $d_{t,\mathcal{G}_b}$ — итоговое направление LANS для блока
- первая часть смеси похожа на LAMB с momentum
- вторая часть добавляет Nesterov-like коррекцию через текущий нормализованный градиент

**Зачем это сделали:** при экстремально больших batch size простой рост learning rate перестаёт работать: LR ограничен кривизной/lipschitz-like ограничениями, и обучение начинает расходиться. LANS пытается сделать шаги более устойчивыми за счёт нормализации градиента и Nesterov-style направления. В эксперименте авторов LANS масштабировал BERT pretraining до batch size `96K/33K`, где LAMB при таком режиме расходился.

**Практический вывод:** LANS — нишевый large-batch optimizer. Для обычного обучения и fine-tuning почти всегда разумнее начать с AdamW; LAMB/LANS имеют смысл, когда реально есть distributed pretraining и проблема масштабирования batch size.

### 5. Nesterov

Проблема momentum в том, что мы сначала накапливаем скорость, потом делаем шаг. Из-за чего мы можем "сильно разогнаться", "пролетать" минимум, то есть поздно замечать, что направление изменилось.

Поэтому вместо расчета градиента в текущей точке мы можем оценивать куда нас приведет momentum и считать градиент там.

То есть сначала делаем предварительный "прыжок", считаем градиенты там и заранее понимаем как изменить направление.


### 6. Learning Rate Schedulers

Если $\eta$ константен, модель бесконечно осциллирует вокруг минимума, не имея стимула «притормозить» у дна. Schedulers плавно снижают $\eta$ по расписанию: **StepLR** (кратно делит каждые N эпох), **CosineAnnealing** (плавно снижает по форме косинуса, наиболее распространён).

**Warmup** — первые шаги обучения, когда learning rate растёт от маленького значения до целевого. Он особенно важен для Transformer и Adam: в начале моменты Adam ещё плохо оценены, веса случайны, активации нестабильны, и большой LR легко даёт NaN. Warmup даёт сети несколько сотен или тысяч шагов на стабилизацию масштаба градиентов.

**Cosine annealing** снижает LR плавно:

$$\Large \eta_t = \eta_{min} + \frac{1}{2}(\eta_{max} - \eta_{min})\left(1 + \cos\frac{\pi t}{T}\right)$$

Он удобен тем, что быстро даёт большой шаг в начале и мягко "дополировывает" решение ближе к концу обучения.

**Adam vs LR Scheduler — дополняющие механизмы, не конкурирующие:**
- **Adam** — «умная коробка передач в пространстве параметров»: *адаптирует шаг отдельно для каждого параметра* (размерность адаптации = число весов). Смотрит на историю градиента конкретного нейрона, игнорируя глобальное время.
- **LR Scheduler** — «GPS-маршрут во времени»: *изменяет единый глобальный $\eta$ для всей сети* по заранее заданному расписанию (в зависимости от эпохи/шага). Не знает ничего о конкретных параметрах.

**Золотой стандарт: Adam + CosineAnnealing.** Adam быстро находит нужный «бассейн» в пространстве потерь; косинусное расписание плавно опускает шаг, и модель точно сходится к дну этого бассейна — без лишних осцилляций.

### 6.1 Learning rate, batch size и шум градиента

На скорость сходимости влияют learning rate, batch size, инициализация, масштаб признаков, нормализация, архитектура, оптимизатор и качество данных.

Слишком большой learning rate перескакивает через хорошие области, вызывает осцилляции и иногда взрыв loss. Слишком маленький learning rate делает обучение медленным и может застревать на плато.

При увеличении batch size оценка градиента становится менее шумной. Это ускоряет вычисления на GPU, но уменьшает регуляризующий шум SGD. Large batch training часто требует:
- увеличивать learning rate примерно пропорционально batch size на старте;
- добавлять warmup;
- дольше обучать или использовать более аккуратный scheduler;
- следить за generalization gap.

**Gradient accumulation** имитирует большой batch size, накапливая градиенты за несколько mini-batch перед `optimizer.step()`. Это помогает, когда большой батч не помещается в VRAM, но не ускоряет один шаг так же, как настоящий большой батч.

### 6.2 Почему Adam быстрее, а SGD часто лучше обобщает

Adam адаптирует шаг для каждого параметра, поэтому быстро проходит плохо масштабированные направления и обычно быстрее снижает train loss. Но из-за этой адаптивности он может прийти в более "острый" минимум, где небольшое изменение весов сильно портит loss.

SGD с momentum шумнее и медленнее, зато этот шум помогает избегать узких острых областей и часто приводит к более плоским минимумам. Плоский минимум обычно устойчивее к сдвигам данных, квантизации и небольшим изменениям весов.

**Sharp minima** — область, где loss быстро растёт при малом сдвиге параметров. **Flat minima** — широкая область с похожим loss. Интервьюерская интуиция: flat minima чаще лучше обобщают.

### 6.3 Как выбирать batch size

Batch size выбирают как компромисс:
- достаточно большой, чтобы эффективно загрузить GPU;
- достаточно маленький, чтобы сохранить полезный шум градиента;
- помещается в память с учётом активаций, градиентов и состояний оптимизатора;
- не ломает BatchNorm, если он используется.

Если GPU простаивает, batch size можно увеличить. Если качество падает при large batch, стоит добавить warmup, изменить LR schedule, включить gradient accumulation или вернуться к меньшему effective batch.

### 6.4 Сводная таблица: какой оптимизатор когда выбирать

| Оптимизатор | Главная идея | Где чаще используют | Плюсы | Минусы и нюансы |
|---|---|---|---|---|
| **SGD** | простой шаг по mini-batch gradient | классика, convex/простые модели, иногда финальное CV-обучение | мало памяти, хороший baseline, шум может помогать generalization | медленно сходится, чувствителен к LR и scale признаков |
| **SGD + Momentum** | сглаживает направление через накопленный импульс | CNN, CV, обучение с большим числом эпох | часто лучше обобщает, чем Adam; дешёвый по памяти | требует аккуратного scheduler, дольше выходит на хороший loss |
| **RMSProp** | адаптивный LR через квадрат градиентов | исторически RNN/RL, нестабильные градиенты | меньше осцилляций по координатам с большим gradient scale | реже default в современных Transformer |
| **Adam** | Momentum + RMSProp | быстрый baseline почти для любой DL-задачи | быстро снижает train loss, мало ручного тюнинга | с L2/weight decay есть тонкая ловушка; иногда хуже generalization |
| **AdamW** | Adam + decoupled weight decay | Transformer, BERT, LLM, ViT, fine-tuning | стандартный modern default, лучше контролирует регуляризацию | нужно правильно исключать bias/norm из decay; важны warmup и scheduler |
| **LAMB** | Adam-like update + layer-wise trust ratio | large-batch BERT/Transformer pretraining | помогает масштабировать batch size, выравнивает относительные шаги слоёв | лишняя сложность; обычно не нужен для small/medium fine-tuning |
| **LANS** | LAMB + block gradient normalization + Nesterov-style mixture | экстремальный distributed BERT pretraining | может держать ещё больший batch size, где LAMB расходится | исследовательский/нишевый метод, мало стандартных реализаций |
| **Lookahead** | wrapper: быстрые веса делают $k$ шагов, медленные подтягиваются | иногда поверх SGD/Adam/RAdam/Ranger | снижает variance inner optimizer, может стабилизировать обучение | не заменяет выбор базового optimizer; не связан с Noisy Student |

> [!tip] Быстрое правило
> Если нет особой причины, начинай с **AdamW + warmup + cosine/linear decay**.
> Переходи к **SGD+Momentum**, если важна финальная generalization в CV и есть бюджет на долгую тренировку.
> Смотри на **LAMB/LANS**, только если у тебя действительно large-batch distributed pretraining.

### 7. Методы второго порядка (Newton, L-BFGS) и почему они редко в DL
Методы второго порядка (например, метод Ньютона) используют не только градиент (направление), но и **матрицу Гессе** (кривизну поверхности), что позволяет делать огромные и точные шаги прямо в минимум.
**Почему они редко используются в Deep Learning:**
1. **Вычислительная сложность:** Матрица Гессе имеет размер $N \times N$, где $N$ — число параметров. Для современных сетей с миллионами параметров её вычисление и обращение ($O(N^3)$) физически невозможно.
2. **Седловые точки:** В многомерных пространствах методы второго порядка склонны устремляться в седловые точки, а не в локальные минимумы (так как кривизна там тоже нулевая по некоторым осям).
3. Существуют квази-ньютоновские аппроксимации (как **L-BFGS**), которые не хранят всю матрицу, но они хорошо работают только на полном батче (Full Batch GD), что несовместимо с шумным стохастическим режимом (mini-batch) в DL.

### 8. Early Stopping как регуляризация
Ранняя остановка (Early Stopping) — это метод оптимизации и регуляризации одновременно. Обучение прерывается, когда ошибка на валидационной выборке начинает расти, несмотря на падение ошибки на трейне.
**Почему это регуляризация?** Ограничивая время обучения (количество шагов оптимизатора), мы не даем весам модели вырасти до огромных значений и подстроиться под шум обучающей выборки. Это математически эквивалентно L2-регуляризации: меньшее число шагов градиентного спуска соответствует большему коэффициенту регуляризации $\lambda$.

### 9. Gradient Clipping (Отсечение градиентов)

Представьте горную дорогу. Adam — умный стабилизатор подвески, адаптирующийся к кочкам. Но если вы несётесь к обрыву, никакой стабилизатор не поможет — нужен **аварийный тормоз**. Gradient Clipping и есть этот тормоз.

Проблема «взрыва градиентов» особенно остра в RNN и Seq2Seq: произведение якобианов за много шагов по времени может превратиться в огромное число, и один шаг оптимизатора выбросит веса далеко в никуда.

**Norm Clipping — предпочтительный метод:**
Если евклидова норма суммарного вектора всех градиентов превышает порог, вектор пропорционально масштабируется обратно на поверхность сферы:

$$\Large \mathbf{g} \leftarrow \mathbf{g} \cdot \frac{\text{max\_norm}}{\|\mathbf{g}\|_2}$$

где:
- $\mathbf{g}$ — вектор всех градиентов сети (сплющенные и конкатенированные параметры)
- $\|\mathbf{g}\|_2$ — евклидова (L2) норма этого вектора
- $\text{max\_norm}$ — порог; при $\|\mathbf{g}\|_2 \leq \text{max\_norm}$ градиент не трогается

*Ключевое преимущество перед Value Clipping:* **направление** вектора обновления сохраняется — меняется только масштаб. Value Clipping (зажим каждого градиента в $[-c, c]$ независимо) искажает направление градиента, меняя относительные пропорции параметров.

**Рекомендуемые пороги `max_norm`:**
- **RNN/LSTM:** `5.0` — эмпирически проверено в оригинальных seq2seq-статьях.
- **Трансформеры:** `1.0` — стандарт оригинальной статьи *Attention is All You Need* (Vaswani et al., 2017).
- **CNN:** `10–100` — свёрточные сети значительно устойчивее к взрыву градиентов.

На практике мониторят `grad_norm` в логах: если постоянно «клипится» → порог занижен; если никогда не клипится → порог можно убрать.

```python
import torch

# ВАЖНО: вызывать ПОСЛЕ .backward(), но ДО optimizer.step()
optimizer.zero_grad()
loss.backward()

# Отсечение нормы по всем параметрам сети
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)

optimizer.step()
```

### 10. Mixed Precision Training (AMP)

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

$$\Large m_t = \beta_1m_{t-1} + (1-\beta_1)g_t$$

$$\Large v_t = \beta_2v_{t-1} + (1-\beta_2)g_t^2$$

$$\Large \theta_{t+1} = \theta_t - \eta_t\frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

где:
- $g_t$ — текущий градиент
- $\hat{m}_t$ — скорректированное скользящее среднее градиента (первый момент)
- $\hat{v}_t$ — скорректированное скользящее среднее квадрата градиента (второй момент)
- $\epsilon$ — малая константа для стабильности ($\sim 10^{-8}$)

**Adam + L2-регуляризация:**

$$\Large g_t^{L2} = \nabla_{\theta}L_t(\theta_t) + \lambda\theta_t$$

Здесь $\lambda\theta_t$ попадает внутрь Adam-моментов и адаптивного деления на $\sqrt{\hat{v}_t}$.

**AdamW / decoupled weight decay:**

$$\Large u_t = \frac{\hat{m}_t}{\sqrt{\hat{v}_t}+\epsilon}$$

$$\Large \theta_{t+1} = (1-\eta_t\lambda)\theta_t - \eta_t u_t$$

Здесь weight decay применяется напрямую к весам и не смешивается с gradient history.

**LAMB trust ratio для блока $\mathcal{G}_b$:**

$$\Large \rho_{t,\mathcal{G}_b} =
\frac{\phi(\|\theta_{t,\mathcal{G}_b}\|_2)}
{\left\|\frac{\hat{m}_{t,\mathcal{G}_b}}{\sqrt{\hat{v}_{t,\mathcal{G}_b}}+\epsilon} + \lambda\theta_{t,\mathcal{G}_b}\right\|_2}$$

$$\Large \theta_{t+1,\mathcal{G}_b}
=
\theta_{t,\mathcal{G}_b}
-
\eta_t\rho_{t,\mathcal{G}_b}
\left(
\frac{\hat{m}_{t,\mathcal{G}_b}}{\sqrt{\hat{v}_{t,\mathcal{G}_b}}+\epsilon}
+
\lambda\theta_{t,\mathcal{G}_b}
\right)$$

где $\rho_{t,\mathcal{G}_b}$ масштабирует шаг слоя относительно нормы его весов.

**Gradient Clipping (Norm Clipping):**
$$\Large \mathbf{g} \leftarrow \mathbf{g} \cdot \frac{\text{max\_norm}}{\|\mathbf{g}\|_2}, \quad \text{если } \|\mathbf{g}\|_2 > \text{max\_norm}$$

где:
- $\mathbf{g}$ — вектор всех градиентов сети
- $\|\mathbf{g}\|_2$ — евклидова норма
- $\text{max\_norm}$ — порог (1.0 для Transformer, 5.0 для RNN)

## Короткий пример

### AdamW для Transformer/BERT fine-tuning

```python
import torch

decay_params = []
no_decay_params = []

for name, param in model.named_parameters():
    if not param.requires_grad:
        continue
    if name.endswith("bias") or "norm" in name.lower():
        no_decay_params.append(param)
    else:
        decay_params.append(param)

optimizer = torch.optim.AdamW(
    [
        {"params": decay_params, "weight_decay": 0.01},
        {"params": no_decay_params, "weight_decay": 0.0},
    ],
    lr=2e-5,
    betas=(0.9, 0.999),
)

for epoch in range(100):
    for batch in dataloader:
        optimizer.zero_grad()
        loss = compute_loss(model, batch)
        loss.backward()
        optimizer.step()
```

### AdamW + scheduler

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=3e-4, weight_decay=0.1)
scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(
    optimizer,
    T_max=num_training_steps,
)

for batch in dataloader:
    optimizer.zero_grad()
    loss = compute_loss(model, batch)
    loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
    optimizer.step()
    scheduler.step()  # Если scheduler рассчитан на шаги, а не эпохи
```

> [!warning] Scheduler зависит от того, как он задан
> `StepLR` часто вызывают раз в эпоху, а `CosineAnnealingLR`/linear warmup в Transformer-пайплайнах часто вызывают раз в optimizer step. Ошибка не в самом месте вызова, а в несоответствии между scheduler и ожидаемой единицей времени.

### LAMB для large-batch pretraining

```python
# В torch.optim обычно нет LAMB из коробки.
# Пример с внешней библиотекой pytorch-optimizer:

from pytorch_optimizer import Lamb

optimizer = Lamb(
    model.parameters(),
    lr=1e-3,
    betas=(0.9, 0.999),
    weight_decay=0.01,
)
```

LAMB имеет смысл проверять, когда effective batch size уже очень большой и AdamW начинает терять качество или нестабильно масштабируется.

## Типичные ошибки

- **Вызывать `scheduler.step()` не в той единице времени:** `StepLR` часто ожидает вызов раз в эпоху, а warmup/linear/cosine scheduler для Transformer часто ожидает вызов раз в optimizer step. Нужно смотреть, на что рассчитан `T_max`, `num_training_steps` или `step_size`.
- **Забыть про VRAM для Adam:** Adam хранит два момента для каждого параметра → потребление памяти в **3 раза** больше, чем у SGD.
- **Не применять Gradient Clipping в RNN:** В рекуррентных сетях взрыв градиентов — системная проблема, а не редкость. `clip_grad_norm_` — обязательная строка для любого LSTM/GRU-обучения.
- **Вызывать `clip_grad_norm_()` до `.backward()`:** Градиенты ещё не вычислены — вызов бессмысленен. Правильный порядок: `loss.backward()` → `clip_grad_norm_()` → `optimizer.step()`.
- **Считать `Adam(weight_decay=...)` и AdamW одним и тем же:** в Adam L2-штраф проходит через адаптивное масштабирование, в AdamW decay отделён от gradient update.
- **Decay-ить bias и normalization weights:** часто ухудшает качество, особенно в Transformer. Для `bias`, `LayerNorm`, `BatchNorm`, `RMSNorm` обычно ставят `weight_decay=0`.
- **Ждать магии от LAMB на маленьком fine-tuning:** LAMB создан прежде всего для large-batch distributed pretraining; на обычном fine-tuning он часто не даёт выигрыша.
- **Называть LANS `Lookahead + Noisy Student`:** это неверная связка. LANS — large-batch optimizer из семейства LAMB; Noisy Student — semi-supervised training pipeline.

## Каверзные вопросы

> [!question] Модель на SGD застряла на loss $0.5$. Поможет ли замена на Adam?
> Не факт. Adam быстрее на старте, но часто сходится к менее обобщающим «острым» оптимумам. Коллеге сначала стоит применить LR Scheduler для SGD.

## Проверка себя

- За счёт чего Momentum «выскакивает» из локальных ям?
- Какую проблему решает деление на $\sqrt{\hat{v}_t}$ в Adam?
- Чем AdamW отличается от Adam с L2-регуляризацией?
- Почему bias и LayerNorm weights часто исключают из weight decay?
- Что такое trust ratio в LAMB и зачем он нужен при large batch training?
- Почему LANS не стоит объяснять как Lookahead + Noisy Student?

## Предпосылки

- [[Градиентный спуск]]
- [[Метод обратного распространения ошибки (Backpropagation)]]

## Связано

- [[Регуляризация в DL — Dropout]]
- [[Batch Normalization]]

## Источники

- Интенсив Основы глубокого обучения 4 Радослав Нейчев MADE
- [Decoupled Weight Decay Regularization — Loshchilov, Hutter](https://arxiv.org/abs/1711.05101)
- [Large Batch Optimization for Deep Learning: Training BERT in 76 minutes — You et al.](https://arxiv.org/abs/1904.00962)
- [Accelerated Large Batch Optimization of BERT Pretraining in 54 minutes — Zheng et al.](https://arxiv.org/abs/2006.13484)
- [Lookahead Optimizer: k steps forward, 1 step back — Zhang et al.](https://arxiv.org/abs/1907.08610)
- [Self-training with Noisy Student improves ImageNet classification — Xie et al.](https://arxiv.org/abs/1911.04252)
- [PyTorch AdamW documentation](https://docs.pytorch.org/docs/stable/generated/torch.optim.adamw.AdamW_class.html)
- Семинар. PyTorch. Оптимизаторы
- Алгоритмы градиентного спуска

---
[[🗺️ Индекс|Назад к разделу]]
