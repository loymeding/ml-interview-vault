---
tags: [аналитика, данные, события, grain, event-model, sql, идентификаторы]
тип: теория
уровень: middle
сложность: высокая
статус: готово
готовность: 95
создано: 2026-08-25
источники:
  - "Designing Data-Intensive Applications — Martin Kleppmann"
предпосылки:
  - "Воронка и конверсия — от события до диагностики потерь"
связано:
  - "Качество данных — дубликаты, пропуски, late events, часовые пояса и join explosion"
  - "SQL-паттерны аналитика — воронки, retention, rolling metrics и top-N"
сравнить-с: []
---

# Событийная модель и grain данных — как не посчитать метрику неверно

> [!abstract] Суть
> Перед SQL нужно одним предложением назвать, чему соответствует одна строка каждой таблицы. Это grain. Большинство тихих аналитических ошибок возникает, когда таблицы с разным grain соединяют или агрегируют без явного перехода между уровнями.

## Что такое grain

Grain — минимальный объект, который представляет одна строка.

Примеры:

- `events`: одно клиентское событие;
- `orders`: один заказ;
- `order_items`: одна позиция товара в заказе;
- `daily_users`: один пользователь в один календарный день;
- `experiments`: одно назначение пользователя в эксперимент;
- `search_impressions`: один показ одного объекта в одном запросе.

Фраза «таблица с заказами» недостаточна. В одной системе строка — заказ, в другой — товар заказа, в третьей — изменение статуса. `COUNT(*)` означает совершенно разные вещи.

## Почему grain важнее синтаксиса

Есть `orders(order_id, user_id, total)` и `order_items(order_id, product_id)`. Один заказ содержит три товара. После JOIN строк заказа станет три.

```sql
SELECT SUM(o.total)
FROM orders o
JOIN order_items i USING (order_id);
```

Этот запрос посчитает `total` три раза. SQL синтаксически корректен, число выглядит правдоподобно, но grain результата — товар заказа, а `total` относится к заказу.

Исправления зависят от цели:

```sql
-- Если нужен общий оборот заказов, JOIN вообще не нужен.
SELECT SUM(total) FROM orders;

-- Если нужен оборот заказов, содержащих нужную категорию:
SELECT SUM(o.total)
FROM orders o
WHERE EXISTS (
    SELECT 1
    FROM order_items i
    WHERE i.order_id = o.order_id
      AND i.category = 'books'
);
```

`SUM(DISTINCT total)` не является общим решением: два разных заказа могут иметь одинаковую сумму.

## Событие — это бизнес-факт, а не просто строка лога

Хорошее событие отвечает:

- кто сделал действие;
- что сделал;
- над каким объектом;
- когда это произошло на клиенте и когда принято сервером;
- в каком контексте: session, request, screen, experiment;
- какой уникальный `event_id` позволяет дедуплицировать;
- какая версия схемы и приложения;
- было ли действие успешно.

Пример:

```text
event_id
event_name = "payment_completed"
event_time
ingested_at
user_id / anonymous_id
session_id
order_id
amount / currency
platform / app_version
experiment_id / variant
schema_version
```

Чем ближе событие к важному outcome, тем полезнее подтверждать его сервером. Клиент может закрыться до отправки `purchase_success`, повторить запрос или сфальсифицировать событие.

## Event time и processing time

**Event time** — когда действие произошло в мире пользователя.

**Processing/ingestion time** — когда событие попало в систему.

Пользователь совершил заказ в 23:58, телефон был offline, событие пришло в 00:10. По `ingested_at` заказ попадёт в следующий день. Для продуктовой метрики обычно нужен event time, а для мониторинга pipeline — ingestion delay между ними.

## Идентификаторы и identity stitching

До входа пользователь известен как `anonymous_id`, после — как `user_id`. Если просто считать `COUNT(DISTINCT user_id)`, анонимная часть пути исчезнет. Если объединять устройства слишком агрессивно, разные люди одного household станут одним.

Нужно определить:

- что является аналитическим объектом: человек, аккаунт, устройство, cookie;
- можно ли стабильно связать anonymous → authorized;
- что делать со shared device и несколькими аккаунтами;
- не использует ли stitching информацию из будущего.

Для эксперимента bucketing key и аналитический identity должны быть согласованы. Иначе один человек может оказаться в обоих вариантах.

## Sessionization

Сессия часто не хранится готовой и строится правилом: новая сессия начинается после 30 минут бездействия.

С помощью `LAG`:

```sql
WITH ordered AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (
            PARTITION BY user_id ORDER BY event_time
        ) AS prev_time
    FROM events
), marked AS (
    SELECT *,
        CASE
            WHEN prev_time IS NULL
              OR event_time > prev_time + INTERVAL '30 minutes'
            THEN 1 ELSE 0
        END AS new_session
    FROM ordered
)
SELECT *,
    SUM(new_session) OVER (
        PARTITION BY user_id ORDER BY event_time
    ) AS session_num
FROM marked;
```

30 минут — convention, а не истина. Для просмотра видео, редактора документов и такси естественный разрыв различается.

## Факт и измерение

В аналитическом хранилище часто разделяют:

- **fact tables:** события, заказы, платежи — много строк, измеримые факты;
- **dimension tables:** пользователь, товар, продавец, календарь — контекст и атрибуты.

Но измерения меняются. Пользователь сегодня premium, вчера free. Если JOIN всегда берёт текущий статус, исторические метрики переписываются.

### Slowly Changing Dimensions

Для исторической корректности можно хранить интервалы действия атрибута:

```text
user_id | plan    | valid_from | valid_to
42      | free    | Jan 01     | Mar 10
42      | premium | Mar 10     | infinity
```

Событие соединяется с версией, действовавшей на момент event time.

## Snapshot и event log

**Event log** хранит изменения: заказ создан, оплачен, отменён.

**Snapshot** хранит состояние на момент: текущий статус заказа или дневной остаток.

Если в snapshot сейчас `cancelled`, из него нельзя восстановить, сколько заказов считалось активными вчера, если история не сохранена. Для временных метрик нужны event history или периодические snapshots.

## Дата метрики

У одного заказа есть:

- `created_at`;
- `paid_at`;
- `delivered_at`;
- `refunded_at`.

Заказы, revenue и возвраты могут относиться к разным датам. Поэтому перед запросом нужно определить бизнес-время. Иначе месячный отчёт будет меняться неясным образом.

## Eligibility и exposure

В эксперименте:

- **assigned:** пользователь назначен в вариант;
- **eligible:** соответствовал условиям;
- **exposed:** реально получил treatment;
- **engaged:** взаимодействовал.

Основной ITT-анализ обычно строится по назначенным eligible users. Анализ только engaged нарушает рандомизацию. Для диагностики полезно строить exposure funnel, но не подменять им causal estimate.

## Пример проектирования: поиск товаров

Вместо одного события `search` полезно иметь:

- один `search_request` на запрос: query, request_id, filters, latency;
- много `search_impression`: request_id, product_id, position, model_version, score, viewport flag;
- `click`: request_id, product_id, position, event_time;
- downstream `add_to_cart` и `purchase` с устойчивой атрибуцией;
- candidate set или хотя бы retrieval/ranker versions;
- experiment assignments.

Без `request_id` трудно связать клик с конкретной выдачей. Без позиции невозможно анализировать position bias. Без model_version нельзя объяснить деградацию.

## Как ответить на интервью

> [!question] Что спросить перед написанием SQL?
> Какой grain у каждой таблицы, какие ключи уникальны, какие связи one-to-one/one-to-many/many-to-many, какой статус и время определяют бизнес-факт, как устроены дубли и late events. Затем зафиксирую grain результата — например, одна строка на пользователя за день.

> [!question] Почему `COUNT(DISTINCT user_id)` не всегда решает дубли?
> Он может скрыть размножение на уровне пользователей, но суммы и другие агрегаты останутся искажены. Кроме того, identity может быть неполной. Нужно исправить join/grain и дедупликацию по бизнес-ключу, а не маскировать проблему DISTINCT.

> [!question] Как логировать ranking-систему?
> Request с контекстом, полный показанный список с позициями и scores, model/feature versions, exposure viewport, действия пользователя и downstream outcomes, latency/errors, experiment assignment. Желательно хранить candidate provenance и propensity для counterfactual анализа.

## Типичные ошибки

- Не назвать grain до JOIN.
- Суммировать order-level поле после JOIN с items.
- Использовать `SUM(DISTINCT amount)` как дедупликацию.
- Считать клиентский success надёжным финансовым фактом.
- Группировать по ingestion date вместо event date без объяснения.
- Применять текущий user segment ко всей истории.
- Анализировать engaged users вместо assigned users.

## Проверка себя

- Какой grain у `order_items`, `orders` и результата monthly user features?
- Почему текущий тариф пользователя нельзя JOIN-ить к прошлогодним заказам?
- Чем assigned, exposed и engaged отличаются в эксперименте?
- Какие поля нужны, чтобы исследовать position bias?

## Связано

- [[Качество данных — дубликаты, пропуски, late events, часовые пояса и join explosion]]
- [[SQL-паттерны аналитика — воронки, retention, rolling metrics и top-N]]
- [[Смещения в поиске и рекомендациях — position bias, feedback loops и offline-online gap]]

---
[[Аналитика/🗺️ Индекс|Назад к индексу аналитики]]

