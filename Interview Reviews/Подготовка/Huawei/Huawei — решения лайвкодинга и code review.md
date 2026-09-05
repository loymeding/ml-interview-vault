---
tags: [interview-prep, huawei, live-coding, code-review, answers, multimodal, edge-ai]
company: Huawei
type: live-coding-solutions
status: active
tasks_total: 10
tasks_solved: 10
cssclasses: [wide-page]
---

# ✅ Huawei — решения лайвкодинга и code review

> [!warning] Спойлеры
> Разборы к [[Interview Reviews/Подготовка/Huawei/Лайвкодинг и code review Huawei|карточке задач]]. Задачи близки к роли по Document AI, speech/video, VLM и mobile inference.

## 👁️🎙️ Live coding

### HW-LC1. CER и WER

CER считает edit distance на символах, WER - на словах:

$$
\Large \operatorname{WER} = \frac{S + D + I}{N},
$$

где `S`, `D`, `I` - число замен, удалений и вставок, а `N` - число слов в reference. При пустом reference WER обычно не определяют либо возвращают `0`, если hypothesis тоже пуста, и `inf` иначе.

```python
def edit_distance(reference, hypothesis):
    previous = list(range(len(hypothesis) + 1))
    for i, ref_item in enumerate(reference, start=1):
        current = [i]
        for j, hyp_item in enumerate(hypothesis, start=1):
            current.append(min(
                previous[j] + 1,
                current[j - 1] + 1,
                previous[j - 1] + (ref_item != hyp_item),
            ))
        previous = current
    return previous[-1]


def cer(reference, hypothesis):
    if not reference:
        return 0.0 if not hypothesis else float("inf")
    return edit_distance(list(reference), list(hypothesis)) / len(reference)


def wer(reference, hypothesis):
    ref_words, hyp_words = reference.split(), hypothesis.split()
    if not ref_words:
        return 0.0 if not hyp_words else float("inf")
    return edit_distance(ref_words, hyp_words) / len(ref_words)
```

До метрики обычно нормализуют Unicode, регистр, пунктуацию и числа - но это правило должно быть одинаковым для reference и prediction и зафиксировано в benchmark.

### HW-LC2. IoU и NMS

```python
def iou(a, b):
    ax1, ay1, ax2, ay2 = a
    bx1, by1, bx2, by2 = b
    if ax2 <= ax1 or ay2 <= ay1 or bx2 <= bx1 or by2 <= by1:
        raise ValueError("Invalid box")
    inter_w = max(0, min(ax2, bx2) - max(ax1, bx1))
    inter_h = max(0, min(ay2, by2) - max(ay1, by1))
    intersection = inter_w * inter_h
    union = (ax2 - ax1) * (ay2 - ay1) + (bx2 - bx1) * (by2 - by1) - intersection
    return intersection / union


def nms(boxes, scores, threshold):
    order = sorted(range(len(boxes)), key=lambda i: (-scores[i], i))
    kept = []
    while order:
        current = order.pop(0)
        kept.append(current)
        order = [idx for idx in order if iou(boxes[current], boxes[idx]) <= threshold]
    return kept
```

Class-wise NMS подавляет боксы только внутри одного класса. Class-agnostic вариант может случайно удалить, например, «таблицу» из-за пересечения с «подписью» - в Document AI это часто неверно.

### HW-LC3. Порядок чтения документа

Для простой двухколоночной страницы можно кластеризовать блоки по `x`-центрам и сортировать внутри колонки по `y`. Это эвристика, а не универсальное решение.

```python
from collections import defaultdict


def reading_order(blocks, two_columns=False):
    by_page = defaultdict(list)
    for block in blocks:
        by_page[block["page"]].append(block)
    result = []
    for page in sorted(by_page):
        page_blocks = by_page[page]
        if not two_columns:
            result.extend(sorted(page_blocks, key=lambda b: (b["bbox"][1], b["bbox"][0])))
            continue
        centers = [((b["bbox"][0] + b["bbox"][2]) / 2) for b in page_blocks]
        split_x = (min(centers) + max(centers)) / 2
        left = [b for b in page_blocks if (b["bbox"][0] + b["bbox"][2]) / 2 < split_x]
        right = [b for b in page_blocks if b not in left]
        result.extend(sorted(left, key=lambda b: (b["bbox"][1], b["bbox"][0])))
        result.extend(sorted(right, key=lambda b: (b["bbox"][1], b["bbox"][0])))
    return result
```

Таблицы, боковые подписи, full-width заголовки, mixed layout и rotated text требуют layout model, типов блоков и иногда графа reading order.

### HW-LC4. Временное выравнивание

При отсортированных кадрах используем два указателя. Каждый кадр рассматривается ограниченное число раз, поэтому время `O(S + F + output)`.

```python
def align_segments_to_frames(segments, frames, context_seconds=0.0):
    frames = sorted(frames, key=lambda x: x["timestamp"])
    result, left = [], 0
    for segment in sorted(segments, key=lambda x: x["start"]):
        start = segment["start"] - context_seconds
        end = segment["end"] + context_seconds
        while left < len(frames) and frames[left]["timestamp"] < start:
            left += 1
        right = left
        selected = []
        while right < len(frames) and frames[right]["timestamp"] <= end:
            selected.append(frames[right])
            right += 1
        result.append({**segment, "frames": selected})
    return result
```

Для сильно перекрывающихся сегментов лучше хранить активное окно или использовать interval index, иначе одни и те же кадры будут повторно сканироваться.

### HW-LC5. Multimodal collate

Mask должен содержать `1` на реальных токенах и `0` на padding для большинства Hugging Face/PyTorch Transformer API.

```python
def pad_sequences(sequences, pad_value=0):
    max_len = max(map(len, sequences), default=0)
    values = [seq + [pad_value] * (max_len - len(seq)) for seq in sequences]
    mask = [[1] * len(seq) + [0] * (max_len - len(seq)) for seq in sequences]
    return values, mask


def collate(samples):
    input_ids, text_mask = pad_sequences([s["input_ids"] for s in samples], pad_value=0)
    patches, image_mask = pad_sequences([s["image_patches"] for s in samples], pad_value=[0.0])
    audio, audio_mask = pad_sequences([s.get("audio_frames", []) for s in samples], pad_value=[0.0])
    return {
        "input_ids": input_ids,
        "text_attention_mask": text_mask,
        "image_patches": patches,
        "image_attention_mask": image_mask,
        "audio_frames": audio,
        "audio_attention_mask": audio_mask,
    }
```

На практике `pad_value` должен иметь форму одного patch/frame, а не `[0.0]`; это упрощённый Python-вариант. Bucketing по длине текста, числу patches и длительности аудио уменьшает пустые вычисления.

### HW-LC6. Affine int8 quantization

$$
\Large q = \operatorname{clip}\left(\operatorname{round}\left(\frac{x}{s}\right) + z, -128, 127\right), \qquad \hat{x} = s(q-z).
$$

`s` - scale, `z` - zero point, `q` - int8-представление, `hat{x}` - восстановленное значение.

```python
def quantize_int8(values, min_val=None, max_val=None):
    min_val = min(values) if min_val is None else min_val
    max_val = max(values) if max_val is None else max_val
    if max_val == min_val:
        return [0] * len(values), 1.0, 0
    qmin, qmax = -128, 127
    scale = (max_val - min_val) / (qmax - qmin)
    zero_point = round(qmin - min_val / scale)
    zero_point = max(qmin, min(qmax, zero_point))
    q = [max(qmin, min(qmax, round(x / scale + zero_point))) for x in values]
    return q, scale, zero_point


def dequantize_int8(q, scale, zero_point):
    return [(value - zero_point) * scale for value in q]
```

Per-channel quantization считает отдельный scale для каждого выходного канала весов. Это точнее при неодинаковых диапазонах каналов, но усложняет kernel и metadata.

## 🔎 Code review

### HW-CR1. OCR на мобильном фото

Нужно: применить `ImageOps.exif_transpose`, валидировать формат и лимит размера, сделать resize/deskew/quality check, не логировать текст и user id вместе без legal basis, не возвращать исходное фото по умолчанию, добавить timeout/error code и хранить PII только по согласованной политике. API должен возвращать структурированный результат: text, confidence, warnings и model/preprocessing version.

### HW-CR2. Padding mask

В данном коде mask инвертирована: реальным токенам назначен `0`, padding - `1`. Для обычного `attention_mask` нужно наоборот. Также нужны явные `dtype=torch.long` для ids и совместимый тип mask, защита от пустого batch, padding изображений/аудио по собственным осям и тест, что attention не меняется при добавлении padding.

### HW-CR3. Квантизация весов

`w.max()` игнорирует отрицательный минимум, scale может быть нулевым, нет округления и clipping, преобразование `astype(int8)` может переполниться. Минимально нужна симметричная схема `scale = max(abs(w)) / 127`, `q = clip(round(w / scale), -127, 127)`. Для real acceleration добавляют per-channel quantization, calibration активаций и проверку quality/latency на целевом устройстве, а не только MSE весов.

### HW-CR4. Cloud/edge fallback

Вызывать cloud только по confidence недостаточно: нужно проверить consent и privacy policy до передачи изображения, сетевой budget/timeout, стоимость, thermal/battery, доступную память, версию локальной и облачной модели. Нужен явный fallback: локальный безопасный ответ, постановка в очередь или просьба повторить позже. В telemetry отправляют агрегированные технические метрики и request id, но не сырой контент без разрешения.
