---
tags: [nlp, llm, evaluation, llm-eval, hallucinations, groundedness, safety, metrics, production]
тип: теория и практика
уровень: middle-to-senior
сложность: высокая
статус: готово
готовность: 90
создано: 2026-07-09
источники:
  - "Разбор вакансии Ozon Tech: Data Scientist, модели оценки качества"
  - "Шаренный чат ChatGPT: Объяснение алгоритма BPE и оценка LLM"
  - "Текущий чат: подготовка к интервью по LLM evaluation"
предпосылки:
  - "Метрики оценки NLP-моделей (Perplexity, BLEU, ROUGE, BERTScore, GLUE)"
  - "RAG — генерация, оценка качества и production"
связано:
  - "LLM-as-a-Judge — rubric, bias и дообучение"
  - "Метрики RAG-систем — retrieval, generation, faithfulness и RAGAS"
  - "Агенты, function calling и structured output"
сравнить-с:
  - "Метрики RAG-систем — retrieval, generation, faithfulness и RAGAS"
---

# Оценка LLM-ответов — relevance, completeness, factuality и safety

> [!abstract] Суть
> Оценка LLM-ответа — это не одна метрика, а pipeline независимых проверок: ответ должен быть релевантным вопросу, полным по рубрике, фактически корректным, grounded в источниках, безопасным, стилистически уместным и следующим инструкции.

## Ответ

### 1. Что именно оцениваем

В LLM-приложениях важно разделять несколько уровней качества:

| Уровень | Что проверяем | Примеры метрик |
|---|---|---|
| Отдельный ответ | ответил ли бот на вопрос и не соврал ли | relevance, completeness, factuality, groundedness, safety |
| Диалог целиком | решил ли агент задачу пользователя | task success, number of turns, escalation correctness |
| Система в production | стало ли лучше пользователям и бизнесу | CSAT, transfer to operator, repeat contact rate, latency, cost |

На интервью лучше начинать именно с этого разделения. Иначе ответ звучит так, будто качество LLM можно измерить одним `BLEU` или одним judge-score.

### 2. Входы evaluation pipeline

Обычно evaluator получает не только ответ модели:

```text
user_query
candidate_answer
retrieved_context / documents
expected_behavior / rubric
metadata: intent, product, locale, channel, risk level
```

Для RAG-систем критически важно передавать `retrieved_context`, потому что factuality в продукте часто означает не "похоже на правду вообще", а "подтверждено конкретными источниками".

Схема:

```text
question + answer + context + rubric
      -> evaluators
      -> scores + reasons + pass/fail
      -> aggregate by scenario / intent / risk segment
      -> release decision
```

### 3. Relevance — ответил ли бот на вопрос

**Relevance** отвечает: относится ли ответ к запросу пользователя.

Пример плохой релевантности:

```text
User: Когда вернут деньги за возврат?
Bot: Возврат можно оформить в приложении.
```

Ответ связан с темой возврата, но не отвечает на вопрос о сроке денег.

Как реализовать:

- `LLM-as-a-Judge`: вопрос + ответ + рубрика -> score `1-5`;
- intent-level checker: совпадает ли intent ответа с intent запроса;
- для FAQ/RAG: проверять, что ответ использует документы, релевантные вопросу;
- rule checks для обязательных сценариев: если вопрос содержит "когда", ответ должен содержать срок или отказ "нет данных".

Пример JSON-выхода:

```json
{
  "relevance": {
    "score": 3,
    "passed": false,
    "reason": "Ответ объясняет оформление возврата, но не отвечает на вопрос о сроке возврата денег"
  }
}
```

### 4. Completeness — покрыты ли обязательные пункты

**Completeness** отвечает: все ли важные части вопроса покрыты.

Например, пользователь спрашивает:

```text
Как вернуть товар и когда вернут деньги?
```

В rubric можно задать required points:

```json
{
  "required_points": [
    "как открыть заявку на возврат",
    "какие условия возврата",
    "срок возврата денег",
    "что делать при отказе или споре"
  ]
}
```

Простая метрика:

$$\Large Completeness = \frac{\#\text{covered required points}}{\#\text{all required points}}$$

Если ответ покрыл 3 пункта из 4:

$$\Large Completeness = \frac{3}{4}=0.75$$

Нюанс: полнота не равна длине. Длинный ответ может быть неполным, если он обошел главный пункт. Короткий ответ может быть полным, если вопрос простой.

### 5. Factuality, groundedness и hallucination rate

Три близких, но разных понятия:

| Термин | Что означает |
|---|---|
| `factuality` | фактически ли утверждение верно относительно мира или доменной истины |
| `groundedness / faithfulness` | следует ли утверждение из переданного контекста |
| `hallucination` | модель добавила неподтвержденное или противоречащее утверждение |

Для RAG лучше проверять не ответ целиком, а отдельные atomic claims.

Pipeline:

1. Разбить ответ на claims.
2. Для каждого claim найти поддержку в context.
3. Разметить claim как `supported`, `unsupported`, `contradicted`, `not_applicable`.
4. Посчитать агрегаты.

Пример:

```json
{
  "claims": [
    {
      "text": "Деньги возвращаются в течение 10 рабочих дней",
      "status": "supported",
      "source": "refund_policy_v3"
    },
    {
      "text": "Возврат доступен в любом пункте выдачи",
      "status": "unsupported",
      "source": null
    }
  ]
}
```

Метрики:

$$\Large Groundedness = \frac{\#\text{supported claims}}{\#\text{factual claims}}$$

$$\Large HallucinationRate = \frac{\#\text{unsupported claims}+\#\text{contradicted claims}}{\#\text{factual claims}}$$

Нюанс: если ответ правильный "по памяти модели", но не подтвержден retrieved context, для RAG это всё равно проблема. Продукт может требовать доказуемости и цитирования.

### 6. Safety — можно ли показывать ответ пользователю

Safety часто делают hard gate: если проверка не пройдена, ответ нельзя отдавать, даже если он релевантный.

Что проверять:

- токсичность и оскорбления;
- персональные данные и утечки;
- опасные инструкции;
- юридические, медицинские, финансовые советы без оснований;
- запрещенные обещания: компенсация, возврат денег, скидка, гарантия результата;
- нарушение политики компании;
- prompt injection в retrieved context или user input.

Реализация:

```text
rules / regex / allow-deny lists
+ safety classifier
+ LLM judge для смысловых нарушений
+ deterministic business validators
```

Пример hard rule:

```text
Если ответ содержит обещание точной компенсации, но API/tool не вернул подтверждение,
то safety_passed = false.
```

### 7. Style и instruction following

Style проверяет, соответствует ли ответ voice & tone продукта:

- вежливый;
- краткий;
- без канцелярита;
- не спорит с пользователем;
- не использует внутренние термины;
- не перегружает подробностями.

Instruction following проверяет выполнение формальных требований:

- вернуть валидный JSON;
- указать цитаты;
- не отвечать без источника;
- эскалировать при неопределенности;
- не раскрывать chain-of-thought;
- соблюдать формат канала: чат, email, push.

Часть проверок лучше делать детерминированно:

```text
JSON schema validation
required fields
forbidden phrases
citation format
max length
```

Смысловые инструкции лучше проверять через rubric-based judge.

### 8. Offline evaluation vs online evaluation

**Offline evaluation** проводится до релиза на фиксированном наборе кейсов:

- golden dataset;
- regression set;
- safety set;
- edge cases;
- исторические ошибки production;
- синтетические сценарии для редких случаев.

Плюсы: быстро, воспроизводимо, безопасно.

Минусы: не всегда отражает живой пользовательский трафик.

**Online evaluation** проводится на реальном трафике:

- A/B test;
- canary release;
- shadow mode;
- ручная разметка production-sample;
- мониторинг жалоб и эскалаций.

Плюсы: видно влияние на реальные метрики.

Минусы: дороже, дольше и рискованнее.

Хорошая схема:

```text
offline eval gate -> canary/shadow -> A/B test -> production monitoring
```

### 9. Golden dataset

Golden set — это не просто "набор правильных ответов". Для LLM часто важнее rubric:

```text
question
context
expected behavior
must-have facts
forbidden claims
when to escalate
example good answer
example bad answer
```

Что включать:

- частые intents;
- редкие, но рискованные сценарии;
- неоднозначные вопросы;
- out-of-scope запросы;
- токсичные и adversarial запросы;
- исторические ошибки модели;
- кейсы, где нужно отказаться отвечать.

Golden set нельзя смешивать с train set и prompt tuning examples. Иначе можно переобучиться под benchmark.

### 10. Release gate

Пример release gate для чат-бота поддержки:

```text
relevance >= 4.0
completeness >= 0.8
groundedness >= 0.95
hallucination_rate <= 0.03
safety_passed = true
p95_latency <= 2.0s
candidate win-rate vs baseline >= 55%
no critical regressions on high-risk scenarios
```

Важно смотреть не только среднее, но и сегменты:

- intent;
- язык;
- тип пользователя;
- источник документов;
- риск-сценарий;
- короткие/длинные диалоги.

Средняя метрика может вырасти, а качество на критичном сегменте просесть.

## Формула / Схема

Общая агрегированная оценка может быть взвешенной:

$$\Large Score = w_r R + w_c C + w_g G + w_s S + w_i I$$

где:
- $R$ — relevance;
- $C$ — completeness;
- $G$ — groundedness;
- $S$ — safety score или binary gate;
- $I$ — instruction following;
- $w_i$ — веса, зависящие от продукта.

Но в production часто лучше не сводить всё к одному числу: safety и groundedness должны быть отдельными gate-условиями.

Схема диагностики:

```text
Плохой ответ
  |
  +-- Нужный факт был в context?
      |
      +-- нет -> проблема retrieval/chunking/index
      +-- да
          |
          +-- ответ не следует из context -> prompt/generation/grounding
          +-- ответ неполный -> rubric/context recall
          +-- ответ опасный -> safety/instruction gate
```

## Короткий пример

Вопрос:

```text
Можно ли вернуть товар без упаковки и когда вернут деньги?
```

Контекст:

```text
Возврат без заводской упаковки возможен, если товар не имеет следов использования.
Деньги возвращаются в течение 10 рабочих дней после проверки товара.
```

Ответ модели:

```text
Да, можно вернуть без упаковки. Деньги вернутся завтра.
```

Оценка:

```json
{
  "relevance": 5,
  "completeness": 0.75,
  "groundedness": 0.5,
  "hallucination_rate": 0.5,
  "safety_passed": false,
  "reason": "Ответ релевантен, но срок 'завтра' не подтвержден источником и может создать ложное обещание"
}
```

## Типичные ошибки

- **Оценивать только финальный текст:** непонятно, виноват retrieval, prompt, LLM или tool.
- **Считать factuality и groundedness одним и тем же:** ответ может быть правдивым, но не подтвержденным контекстом.
- **Полагаться на один judge-score:** средняя оценка скрывает критические ошибки.
- **Забывать про сегменты:** модель может улучшиться на простых FAQ и ухудшиться на возвратах денег.
- **Не версионировать eval:** без версии датасета, prompt, judge и модели сравнения между релизами нечестны.
- **Путать полноту с длиной:** длинный ответ не гарантирует coverage.

## Каверзные вопросы

> [!question] Если offline-метрики выросли, а online CSAT упал, что делать?
> Не релизить дальше вслепую. Надо разложить online-трафик по сегментам, проверить mismatch между golden set и реальными запросами, посмотреть latency/escalation/repeat contact rate и руками разобрать выборку плохих диалогов. Offline eval мог переоценить качество из-за синтетических или устаревших кейсов.

> [!question] Что важнее: answer correctness или groundedness?
> Зависит от продукта. В RAG по регламентам groundedness часто обязательна: даже правильный ответ без источника рискован. В открытой QA correctness может быть важнее, но для корпоративного бота лучше требовать и корректность, и доказуемость.

> [!question] Почему BLEU/ROUGE плохо подходят для оценки чат-бота?
> У чат-бота нет одного эталонного ответа. BLEU/ROUGE смотрят на n-граммные совпадения с reference и плохо понимают релевантность, безопасность, фактическую опору и многообразие правильных формулировок.

## Проверка себя

- Чем relevance отличается от completeness?
- Почему hallucination rate лучше считать по atomic claims?
- Какие проверки должны быть hard gate перед релизом?
- Чем offline evaluation отличается от online evaluation?
- Что должно входить в golden dataset для LLM-бота поддержки?

## Предпосылки

- [[NLP/LLM и Промпт-инжиниринг/Метрики оценки NLP-моделей (Perplexity, BLEU, ROUGE, BERTScore, GLUE)]]
- [[NLP/LLM и Промпт-инжиниринг/RAG — генерация, оценка качества и production]]

## Связано

- [[NLP/LLM и Промпт-инжиниринг/LLM-as-a-Judge — rubric, bias и дообучение]]
- [[NLP/LLM и Промпт-инжиниринг/Метрики RAG-систем — retrieval, generation, faithfulness и RAGAS]]
- [[NLP/LLM и Промпт-инжиниринг/Агенты, function calling и structured output]]

## Источники

- Разбор вакансии Ozon Tech: Data Scientist, модели оценки качества.
- Шаренный чат ChatGPT: "Объяснение алгоритма BPE".
- Текущий чат: подготовка ответов по LLM evaluation, RAG и LLM-as-a-Judge.

---
[[🗺️ Индекс|Назад к разделу]]
