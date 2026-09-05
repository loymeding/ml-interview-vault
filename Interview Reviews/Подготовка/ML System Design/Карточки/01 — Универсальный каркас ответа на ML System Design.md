---
tags: [interview-prep, ml-system-design, framework, product-thinking, mlops]
тип: практика
уровень: senior
сложность: высокая
статус: готово
готовность: 100
создано: 2026-08-20
source_id: ml-system-design-book-babushkin-kravchenko
источники:
  - "Valerii Babushkin, Arseny Kravchenko — Machine Learning System Design"
предпосылки:
  - "А-Б тесты/Методология/Иерархия метрик — Целевая, Guardrail и Proxy"
  - "Big Data/Big Data — Spark, YARN, Kafka, HDFS, Delta Lake"
связано:
  - "Interview Reviews/Подготовка/Yandex GenAI/Продукты/Внутренний контур оценки качества GenAI в Яндексе"
---

# Универсальный каркас ответа на ML System Design

> [!abstract] Суть
> Сильный ответ — это не перечень моделей и технологий, а последовательное объяснение: какую пользовательскую проблему решаем, какими данными, какой системой, какими метриками проверяем результат и как поддерживаем решение после релиза.

## 0. Быстрый маршрут ответа

```text
Goal
  → constraints
  → target и data
  → baseline
  → model / serving
  → offline metrics
  → online experiment
  → monitoring
  → failure modes и next iteration
```

На интервью полезно сначала проговорить этот маршрут, а затем углубляться в наиболее важные для кейса блоки.

## 1. Product framing: какую проблему решаем?

Начать нужно не с модели, а с продукта:

- кто пользователь и какое действие он хочет совершить;
- какой момент пользовательского пути улучшаем;
- что считается успешным результатом;
- что произойдёт, если модель ошибётся;
- какие ограничения есть по latency, стоимости, безопасности и доступности данных.

Примеры формулировок:

- «Пользователь быстрее находит релевантный товар»;
- «Оператор получает полезную подсказку и принимает решение быстрее»;
- «Пользователь получает корректный ответ, а опасные ответы блокируются».

### Уточняющие вопросы

> [!question] Кто пользователь, какой у него intent и как сейчас решается задача?
> Без этого невозможно выбрать target, данные и метрику. Одна и та же модель может быть правильной для research-прототипа и неподходящей для массового продукта.

> [!question] Что важнее: качество, latency, стоимость или полнота покрытия?
> Ответ определяет архитектуру: тяжёлая модель, каскад, кэширование, batch-режим или быстрый online inference.

## 2. Формализовать ML-задачу

Перевести продуктовую цель в предсказание или решение:

| Продуктовая задача | Возможная ML-задача |
|---|---|
| показать лучшие товары | ranking / learning-to-rank |
| определить риск | классификация или anomaly detection |
| предсказать спрос | forecasting / regression |
| подобрать похожий контент | retrieval / embeddings |
| оценить ответ LLM | classification, pairwise preference или judge |

Нужно явно назвать:

- объект предсказания;
- target и горизонт прогноза;
- единицу принятия решения;
- момент времени, когда prediction должен быть доступен;
- допустимую цену ошибки.

## 3. Данные и target

Обсудить нужно не только «какие фичи подадим», но и происхождение label:

- откуда берутся positive и negative examples;
- есть ли delayed feedback;
- как устроена разметка и disagreement;
- какие сегменты недопредставлены;
- возможны ли leakage и train-serving skew;
- как поддерживать freshness;
- как разбивать train/validation/test по времени и пользователю.

Для пользовательских логов важно помнить: действие пользователя — не всегда ground truth. Клик может зависеть от позиции, экспозиции, latency и интерфейса. Это особенно важно для рекомендаций, поиска и GenAI-продуктов.

## 4. Baseline и выбор модели

Сначала предложить простой baseline:

- popularity / rules;
- линейная модель;
- gradient boosting;
- существующая production-версия.

Baseline нужен, чтобы понять, даёт ли новая сложность измеримый прирост. Затем сравнить варианты по критериям:

| Критерий | Вопрос |
|---|---|
| качество | насколько улучшается основная метрика? |
| latency | выдерживает ли модель SLA? |
| стоимость | сколько стоит inference и обучение? |
| объяснимость | можно ли расследовать ошибку? |
| устойчивость | как модель ведёт себя на drift и редких срезах? |
| поддержка | насколько сложно обновлять и откатывать? |

## 5. Архитектура системы

Разделить offline и online контуры:

```text
Offline:
raw data → validation → features/embeddings → training → evaluation → model registry

Online:
request → retrieval/candidate generation → model inference → business rules
         → response → logging
```

Для каждого блока назвать:

- вход и выход;
- источник данных;
- SLA;
- место кэширования;
- fallback при недоступности;
- что логируем для будущего анализа.

Для ranking-системы часто нужна многоступенчатая схема:

```text
candidate generation → lightweight ranker → expensive ranker → re-ranking / rules
```

Для GenAI аналогично можно разделить retrieval, генерацию, валидацию, safety-фильтры и fallback.

## 6. Метрики и evaluation

Разделить минимум четыре уровня:

| Уровень | Назначение |
|---|---|
| primary | главная пользовательская или бизнес-ценность |
| quality | качество предсказания или ответа |
| guardrail | безопасность, latency, cost, complaints |
| diagnostic | срезы и классы ошибок |

Порядок проверки:

1. offline evaluation на зафиксированном наборе;
2. error analysis по slices;
3. проверка статистической устойчивости;
4. shadow или пилотный запуск;
5. A/B-тест;
6. контроль guardrail-метрик;
7. решение о rollout или rollback.

Нельзя заменять product outcome удобным proxy без проверки их связи. Например, рост CTR может быть вызван изменением позиции, но не улучшением релевантности.

## 7. Мониторинг после релиза

Минимальный мониторинг:

- data drift и изменение распределений;
- target drift или падение качества на delayed labels;
- train-serving skew;
- latency, error rate, throughput;
- стоимость inference;
- доля fallback и empty responses;
- quality по сегментам;
- safety и policy violations.

Нужно заранее определить:

- пороги алертов;
- кто реагирует;
- как отключается модель;
- как происходит rollback;
- когда запускается retraining;
- как отличить реальную деградацию от сезонности.

## 8. Компромиссы, которые стоит проговаривать

- качество ↔ latency;
- качество ↔ стоимость;
- coverage ↔ precision;
- персонализация ↔ cold start и privacy;
- свежесть данных ↔ стабильность;
- сложность модели ↔ explainability и поддержка;
- exploration ↔ краткосрочная метрика;
- автоматизация оценки ↔ риск незамеченной ошибки.

## 9. Как завершать ответ

В конце собрать решение в 4–5 предложений:

> Я начинаю с конкретной продуктовой цели и primary metric. Для первого релиза использую простой baseline и версионируемый data pipeline, разделяя offline и online контуры. Качество проверяю на временном held-out наборе и затем через A/B-тест, одновременно контролируя latency, cost и safety. После запуска мониторю drift, качество по срезам и train-serving consistency; при деградации включаю fallback или rollback.

## 10. Типичные ошибки

- **Начать с модели:** не определена пользовательская ценность.
- **Не назвать target:** непонятно, чему обучаемся.
- **Одна метрика:** скрываются latency, cost, safety и сегментные регрессии.
- **Нет baseline:** невозможно оценить пользу сложности.
- **Нет offline/online разделения:** путаются качество модели и эффект продукта.
- **Нет data leakage discussion:** offline score может быть ложным.
- **Нет fallback и rollback:** система не выглядит production-ready.
- **Нет iteration loop:** ответ заканчивается на deployment.

## 11. Практика

Для каждого кейса за 15 минут заполнить:

```text
Пользователь и цель:
Primary metric:
Guardrails:
Target:
Источники данных:
Baseline:
Модель:
Offline evaluation:
Online experiment:
Serving architecture:
Monitoring:
Главные риски:
Следующая итерация:
```

### Первые кейсы

- ranking в поиске;
- рекомендации товаров;
- fraud detection;
- прогноз спроса;
- оценка качества ответа LLM;
- генерация изображений с human preference;
- агент, который выполняет действия через API.

## Связано

- [[Литература/Книги/Machine Learning System Design — Babushkin и Kravchenko|Карта книги Machine Learning System Design]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Продукты/Внутренний контур оценки качества GenAI в Яндексе|Quality loop GenAI в Яндексе]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Product signal — offline eval, логи и пользовательская ценность|Product signal — offline eval и пользовательская ценность]]
- [[А-Б тесты/Методология/Иерархия метрик — Целевая, Guardrail и Proxy|Иерархия метрик]]
- [[Big Data/Big Data — Spark, YARN, Kafka, HDFS, Delta Lake|Big Data-контур]]

## Источник

- [Machine Learning System Design — Valerii Babushkin, Arseny Kravchenko](https://www.manning.com/books/machine-learning-system-design)
- Локальная версия: `C:\Users\sharn\Downloads\Machine_Learning_System_Design_v12_MEAP (1).pdf`.

---
[[Interview Reviews/Подготовка/ML System Design/🗺️ Индекс|Назад к индексу ML System Design]]
