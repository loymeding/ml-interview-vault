---
tags: [interview-prep, huawei, live-coding, code-review, multimodal, document-ai, edge-ai]
company: Huawei
type: live-coding-problems
status: active
tasks_total: 10
cssclasses: [wide-page]
---

# 📱 Huawei — лайвкодинг и code review

> [!important] Режим тренировки
> Задачи отражают вакансию: Document AI, speech/video, VLM и edge inference. Сначала решай по условию, затем сверяйся с [[Interview Reviews/Подготовка/Huawei/Huawei — решения лайвкодинга и code review|разборами и решениями]].

## 🎯 Как тренироваться

| Формат | Время | Что проверяет |
|---|---:|---|
| Live coding | 25-40 минут | алгоритм, устойчивый код и метрики |
| Code review | 10-15 минут | качество production-пайплайна, privacy и edge-ограничения |

---

## 👁️🎙️ Live coding: Document AI, speech и edge

### HW-LC1. CER и WER

- [ ] Решено самостоятельно

Реализуй расстояние Левенштейна и функции `cer(reference, hypothesis)` и `wer(reference, hypothesis)` без внешних библиотек.

Определи правила нормализации текста и поведение для пустого reference. Добавь тесты на insertion, deletion, substitution и полностью совпадающие строки.

### HW-LC2. IoU и Non-Maximum Suppression

- [ ] Решено самостоятельно

Реализуй `iou(box_a, box_b)` для боксов `[x1, y1, x2, y2]` и `nms(boxes, scores, iou_threshold)`.

Обработай невалидные/вырожденные боксы, равные score и пустой список. Объясни, чем class-agnostic NMS отличается от class-wise NMS.

### HW-LC3. Порядок чтения документа

- [ ] Решено самостоятельно

Даны блоки OCR после layout detection:

```python
{"text": str, "bbox": (x1, y1, x2, y2), "page": int}
```

Реализуй упрощённое восстановление порядка чтения для одноколоночного и двухколоночного документа. Нужно сначала разделить страницы, затем колонки, затем отсортировать блоки сверху вниз.

Назови случаи, когда геометрической эвристики недостаточно.

### HW-LC4. Выравнивание аудио, видео и транскрипта

- [ ] Решено самостоятельно

Есть сегменты транскрипта `(start, end, text)` и видеокадры/сцены `(timestamp, frame_id)`. Для каждого текстового сегмента верни кадры, попадающие в его временной интервал, с допустимым контекстом `context_seconds` по краям.

Нужна реализация лучше, чем полный перебор всех сегментов и кадров.

### HW-LC5. Collate-функция для мультимодального batch

- [ ] Решено самостоятельно

Каждый пример содержит `input_ids`, `image_patches` и опциональный `audio_frames` разной длины. Реализуй на Python списках или PyTorch-псевдокоде collate-функцию, которая делает padding и создаёт корректные attention masks.

Объясни, какие оси должны быть замаскированы, почему padding нельзя считать данными и как снизить waste через bucketing.

### HW-LC6. Affine int8 quantization

- [ ] Решено самостоятельно

Реализуй функции `quantize_int8(x, min_val, max_val)` и `dequantize_int8(q, scale, zero_point)` для float-вектора.

Верни квантизованные значения, scale и zero point; оцени максимальную и среднюю абсолютную ошибку восстановления. Обсуди, как перейти от per-tensor к per-channel quantization.

---

## 🔎 Code review: production-мультимодальность

### HW-CR1. OCR-пайплайн на мобильном фото

- [ ] Разобрано самостоятельно

```python
def recognize_document(photo_bytes, user_id):
    image = Image.open(io.BytesIO(photo_bytes))
    text = ocr_model(image)
    logger.info("ocr", extra={"user_id": user_id, "text": text})
    return {"text": text, "raw_image": photo_bytes}
```

Найди проблемы, связанные с rotation/EXIF, качеством изображения, privacy/PII, памятью, ошибками модели и контрактом API. Предложи production-версию пайплайна.

### HW-CR2. Padding и attention mask

- [ ] Разобрано самостоятельно

```python
def collate(samples, pad_id=0):
    max_len = max(len(s["input_ids"]) for s in samples)
    ids = []
    mask = []
    for sample in samples:
        pad = [pad_id] * (max_len - len(sample["input_ids"]))
        ids.append(sample["input_ids"] + pad)
        mask.append([0] * len(sample["input_ids"]) + [1] * len(pad))
    return {"input_ids": torch.tensor(ids), "attention_mask": torch.tensor(mask)}
```

Проверь семантику mask для типичных Transformer API, dtypes, пустые последовательности, порядок padding и то, как этот код расширить на изображение и аудио.

### HW-CR3. Квантизация весов

- [ ] Разобрано самостоятельно

```python
def quantize_weights(w):
    scale = w.max() / 127
    q = (w / scale).astype(np.int8)
    return q, scale
```

Найди численные и продуктовые проблемы: асимметричный диапазон, outliers, нулевой scale, clipping/rounding, per-channel scaling, накопление ошибок и оценка качества после оптимизации.

### HW-CR4. Cloud/edge fallback

- [ ] Разобрано самостоятельно

```python
def answer_request(request):
    if request.confidence < 0.8:
        return cloud_vlm(request.image, request.prompt)
    return local_vlm(request.image, request.prompt)
```

Разбери риски: confidence до или после ответа, privacy и consent, сеть и timeout, thermal/battery budget, стоимость, версии моделей, наблюдаемость и безопасный fallback без ответа.

---

## 🧾 После каждой попытки

1. Сформулируй контракт входов и выходов.
2. Покрой крайние случаи и ошибки данных.
3. Отдельно назови latency, память и privacy-ограничения.
4. Проговори trade-offs как инженер продукта, а не только как автор алгоритма.
