---
tags: [аналитика, monitoring, mlops, drift, delayed-labels, alerts, sla, quality]
тип: теория
уровень: senior
сложность: высокая
статус: готово
готовность: 95
создано: 2026-08-25
источники:
  - "Designing Machine Learning Systems — Chip Huyen"
  - "Machine Learning System Design — Babushkin, Kravchenko"
предпосылки:
  - "ML-метрики в продукте — от offline score к бизнес-решению"
  - "Качество данных — дубликаты, пропуски, late events, часовые пояса и join explosion"
связано:
  - "Аналитический каркас ML System Design — цель, данные, эксперимент и решение"
сравнить-с: []
---

# Мониторинг продукта и ML-системы — алерты, срезы, drift и delayed labels

> [!abstract] Суть
> Мониторинг — это цепочка от симптома до действия: какие уровни системы наблюдаем, с каким baseline сравниваем, кто получает алерт и какой fallback или rollback запускается.

## Почему «будем мониторить accuracy» — слабый ответ

Истинный label может появиться через неделю или месяц. А система может сломаться сейчас из-за пустых features, timeout или нового формата данных. Поэтому мониторинг строится слоями — от доступности до бизнес-результата.

```text
инфраструктура
→ данные
→ модель и решения
→ пользовательское поведение
→ бизнес и safety
```

## 1. Системные метрики

- request rate / throughput;
- error rate;
- latency p50/p95/p99;
- timeouts;
- CPU/GPU/memory;
- queue lag;
- cache hit rate;
- стоимость inference;
- availability/SLA.

Средней latency недостаточно: пользовательский вред часто живёт в хвосте. Метрики нужно смотреть по endpoint, model version, region, hardware и request type.

## 2. Качество данных

- freshness и ingestion lag;
- row/event count;
- null rate;
- duplicate rate;
- schema changes;
- invalid ranges/categories;
- key coverage и join miss rate;
- доля default/fallback features;
- feature parity offline/online.

Если feature `user_orders_30d` внезапно стала нулём у всех, model score может формально вычисляться без ошибок, но решения станут бессмысленными.

## 3. Distribution drift

### Covariate drift

Изменилось $P(X)$: пользователи, запросы или признаки стали другими.

### Prior/label drift

Изменилось $P(Y)$: например, fraud rate вырос.

### Concept drift

Изменилось $P(Y\mid X)$: прежняя связь признаков с outcome больше не работает.

Drift не равен деградации. Праздничный трафик может сильно изменить $P(X)$, но модель остаётся качественной. И наоборот, небольшое изменение важного признака разрушит качество.

Методы:

- PSI, KS, Jensen–Shannon divergence;
- population stability по категориям;
- score distribution;
- embedding centroids/distances;
- domain classifier «train vs current»;
- missingness и unseen categories.

Drift-алерт — ранний сигнал, а не доказательство retraining necessity.

## 4. Prediction и decision monitoring

- распределение score;
- positive rate / approval rate;
- threshold crossing rate;
- confidence/entropy;
- abstain/fallback rate;
- candidate count;
- empty-result rate;
- action distribution;
- calibration proxy.

Скачок approval rate может возникнуть до labels и немедленно указать на поломку.

## 5. Quality при delayed labels

Когда labels созревают поздно, строят несколько контуров:

### Быстрые proxy

Например, для ответа бота: escalation, повторная формулировка, dislike, отсутствие клика по источнику. Они быстрые, но biased.

### Human review / golden set

Постоянная выборка свежих кейсов размечается экспертами. Нужны sampling и strata, иначе смотрим только удобные случаи.

### Matured labels

Через заданное окно пересчитываются настоящие outcome: возврат, chargeback, repeat contact. Сравнивать нужно когорты одинаковой зрелости.

### Shadow labels / rules

Старую модель или правило можно запускать параллельно для расхождений, но disagreement не говорит, кто прав.

## 6. Product и business metrics

- conversion, retention, revenue, margin;
- complaints, returns, cancellations;
- time to success;
- operator/moderator load;
- coverage и fairness;
- safety incidents;
- long-term holdout metrics.

Изменение product metric не обязательно вина модели: цена, интерфейс или supply тоже влияют. Поэтому нужен lineage от model version до решения и outcome.

## Срезы

Средняя метрика почти никогда не достаточна. Срезы выбирают заранее:

- model version и experiment variant;
- platform/app version;
- region/language;
- new/returning;
- head/tail и intent;
- confidence band;
- cold start;
- high-cost/high-risk cases;
- protected groups, если применимо.

Отдельно мониторят worst-group и долю трафика сегмента. Качество могло упасть в маленьком сегменте, который быстро растёт.

## Baseline для алерта

Порог «CTR < 10%» плохо работает при сезонности. Варианты:

- статический SLO для технических критических показателей;
- same hour/day-of-week baseline;
- rolling median + MAD;
- forecast interval;
- control chart;
- comparison to control/holdout;
- change-point detection.

### Robust z-score через MAD

$$\Large z_{robust}=\frac{x-median(x)}{1.4826\cdot MAD}$$

Он устойчивее к выбросам, но всё равно требует учёта сезонности и достаточной истории.

## Алерт должен быть actionable

У каждого алерта:

- понятное имя и metric owner;
- severity;
- baseline и условие срабатывания;
- минимальная длительность, чтобы не шуметь;
- ссылка на slices и raw evidence;
- runbook;
- fallback/rollback;
- канал и on-call owner.

Если на алерт никто не реагирует, это не мониторинг, а шум.

## Rollout и rollback

Безопасная схема:

```text
offline gate → shadow → 1% canary → 5% → 25% → 50% → 100%
```

На каждом этапе проверяются system и guardrail metrics. Нужны:

- model registry и immutable version;
- возможность быстро вернуть предыдущую модель;
- feature flag;
- rule-based fallback;
- graceful degradation;
- сохранённые assignment/version logs.

## Retraining policy

Триггеры:

- schedule;
- накопление новых labels;
- подтверждённое падение quality;
- concept drift;
- изменение каталога/таксономии;
- regulatory/business change.

Автоматический retraining без acceptance gates опасен. Новая модель должна пройти data checks, offline evaluation, slices, backtest и controlled rollout.

## Пример: RAG-бот

**System:** latency retrieval/generation, timeout, token count, cost, cache.

**Data:** freshness index, доля документов без embeddings, parse failures, language mix.

**Retrieval:** candidate count, empty results, Recall@K на sampled golden set, score distribution.

**Generation:** refusal, citations, judge scores, safety filter, unsupported claims на human sample.

**Product:** resolution without repeat contact, escalation, CSAT, complaints.

**Slices:** intent, answerability, language, long/short query, document age.

**Fallback:** безопасный refusal, поиск без генерации, перевод оператору, предыдущая model version.

## Как расследовать алерт

```text
product outcome упал
├── traffic/composition изменился?
├── данные и logging целы?
├── serving/latency/errors изменились?
├── score/decision distribution сдвинулась?
├── model/feature version менялась?
└── внешний продукт/supply/price изменился?
```

Сначала локализуем уровень. Если null rate features выросла, нет смысла начинать с retraining.

## Как ответить на интервью

> [!question] Что мониторить после запуска модели?
> Разделю на system, data, prediction/decision, delayed quality и product outcomes. На каждом уровне назову метрики и срезы. Для labels с задержкой использую proxy и human sample, но позже пересчитываю matured outcomes. Для алертов задам baseline, owner, runbook и rollback; staged rollout защищает от массового ущерба.

> [!question] Увидели drift. Переобучать?
> Не автоматически. Проверю, затронул ли drift важные признаки, изменились ли scores, decisions и quality. Это может быть ожидаемая сезонность. Если quality подтверждённо падает, выясню data issue vs concept drift, затем переобучу с acceptance gates и controlled rollout.

> [!question] Как мониторить модель без labels?
> Системные и data checks, score/action distributions, coverage/fallback и быстрые behavioral proxy. Добавлю human review стратифицированной выборки, shadow comparison и дождусь matured labels. Честно укажу, что proxy не заменяет конечный outcome.

## Типичные ошибки

- Мониторить только CPU и среднюю latency.
- Считать любой drift причиной retraining.
- Сравнивать недозрелые labels с зрелыми.
- Смотреть только global average.
- Создать сотни алертов без owner/runbook.
- Не логировать model/feature version.
- Не иметь fallback и rollback.

## Проверка себя

- Чем covariate drift отличается от concept drift?
- Какие метрики доступны до появления labels?
- Почему score distribution полезна, но не доказывает качество?
- Составьте monitoring stack для fraud-модели.

## Связано

- [[Аналитический каркас ML System Design — цель, данные, эксперимент и решение]]
- [[Качество данных — дубликаты, пропуски, late events, часовые пояса и join explosion]]
- [[Декомпозиция метрик и поиск причин изменений]]
- [[Interview Reviews/Подготовка/ML System Design/Карточки/01 — Универсальный каркас ответа на ML System Design|Универсальный каркас ML System Design]]

---
[[Аналитика/🗺️ Индекс|Назад к индексу аналитики]]

