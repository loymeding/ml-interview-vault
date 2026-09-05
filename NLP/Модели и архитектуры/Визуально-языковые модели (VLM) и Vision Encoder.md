---
tags: [nlp, multimodal, vlm, vision-language-model, vision-transformer, vision-encoder, ocr, llm]
тип: теория
уровень: middle
сложность: средняя
статус: готово
готовность: 85
создано: 2026-07-09
источники:
  - "Шаренный чат ChatGPT: блоки про VLM и Vision Encoder"
предпосылки:
  - "Архитектура Transformer — энкодер, декодер и слои"
  - "Механизм внимания (Attention и Self-Attention)"
связано:
  - "Архитектура Transformer в LLM — Attention, MHA и KV-cache"
  - "Архитектура Seq2Seq (Энкодер-Декодер)"
сравнить-с:
  - "Архитектура Transformer — энкодер, декодер и слои"
---

# Визуально-языковые модели (VLM) и Vision Encoder

> [!abstract] Суть
> VLM соединяет зрение и язык: vision encoder превращает изображение в набор визуальных признаков, projector переводит их в пространство LLM, а языковая модель использует эти визуальные токены вместе с текстовым prompt.

## Ответ

### 1. Что такое VLM

**VLM** (`Vision-Language Model`) — модель, которая принимает изображение и текст, а отвечает текстом или выполняет multimodal-задачу.

Примеры задач:

- описать изображение;
- ответить на вопрос по картинке;
- прочитать текст на изображении;
- найти объект;
- объяснить график;
- работать с документом/скриншотом;
- сравнить изображение и текстовое описание.

Главная идея:

```text
image -> visual features
text prompt -> text tokens
visual features + text tokens -> LLM -> answer
```

### 2. Общая архитектура

Типичная VLM состоит из трёх частей:

```text
Image
  -> Vision Encoder
  -> Projector / Adapter
  -> LLM
  -> Text answer
```

**Vision Encoder** извлекает признаки из изображения. Это может быть CNN, Vision Transformer или encoder от CLIP-like модели.

**Projector** переводит визуальные embeddings в размерность и формат, понятный LLM.

**LLM** принимает визуальные токены как дополнительный context и генерирует ответ.

### 3. Vision Encoder

Vision Encoder должен превратить изображение в последовательность векторов.

В ViT-like подходе изображение делится на патчи.

Если изображение размера $H \times W$, а patch size равен $P \times P$, то число патчей:

$$\Large N = \frac{H}{P} \cdot \frac{W}{P}$$

Например, изображение `224x224` и patch size `16x16`:

$$\Large N = \frac{224}{16} \cdot \frac{224}{16} = 14 \cdot 14 = 196$$

Каждый patch разворачивается в вектор и проходит через линейный слой:

$$\Large z_i = W x_i + b$$

где:
- $x_i$ — пиксели i-го patch;
- $z_i$ — embedding patch;
- $W,b$ — обучаемые параметры.

Затем добавляются positional embeddings, потому что Transformer сам по себе не знает расположение patch.

### 4. Projector

Выход vision encoder обычно не совпадает с embedding space языковой модели.

Projector решает задачу согласования:

$$\Large v_i = W_p h_i + b_p$$

где:
- $h_i$ — визуальный embedding от vision encoder;
- $v_i$ — визуальный токен в размерности LLM;
- $W_p,b_p$ — параметры projector.

Projector может быть:

- простой linear layer;
- MLP;
- Q-Former / query transformer;
- cross-attention adapter;
- resampler, который сжимает много visual tokens в меньшее число.

### 5. Как LLM использует изображение

После projector визуальные признаки подаются в LLM как специальные visual tokens:

```text
<image_token_1> <image_token_2> ... <image_token_n>
User: Что изображено?
Assistant: ...
```

LLM не видит "картинку" напрямую. Она видит embedding-представления, которые должны быть выровнены с языковым пространством.

### 6. Как обучают VLM

Часто используют несколько этапов.

**Этап 1: alignment / pretraining**

Цель — научить projector связывать visual features и language model.

Данные:

```text
image + caption
image + alt text
image + OCR text
```

Часто vision encoder и LLM могут быть частично заморожены, а обучается projector.

**Этап 2: instruction tuning**

Модель учат отвечать на вопросы:

```text
image + instruction -> answer
```

Примеры:

```text
Что на изображении?
Сколько объектов?
Прочитай текст на скриншоте.
Объясни график.
```

**Этап 3: preference/safety tuning**

Модель донастраивают, чтобы ответы были полезными, безопасными и не галлюцинировали детали изображения.

### 7. OCR и VLM

VLM может "читать" текст на изображении, но это не всегда классический OCR.

Классический OCR:

```text
image -> detected text + bounding boxes
```

VLM:

```text
image + question -> answer
```

Если задача требует точного извлечения текста, таблиц, чеков или документов, часто лучше комбинировать:

```text
OCR engine -> structured text
+ VLM/LLM -> reasoning over extracted content
```

Нюанс: VLM может уверенно "прочитать" несуществующий текст, особенно на мелких, размытых или повернутых изображениях. Для документов нужны проверки и источники.

### 8. Чем VLM отличается от обычной LLM

Обычная LLM работает с текстовыми токенами:

```text
text tokens -> Transformer -> text
```

VLM добавляет визуальную модальность:

```text
image -> visual tokens
text -> text tokens
visual + text tokens -> multimodal Transformer/LLM
```

Главная сложность не в том, чтобы "прикрутить картинку", а в alignment: визуальные признаки должны стать понятными языковой модели.

### 9. Ограничения VLM

- галлюцинации объектов и текста;
- слабая точность на мелких деталях;
- проблемы с counting;
- чувствительность к crop/resolution;
- трудности с таблицами и сложными документами;
- зависимость от качества vision encoder;
- высокая стоимость context, если visual tokens много;
- риск prompt injection через текст на изображении.

## Формула / Схема

Общая схема:

```text
I -> f_v(I) = H_v
H_v -> projector(H_v) = V
prompt -> tokenizer(prompt) = T
[V; T] -> LLM -> answer
```

где:
- $I$ — изображение;
- $f_v$ — vision encoder;
- $H_v$ — визуальные embeddings;
- $V$ — visual tokens в пространстве LLM;
- $T$ — text tokens.

## Короткий пример

Пользователь отправляет фото товара и спрашивает:

```text
Есть ли на товаре повреждения?
```

VLM pipeline:

```text
photo -> vision encoder -> visual tokens
question -> text tokens
visual tokens + question -> LLM
answer: "На левом нижнем углу видна царапина..."
```

Для production лучше добавить:

```text
confidence
bounding boxes / highlighted regions
fallback to human review for low confidence
```

## Типичные ошибки

- **Думать, что LLM видит пиксели напрямую:** обычно пиксели уже преобразованы vision encoder в embeddings.
- **Путать VLM и OCR:** OCR извлекает текст, VLM рассуждает по изображению и тексту, но может ошибаться в точном чтении.
- **Не учитывать resolution:** мелкие детали могут исчезнуть при resizing.
- **Использовать VLM для точного документа без OCR/валидации:** риск галлюцинаций.
- **Не проверять prompt injection на изображениях:** текст внутри картинки может пытаться управлять моделью.

## Каверзные вопросы

> [!question] Что делает projector в VLM?
> Он переводит visual embeddings из vision encoder в размерность и распределение, понятные LLM. Без projector LLM не сможет интерпретировать визуальные признаки как часть своего input space.

> [!question] Почему ViT делит изображение на патчи?
> Transformer работает с последовательностью токенов. Патчи превращают 2D-изображение в последовательность visual tokens, к которым можно применить self-attention.

> [!question] Можно ли использовать VLM вместо OCR?
> Иногда да для грубого понимания, но для точного извлечения текста из документов лучше использовать OCR + structured extraction + проверки. VLM может галлюцинировать текст.

## Проверка себя

- Какие три основных блока есть в VLM?
- Что такое visual tokens?
- Зачем нужен projector?
- Чем OCR отличается от VLM?
- Почему resolution важен для качества VLM?

## Предпосылки

- [[NLP/Модели и архитектуры/Архитектура Transformer — энкодер, декодер и слои]]
- [[NLP/Модели и архитектуры/Механизм внимания (Attention и Self-Attention)]]

## Связано

- [[NLP/LLM и Промпт-инжиниринг/Архитектура Transformer в LLM — Attention, MHA и KV-cache]]
- [[NLP/Модели и архитектуры/Архитектура Seq2Seq (Энкодер-Декодер)]]

## Источники

- Шаренный чат ChatGPT: блоки про VLM и Vision Encoder.

---
[[🗺️ Индекс|Назад к разделу]]
