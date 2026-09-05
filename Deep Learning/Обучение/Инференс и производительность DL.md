---
tags: [deep-learning, inference, production, gpu, cuda, latency, throughput, quantization, distributed-training]
тип: теория
уровень: senior
сложность: высокая
статус: готово
готовность: 90
создано: 2026-05-26
источники:
  - "Практика production inference"
  - "Семинар. PyTorch. Оптимизаторы"
предпосылки:
  - "Полносвязные нейронные сети (MLP)"
  - "Вычислительный граф и Autograd"
связано:
  - "Алгоритмы оптимизации (SGD, Momentum, RMSProp, Adam)"
  - "Сжатие языковых моделей (Distillation, Quantization, Pruning, LoRA)"
  - "Быстрая генерация LLM — batching, prefill, KV-cache и speculative decoding"
  - "Форматы хранения и экспорта моделей — ONNX, safetensors, TorchScript и GGUF"
сравнить-с: []
cssclasses: [wide-page]
---

# Инференс и производительность DL

> [!abstract] Суть
> Production-инференс — это не просто `model(x)`. Нужно отключить обучающее поведение, убрать autograd, выбрать формат модели, держать GPU загруженным, контролировать latency/throughput, батчинг, память, версионирование и мониторинг качества.

## Ответ

### 1. Training vs inference

Во время training модель делает forward, считает loss, хранит активации для backward, считает градиенты и обновляет веса. Во время inference нужен только forward pass.

На inference обычно:
- вызывают `model.eval()`, чтобы отключить Dropout и перевести BatchNorm на running statistics;
- используют `torch.no_grad()` или `torch.inference_mode()`, чтобы не строить граф autograd;
- часто применяют mixed precision, TorchScript/ONNX/TensorRT, quantization или pruning;
- фиксируют версию модели и препроцессинга.

Подробнее про выбор формата модели: [[Deep Learning/Практика и инструменты/Форматы хранения и экспорта моделей — ONNX, safetensors, TorchScript и GGUF|Форматы хранения и экспорта моделей]].

Inference быстрее, потому что нет backward pass, градиентов, состояний оптимизатора и хранения промежуточных активаций для обучения.

### 2. Latency и throughput

**Latency** — время ответа на один запрос. Обычно смотрят p50/p95/p99, а не только среднее.

**Throughput** — сколько запросов система обрабатывает в секунду.

Batch inference увеличивает throughput, потому что матричные операции на GPU лучше загружаются большим батчем. Но батчинг может увеличить latency отдельного запроса из-за ожидания формирования батча. В real-time сервисах используют dynamic batching: короткое окно ожидания, например 2-10 мс.

### 3. Как уменьшить latency модели

Основные рычаги:
- выбрать более лёгкую архитектуру: MobileNet/EfficientNet-lite вместо тяжёлой ResNet;
- уменьшить входное разрешение или длину последовательности;
- включить mixed precision (`fp16`/`bf16`);
- применить quantization: `fp32 -> int8`;
- применить pruning или distillation;
- сделать operator/kernel fusion;
- использовать TensorRT/ONNX Runtime/Torch Compile;
- вынести препроцессинг на GPU или распараллелить CPU pipeline;
- кэшировать повторяющиеся эмбеддинги/признаки;
- батчить запросы, если SLA позволяет.

### 4. Quantization и pruning

**Quantization** хранит веса и/или активации в меньшей точности: например, INT8 вместо FP32. Это снижает память, уменьшает bandwidth pressure и может ускорить inference на железе, где есть быстрые INT8/FP8 kernels. Но ускорение не гарантировано: нужна поддержка runtime, kernels и подходящие размеры матриц.

Главная идея: вещественное значение $x$ заменяется целым числом $q$, а при вычислениях приближённо восстанавливается через scale и zero-point.

$$\Large
q = \operatorname{round}\left(\frac{x}{s}\right) + z
$$

$$\Large
\hat{x} = s \cdot (q - z)
$$

где:

- $x$ — исходное вещественное значение веса или активации;
- $q$ — квантованное целое значение, например INT8;
- $\hat{x}$ — восстановленное приближение исходного значения;
- $s$ — scale, шаг квантования;
- $z$ — zero-point, целое значение, соответствующее нулю;
- $\operatorname{round}$ — округление до ближайшего целого.

Чем меньше битность, тем сильнее сжатие, но тем выше quantization error:

| Формат | Размер | Где часто применяют | Комментарий |
|---|---:|---|---|
| FP32 | 32 bit | обучение, baseline inference | максимум точности, дорого |
| FP16/BF16 | 16 bit | GPU inference/training | mixed precision, обычно почти без потери качества |
| FP8 | 8 bit float | современные GPU, большие модели | быстрее и компактнее, но нужна аппаратная поддержка |
| INT8 | 8 bit integer | CPU/GPU inference, CV/NLP | стандартная production-квантизация |
| INT4/NF4 | 4 bit | LLM weight-only, QLoRA | сильное сжатие, чаще для весов, не всегда ускоряет |

#### 4.1 Symmetric vs asymmetric quantization

**Symmetric quantization** использует нулевой zero-point:

$$\Large
q = \operatorname{round}\left(\frac{x}{s}\right)
$$

где:

- $q$ — целое квантованное значение;
- $x$ — исходное значение;
- $s$ — scale.

Плюсы: проще и быстрее для матричных умножений. Часто используется для весов, потому что распределение весов обычно примерно центрировано вокруг нуля.

**Asymmetric quantization** использует zero-point $z$ и лучше покрывает несимметричные распределения:

$$\Large
q = \operatorname{round}\left(\frac{x}{s}\right) + z
$$

Плюсы: лучше для активаций после ReLU, где значения неотрицательные и распределение сдвинуто. Минус: вычисления сложнее из-за поправки на zero-point.

#### 4.2 Per-tensor vs per-channel quantization

**Per-tensor quantization** задаёт один scale для всего тензора.

**Per-channel quantization** задаёт отдельный scale для каждого выходного канала или строки матрицы весов.

Пример: если в convolution layer один канал имеет маленькие веса, а другой большие, общий scale будет грубым для маленького канала. Per-channel scale уменьшает ошибку.

Практическое правило:
- веса лучше квантовать per-channel;
- активации часто квантуют per-tensor, потому что per-channel activation quantization дороже;
- для LLM часто используют group-wise quantization: отдельный scale на группу весов, например 32/64/128 элементов.

#### 4.3 Post-Training Quantization (PTQ)

**PTQ** применяется после обучения: берём готовую FP32/FP16 модель и квантуем без дообучения или с минимальной calibration-процедурой.

Варианты PTQ:

1. **Dynamic quantization**
   - веса квантуются заранее;
   - активации квантуются на лету во время inference;
   - хорошо подходит для LSTM/Transformer linear layers на CPU;
   - почти не требует calibration dataset;
   - ускорение зависит от backend.

2. **Static quantization**
   - заранее квантуются и веса, и активации;
   - нужен calibration dataset, чтобы подобрать диапазоны активаций;
   - часто быстрее dynamic quantization;
   - чувствительна к representativeness calibration data.

3. **Weight-only quantization**
   - квантуются только веса, активации остаются FP16/BF16/FP32;
   - популярна для LLM, где весовые матрицы огромные и bottleneck часто в памяти;
   - даёт сильное сжатие VRAM;
   - не всегда ускоряет inference, если dequantization становится bottleneck.

PTQ хороша, когда нужно быстро попробовать сжатие без переобучения. Если качество просело слишком сильно, переходят к QAT.

#### 4.4 Calibration

Calibration — это прогон небольшого набора representative data через модель, чтобы подобрать диапазоны активаций.

Самый простой scale через min/max:

$$\Large
s = \frac{x_{\max} - x_{\min}}{q_{\max} - q_{\min}}
$$

$$\Large
z = q_{\min} - \operatorname{round}\left(\frac{x_{\min}}{s}\right)
$$

где:

- $x_{\min}$ и $x_{\max}$ — минимальное и максимальное значения исходного тензора или активации;
- $q_{\min}$ и $q_{\max}$ — минимальное и максимальное значения целевого integer-формата, например `0..255` для UINT8 или `-128..127` для INT8;
- $s$ — scale;
- $z$ — zero-point.

Проблема min/max: один выброс может растянуть диапазон и сделать квантование грубым для большинства значений. Поэтому используют percentile clipping, moving average ranges, KL-divergence calibration или MSE-based calibration.

#### 4.5 Quantization-Aware Training (QAT)

**QAT** имитирует квантизацию во время обучения или fine-tuning. Модель заранее привыкает к quantization noise, поэтому качество после INT8 inference обычно лучше, чем у PTQ.

Внутри QAT часто используют fake quantization:

```text
FP32 weights/activations
  -> fake quantize to INT8 grid
  -> dequantize back to FP32
  -> forward/backward как обычно
```

Градиент через округление обычно проводят через straight-through estimator:

$$\Large
\frac{\partial \operatorname{round}(x)}{\partial x} \approx 1
$$

где:

- $\operatorname{round}(x)$ — недифференцируемое округление;
- приближение $\approx 1$ позволяет backpropagation проходить через fake quantization.

QAT стоит использовать, если:
- INT8 PTQ даёт заметную просадку качества;
- модель маленькая и чувствительная к ошибкам квантования;
- есть доступ к train/fine-tuning данным;
- production SLA требует именно INT8.

Минусы QAT: сложнее pipeline, дольше обучение, нужно следить, чтобы train-time fake quantization совпадала с production runtime.

#### 4.6 Weight-only quantization для LLM

Для больших языковых моделей часто квантуют веса до INT8/INT4/NF4, а активации оставляют FP16/BF16. Причина: LLM inference часто memory-bound, особенно при загрузке огромных матриц весов.

Типичные варианты:

- **INT8 weight-only:** хороший компромисс, часто малая потеря качества.
- **INT4:** сильное сжатие, но выше риск деградации.
- **NF4:** 4-битный формат, подобранный под примерно нормальное распределение весов; часто используется в QLoRA/fine-tuning.
- **GPTQ/AWQ-подобные подходы:** стараются квантизовать веса с учётом чувствительности слоёв и активаций.

Практическая интерпретация:
- если цель — влезть в VRAM, weight-only INT4 может быть главным рычагом;
- если цель — минимальная latency, нужно проверять реальные kernels: иногда INT4 экономит память, но не ускоряет;
- attention KV-cache тоже может стать bottleneck, и квантизация весов сама по себе не решит всю проблему.

#### 4.7 Что выбирать на практике

| Ситуация | Хороший первый выбор | Почему |
|---|---|---|
| CNN/ResNet на edge/CPU | static INT8 PTQ | быстрый inference, зрелые kernels |
| Модель на CPU с Linear/LSTM | dynamic INT8 | простой старт без calibration |
| Transformer/LLM inference на GPU | FP16/BF16, затем weight-only INT8/INT4 | сначала безопасная mixed precision, потом сжатие весов |
| Жёсткий SLA и INT8 просаживает качество | QAT | модель адаптируется к quantization noise |
| Нужно просто уменьшить VRAM | weight-only quantization | сжимает самый большой компонент — веса |
| Маленький calibration dataset | dynamic или осторожная PTQ | static PTQ может плохо подобрать диапазоны |

#### 4.8 Когда квантизация не помогает

Квантизация может не дать ускорения, если:
- runtime не использует INT8/FP8 kernels;
- модель bottleneck не в матричных умножениях, а в preprocessing/data transfer;
- batch size слишком маленький и overhead dequantization съедает выигрыш;
- операция memory-bound, но квантизуется не тот тензор;
- есть много unsupported ops, из-за которых граф прыгает между INT8 и FP32;
- accuracy падает и приходится добавлять дорогие fallback-ветки.

**Pruning** удаляет веса, каналы, головы attention, нейроны или целые блоки, которые мало влияют на качество. Идея похожа на "обрезку лишнего": сделать модель меньше, дешевле и иногда быстрее.

Важно: pruning почти всегда нужно оценивать не по числу занулённых весов, а по реальному latency/throughput на целевом железе. Модель может стать на 80% sparse, но не ускориться, если runtime не умеет эффективно считать sparse-операции.

#### 4.9 Зачем нужен pruning

Pruning используют, чтобы:

- уменьшить размер модели на диске;
- снизить VRAM/RAM;
- ускорить inference;
- уменьшить энергопотребление;
- убрать избыточность после overparameterized training;
- подготовить модель к deployment на edge/mobile;
- иногда улучшить generalization за счёт удаления шумовых параметров.

Но pruning не бесплатен: если удалить слишком много, качество падает, а восстановление через fine-tuning может быть дорогим.

#### 4.10 Unstructured pruning

**Unstructured pruning** удаляет отдельные веса внутри матриц. Например, зануляем все веса, модуль которых меньше порога.

$$\Large
W'_{ij} =
\begin{cases}
0, & |W_{ij}| < \tau \\
W_{ij}, & |W_{ij}| \ge \tau
\end{cases}
$$

где:

- $W_{ij}$ — исходный вес в строке $i$, столбце $j$;
- $W'_{ij}$ — вес после pruning;
- $|W_{ij}|$ — модуль веса;
- $\tau$ — порог pruning: веса меньше порога зануляются.

Плюсы:

- просто реализовать;
- можно получить очень высокую sparsity;
- часто хорошо сохраняет качество при умеренной обрезке.

Минусы:

- обычные dense kernels не ускоряются от нулей;
- нужна sparse inference engine или аппаратная поддержка;
- нерегулярная разреженность плохо ложится на GPU;
- размер модели уменьшается только при специальном sparse storage.

Практическая интерпретация: unstructured pruning полезен для compression, но не гарантирует speedup.

#### 4.11 Structured pruning

**Structured pruning** удаляет целые структурные элементы:

- каналы в CNN;
- convolution filters;
- нейроны MLP;
- attention heads;
- FFN intermediate dimensions;
- residual blocks;
- encoder/decoder layers.

Пример channel pruning: если удаляем выходной канал convolution layer, следующий слой тоже нужно перестроить, потому что у него уменьшается число входных каналов.

Плюсы:

- реально уменьшает формы тензоров;
- работает с обычными dense kernels;
- чаще даёт настоящий speedup;
- проще деплоить в ONNX/TensorRT/мобильные runtime.

Минусы:

- сложнее реализовать корректно;
- сильнее риск просадки качества;
- нужно аккуратно чинить зависимости между слоями;
- residual/skip connections накладывают ограничения на совместимость размерностей.

Практическое правило: если цель — **ускорить inference**, сначала смотреть structured pruning. Если цель — **сжать веса**, можно рассматривать unstructured pruning.

#### 4.12 Magnitude pruning

Самый простой критерий: маленькие по модулю веса считаются менее важными.

Глобальный вариант:

$$\Large
\mathcal{M}_{ij} =
\mathbf{1}\left(|W_{ij}| \ge \tau\right)
$$

$$\Large
W' = W \odot \mathcal{M}
$$

где:

- $\mathcal{M}_{ij}$ — бинарная маска для веса $W_{ij}$;
- $\mathbf{1}(\cdot)$ — индикатор: 1, если условие истинно, иначе 0;
- $W$ — исходная матрица весов;
- $W'$ — матрица после pruning;
- $\odot$ — поэлементное умножение;
- $\tau$ — порог по модулю веса.

Варианты:

- **global magnitude pruning:** один общий порог на всю модель;
- **layer-wise magnitude pruning:** отдельный порог для каждого слоя;
- **block-wise pruning:** удаляются блоки весов, например 4x1 или 4x4.

Global pruning чаще эффективнее, потому что сам распределяет sparsity между слоями. Но он может слишком сильно обрезать чувствительные слои, поэтому иногда вводят ограничения по минимальной плотности каждого слоя.

#### 4.13 Pruning по нормам каналов/фильтров

Для structured pruning часто оценивают важность канала через норму его весов.

$$\Large
Score(c) = \left\| W_c \right\|_p
$$

где:

- $c$ — канал или фильтр;
- $W_c$ — все веса, относящиеся к каналу $c$;
- $\|\cdot\|_p$ — $L_p$-норма, часто $L_1$ или $L_2$;
- $Score(c)$ — важность канала.

Если $Score(c)$ маленький, канал кандидат на удаление.

Для BatchNorm есть популярная эвристика: смотреть на параметр масштаба $\gamma$.

$$\Large
y = \gamma \hat{x} + \beta
$$

где:

- $y$ — выход BatchNorm;
- $\hat{x}$ — нормализованная активация;
- $\gamma$ — обучаемый scale канала;
- $\beta$ — обучаемый bias/shift.

Если $|\gamma|$ близок к нулю, канал почти подавлен BatchNorm и может быть кандидатом на pruning. Но это эвристика: перед удалением нужно проверять качество.

#### 4.14 Activation-based pruning

Можно смотреть не только на веса, но и на реальные активации на calibration/validation data.

$$\Large
Score(c) =
\frac{1}{N}
\sum_{n=1}^{N}
\left|
a_{n,c}
\right|
$$

где:

- $Score(c)$ — средняя активность канала $c$;
- $N$ — число объектов calibration/validation набора;
- $a_{n,c}$ — активация канала $c$ на объекте $n$.

Если канал почти всегда даёт нулевые или очень малые активации, его можно рассмотреть для удаления.

Плюс: критерий учитывает данные. Минус: нужен representative dataset, иначе можно удалить канал, который важен для редкого, но бизнес-критичного сегмента.

#### 4.15 Gradient/Taylor pruning

Более продвинутый подход: оценивать, как сильно изменится loss при удалении веса/канала.

Первый порядок Taylor approximation:

$$\Large
\Delta L_j \approx
\left|
\frac{\partial L}{\partial \theta_j}
\cdot
\theta_j
\right|
$$

где:

- $\Delta L_j$ — приблизительное изменение loss при удалении параметра $\theta_j$;
- $L$ — функция потерь;
- $\theta_j$ — параметр, канал или группа параметров;
- $\frac{\partial L}{\partial \theta_j}$ — градиент loss по этому параметру.

Если $\Delta L_j$ маленькое, удаление элемента должно мало повлиять на loss.

Плюсы:

- учитывает чувствительность loss;
- может быть точнее magnitude pruning.

Минусы:

- требует градиентов;
- дороже;
- оценка локальная и шумная;
- нужно аккуратно усреднять по batch/fold.

#### 4.16 Iterative pruning и fine-tuning

Обычно pruning делают не одним грубым шагом, а итеративно:

```text
1. Обучить dense model
2. Удалить 10-20% наименее важных весов/каналов
3. Fine-tune несколько эпох
4. Снова удалить часть весов/каналов
5. Fine-tune
6. Остановиться, когда качество/latency достигли нужного компромисса
```

Почему не удалять сразу 80%:

- модель получает резкий distribution shift внутри себя;
- оставшиеся веса не успевают адаптироваться;
- качество может упасть необратимо;
- сложнее понять, какой уровень sparsity ещё безопасен.

Fine-tuning после pruning почти всегда нужен. Иногда достаточно нескольких эпох с маленьким learning rate.

#### 4.17 One-shot pruning

**One-shot pruning** удаляет большую долю параметров один раз, а затем fine-tune. Это быстрее, но рискованнее.

Когда подходит:

- модель очень избыточная;
- pruning ratio небольшой;
- есть сильный baseline и много данных для fine-tuning;
- нужно быстро получить compression baseline.

Когда лучше iterative:

- высокий pruning ratio;
- маленькая модель;
- чувствительная метрика;
- production SLA строгий.

#### 4.18 Lottery Ticket Hypothesis

Lottery Ticket Hypothesis говорит, что внутри большой случайно инициализированной сети могут существовать маленькие подсети, которые при удачной исходной инициализации обучаются почти так же хорошо, как полная сеть.

Практический смысл для pruning:

- большие модели часто содержат избыточные параметры;
- pruning может найти компактную подсеть;
- но найти её заранее сложно;
- простое удаление после обучения не всегда равно нахождению "выигрышного билета".

Для production это скорее интуиция, чем готовый алгоритм: pruning всё равно проверяется экспериментами.

#### 4.19 Pruning attention heads и Transformer-блоков

В Transformer можно удалять:

- отдельные attention heads;
- FFN hidden dimensions;
- целые слои;
- embedding dimensions;
- блоки MLP/attention.

Нюансы:

- не все attention heads одинаково важны;
- pruning heads может почти не менять размер FFN, а FFN часто занимает большую часть параметров;
- удаление слоёв даёт понятный speedup, но сильнее бьёт по качеству;
- residual stream и LayerNorm требуют аккуратного сохранения размерностей;
- для autoregressive LLM нужно учитывать KV-cache: pruning весов не всегда уменьшает cache memory.

Практическое правило: для Transformer speedup часто дают layer pruning, FFN dimension pruning и optimized kernels, а не только head pruning.

#### 4.20 Sparse pruning и hardware

Нерегулярная sparse matrix multiplication на GPU часто не быстрее dense multiplication, потому что:

- индексы sparse-структуры тоже нужно читать из памяти;
- доступ к памяти становится нерегулярным;
- GPU хуже заполняет SIMD/SIMT lanes;
- overhead может превышать экономию FLOPS.

Более hardware-friendly варианты:

- **structured pruning:** удалять каналы/блоки;
- **block sparsity:** удалять блоки фиксированного размера;
- **N:M sparsity:** например, 2:4 sparsity, где в каждой группе из 4 весов ровно 2 ненулевых.

N:M sparsity:

```text
[w1, w2, w3, w4] -> оставить только 2 значения из 4
```

Она более регулярна, поэтому лучше поддерживается специализированным железом и kernels.

#### 4.21 Pruning vs quantization vs distillation

| Метод | Что уменьшает | Когда хорош | Главный риск |
|---|---|---|---|
| Pruning | число весов/каналов/блоков | модель избыточна, нужен speedup/сжатие | sparse не ускорится без поддержки |
| Quantization | битность весов/активаций | нужно уменьшить память и bandwidth | просадка качества, unsupported ops |
| Distillation | архитектуру целиком | можно обучить student model | нужен teacher/student training pipeline |

Часто их комбинируют:

```text
Teacher model
  -> distillation в меньшую архитектуру
  -> structured pruning
  -> quantization
  -> benchmark на production runtime
```

Но порядок важен: если сначала слишком агрессивно квантизовать, а потом pruning, качество может деградировать сильнее. Частый production-подход: сначала distillation/pruning, затем quantization-aware или post-training quantization.

#### 4.22 Практический пример pruning-пайплайна

```python
import torch
import torch.nn.utils.prune as prune
import torch.nn as nn

model = MyModel()

# 1. Unstructured magnitude pruning для одного Linear слоя
prune.l1_unstructured(
    module=model.classifier,
    name="weight",
    amount=0.3,  # занулить 30% весов с наименьшим |w|
)

# 2. Проверяем качество на validation
val_score = evaluate(model, val_loader)

# 3. Fine-tune модель с маской pruning
fine_tune(model, train_loader, lr=1e-5)

# 4. Если хотим физически убрать reparameterization pruning из PyTorch
prune.remove(model.classifier, "weight")
```

Важный нюанс PyTorch pruning: стандартный `torch.nn.utils.prune` часто создаёт маску и зануляет веса, но не меняет форму матрицы. Это значит, что latency может не снизиться. Для реального speedup обычно нужно physically shrink architecture: удалить каналы/нейроны и пересобрать слои.

#### 4.23 Как понять, что pruning сработал

Оценивать нужно несколько метрик:

- validation/test quality;
- model size on disk;
- RAM/VRAM usage;
- p50/p95/p99 latency;
- throughput;
- CPU/GPU utilization;
- calibration/robustness;
- качество на редких сегментах.

Если sparsity выросла, но latency не изменилась — pruning сработал как compression, но не как acceleration.

### 5. GPU, CUDA и tensor cores

GPU хорошо подходят для DL, потому что умеют выполнять тысячи одинаковых операций параллельно. CPU силён в ветвлениях и низкой latency одиночных задач, GPU — в массивном параллелизме.

**CUDA** — платформа NVIDIA для запуска вычислительных kernels на GPU. **CUDA kernel** — функция, которая выполняется параллельно на множестве потоков GPU.

**Tensor cores** — специализированные блоки для матричных умножений в FP16/BF16/TF32/INT8. Они дают огромный выигрыш, если размеры матриц и типы данных подходят аппаратным требованиям.

Часто узкое место не FLOPS, а **memory bandwidth**: данные нужно прочитать из GPU-памяти, передать между CPU/GPU и сохранить обратно. Поэтому kernel fusion помогает: несколько операций объединяются в один kernel, уменьшая чтение/запись промежуточных тензоров.

### 6. Почему GPU может простаивать

Типичные причины:
- маленький batch size;
- медленный data loading или препроцессинг на CPU;
- частые CPU↔GPU transfer;
- много маленьких kernels вместо нескольких крупных;
- синхронизации `.item()`, `print(tensor)`, явные `torch.cuda.synchronize()`;
- модель memory-bound, а не compute-bound;
- неэффективные формы тензоров.

CPU↔GPU transfer дорогой, потому что данные пересекают PCIe/NVLink и часто синхронизируют pipeline. Правило: держать данные на GPU как можно дольше и переносить крупными блоками.

### 7. Asynchronous execution

GPU-операции часто запускаются асинхронно: Python отправляет kernel и идёт дальше, пока GPU работает. Из-за этого наивное измерение времени через `time.time()` может быть неверным. Для честного профилирования используют CUDA events или явную синхронизацию вокруг измеряемого участка.

### 8. Что профилировать

В DL pipeline профилируют:
- model forward;
- preprocessing/postprocessing;
- data loading;
- CPU↔GPU transfers;
- загрузку GPU/CPU;
- VRAM;
- p50/p95/p99 latency;
- batch queue waiting time;
- долю времени в kernels.

Инструменты: PyTorch Profiler, TensorBoard trace viewer, Nsight Systems, Nsight Compute, метрики сервинга.

### 9. Distributed training и gradient accumulation

**Distributed training** ускоряет обучение, распределяя данные или модель по нескольким GPU/нодам.

Data parallel training: каждая GPU считает градиенты на своей части батча, затем градиенты синхронизируются через all-reduce. Узкие места: сеть между GPU, дисбаланс батчей, синхронизация и память.

**Gradient accumulation** накапливает градиенты несколько шагов перед `optimizer.step()`. Это увеличивает effective batch size без роста VRAM на активации одного forward, но делает обновления реже и не заменяет полностью настоящий distributed batch.

### 10. Как задеплоить модель для real-time inference

Минимальная production-схема:
1. Зафиксировать артефакты: веса, код модели, препроцессинг, postprocessing, словари/нормализации.
2. Экспортировать или скомпилировать модель: TorchScript/ONNX/TensorRT, если это даёт выигрыш.
3. Поднять stateless inference service с health checks.
4. Добавить dynamic batching и очередь с ограничением по SLA.
5. Настроить autoscaling по RPS, latency, GPU utilization.
6. Логировать входные признаки, предсказания, версию модели и технические метрики.
7. Сделать canary/blue-green deploy для обновления без даунтайма.

Для 10k RPS обычно нужны батчинг, горизонтальное масштабирование, балансировщик, несколько replicas, GPU pooling, circuit breakers и деградационный режим.

### 11. Мониторинг качества и обновление модели

Мониторят:
- data drift и prediction drift;
- бизнес-метрики;
- latency/throughput/error rate;
- долю fallback;
- качество на отложенной разметке;
- расхождение между train и production distribution.

Обновление без даунтайма делают через shadow traffic, canary rollout или blue-green deployment. Новая модель сначала получает малую долю трафика, затем доля растёт при стабильных метриках.

### 12. Как уменьшить стоимость inference

- уменьшить модель через distillation/pruning/quantization;
- оптимизировать batch size под GPU utilization;
- использовать более дешёвое железо под нужный SLA;
- кэшировать частые запросы или признаки;
- разделить fast path и slow path;
- запускать тяжёлую модель только для сложных случаев;
- подобрать autoscaling и scale-to-zero для непиковых нагрузок.

## Короткий пример

```python
import torch

model.eval()

with torch.inference_mode():
    with torch.autocast(device_type="cuda", dtype=torch.float16):
        logits = model(batch.cuda(non_blocking=True))
```

## Типичные ошибки

- Использовать `model.eval()` без `torch.no_grad()` и думать, что граф отключён.
- Мерить latency без CUDA synchronization.
- Ускорять только модель, игнорируя preprocessing и очередь батчинга.
- Делать INT8 quantization без проверки качества на production-like данных.
- Калибровать static quantization на маленьком или нерепрезентативном calibration dataset.
- Ждать ускорения от weight-only INT4 без проверки, есть ли быстрые kernels под конкретный runtime.
- Смешивать quantized и non-quantized ops так, что dequantize/quantize overhead съедает весь выигрыш.
- Гнаться за throughput и случайно нарушить p99 latency.

## Каверзные вопросы

> [!question] Почему batch inference быстрее single inference?
> Потому что GPU эффективнее выполняет крупные матричные операции, чем много маленьких запусков kernels. Но batch inference может ухудшить latency, если запрос слишком долго ждёт формирования батча.

> [!question] Почему memory bandwidth часто важнее FLOPS?
> Если операция читает и пишет много данных, но мало считает, GPU простаивает в ожидании памяти. Ускорение приходит не от большего числа FLOPS, а от fusion, лучшей layout-формы тензоров и уменьшения transfer.

> [!question] Чем PTQ отличается от QAT?
> PTQ квантует уже обученную модель и быстрее внедряется, но может сильнее просадить качество. QAT имитирует квантизацию во время обучения/fine-tuning, поэтому обычно точнее, но дороже по процессу.

> [!question] Почему INT4 LLM может занимать меньше VRAM, но не стать быстрее?
> Потому что веса нужно dequantize перед вычислениями или использовать специальные kernels. Если kernels не оптимизированы, overhead распаковки и dequantization может съесть выигрыш от меньшего объёма памяти.

## Проверка себя

- Чем `model.eval()` отличается от `torch.no_grad()`?
- Почему p99 latency важнее среднего для real-time сервиса?
- Когда quantization даст реальный speedup, а когда только уменьшит размер весов?
- Чем dynamic quantization отличается от static quantization?
- Когда нужен QAT, а когда достаточно PTQ?
- Почему GPU может быть загружен на 20%, хотя модель "тяжёлая"?
- Как бы ты обслужил 10k RPS для CV-модели?

## Связано

- [[Deep Learning/Основы/Вычислительный граф и Autograd]]
- [[Deep Learning/Обучение/Алгоритмы оптимизации (SGD, Momentum, RMSProp, Adam)]]
- [[NLP/LLM и Промпт-инжиниринг/Сжатие языковых моделей (Distillation, Quantization, Pruning, LoRA)]]
- [[NLP/LLM и Промпт-инжиниринг/Быстрая генерация LLM — batching, prefill, KV-cache и speculative decoding]]
- [[Deep Learning/Практика и инструменты/Форматы хранения и экспорта моделей — ONNX, safetensors, TorchScript и GGUF]]

---
[[🗺️ Индекс|Назад к разделу]]
