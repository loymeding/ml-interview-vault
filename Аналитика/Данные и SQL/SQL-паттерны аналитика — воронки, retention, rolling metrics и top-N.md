---
tags: [аналитика, sql, воронка, retention, оконные-функции, rolling, top-n]
тип: практика
уровень: middle
сложность: высокая
статус: готово
готовность: 95
создано: 2026-08-25
источники:
  - "SQL/🏠 Главная"
предпосылки:
  - "Событийная модель и grain данных — как не посчитать метрику неверно"
связано:
  - "Воронка и конверсия — от события до диагностики потерь"
  - "Retention, churn и когортный анализ"
  - "Качество данных — дубликаты, пропуски, late events, часовые пояса и join explosion"
сравнить-с: []
---

# SQL-паттерны аналитика — воронки, retention, rolling metrics и top-N

> [!abstract] Суть
> На аналитическом SQL-интервью проверяют не количество выученных функций, а способность зафиксировать grain, корректно обработать время, сохранить пользователей без событий и объяснить, почему запрос считает именно нужную бизнес-метрику.

## Как начинать решение вслух

Перед кодом полезно сказать:

1. «Одна строка исходной таблицы — ...»
2. «Одна строка результата должна быть — ...»
3. «Окно считаю полуоткрытым `[start, end)`».
4. «Нужно ли сохранить пользователей без событий?»
5. «Как дедуплицируются события и какой timestamp бизнесовый?»

Эти вопросы показывают зрелость сильнее, чем мгновенно написанный `SELECT`.

Ниже PostgreSQL-синтаксис; в MySQL отличаются interval/date functions и некоторые детали.

## 1. Conditional aggregation

Из событий получить одну строку на пользователя:

```sql
SELECT
    user_id,
    COUNT(*) FILTER (WHERE event_name = 'search') AS searches,
    COUNT(*) FILTER (WHERE event_name = 'click') AS clicks,
    COUNT(*) FILTER (WHERE event_name = 'purchase') AS purchases,
    SUM(amount) FILTER (WHERE event_name = 'purchase') AS revenue
FROM events
WHERE event_time >= TIMESTAMP '2026-08-01'
  AND event_time <  TIMESTAMP '2026-09-01'
GROUP BY user_id;
```

Переносимый вариант:

```sql
SUM(CASE WHEN event_name = 'purchase' THEN 1 ELSE 0 END)
```

Почему `ELSE 0` важен для суммы: без подходящих строк `SUM(CASE ... THEN 1 END)` может вернуть NULL.

## 2. Сохранить нулевых пользователей

Если нужны признаки для **каждого** пользователя, начинать надо с population:

```sql
WITH population AS (
    SELECT user_id
    FROM users
    WHERE type = 'person'
), agg AS (
    SELECT user_id, COUNT(*) AS tx_cnt
    FROM transactions
    WHERE dt >= TIMESTAMP '2025-09-01 23:59:59'
      AND dt <= TIMESTAMP '2025-09-30 23:59:59'
    GROUP BY user_id
)
SELECT
    p.user_id,
    COALESCE(a.tx_cnt, 0) AS tx_cnt
FROM population p
LEFT JOIN agg a USING (user_id);
```

Если поместить условие по правой таблице в `WHERE` после LEFT JOIN, нулевые пользователи исчезнут:

```sql
-- Ошибка: фактически INNER JOIN
FROM users u
LEFT JOIN events e ON e.user_id = u.user_id
WHERE e.event_time >= ...
```

Условие должно быть внутри `ON` или в предварительном CTE.

## 3. Воронка по пользователю

Сначала найдём первое время каждого шага:

```sql
WITH steps AS (
    SELECT
        user_id,
        MIN(event_time) FILTER (WHERE event_name = 'view') AS view_at,
        MIN(event_time) FILTER (WHERE event_name = 'cart') AS cart_at,
        MIN(event_time) FILTER (WHERE event_name = 'purchase') AS purchase_at
    FROM events
    GROUP BY user_id
)
SELECT
    COUNT(*) FILTER (WHERE view_at IS NOT NULL) AS viewed,
    COUNT(*) FILTER (
        WHERE cart_at > view_at
          AND cart_at <= view_at + INTERVAL '1 day'
    ) AS carted,
    COUNT(*) FILTER (
        WHERE purchase_at > cart_at
          AND purchase_at <= view_at + INTERVAL '1 day'
    ) AS purchased
FROM steps;
```

Нюанс: `MIN(purchase)` вообще может быть раньше выбранного `cart`, если пользователь покупал раньше. Для сложных повторяющихся путей нужен event sequence, correlated subquery/LATERAL или state-machine логика по session/order_id.

## 4. Retention

```sql
WITH cohorts AS (
    SELECT
        user_id,
        MIN(event_time::date) AS cohort_date
    FROM events
    WHERE event_name = 'signup'
    GROUP BY user_id
), activity AS (
    SELECT DISTINCT user_id, event_time::date AS activity_date
    FROM events
    WHERE event_name = 'meaningful_action'
), retention AS (
    SELECT
        c.cohort_date,
        a.activity_date - c.cohort_date AS day_n,
        COUNT(DISTINCT c.user_id) AS retained
    FROM cohorts c
    JOIN activity a
      ON a.user_id = c.user_id
     AND a.activity_date >= c.cohort_date
    GROUP BY 1, 2
), sizes AS (
    SELECT cohort_date, COUNT(*) AS cohort_size
    FROM cohorts
    GROUP BY cohort_date
)
SELECT
    r.cohort_date,
    r.day_n,
    r.retained,
    s.cohort_size,
    r.retained::numeric / s.cohort_size AS retention
FROM retention r
JOIN sizes s USING (cohort_date)
ORDER BY 1, 2;
```

Здесь exact-day retention. Для bracket нужно сгруппировать `day_n` по интервалам, для rolling — проверить активность на Dn или позже. Недозрелые когорты следует фильтровать относительно максимальной полной даты данных.

## 5. Rolling metrics: ROWS vs RANGE

```sql
SELECT
    dt,
    revenue,
    SUM(revenue) OVER (
        ORDER BY dt
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS revenue_7_rows
FROM daily_metrics;
```

Это последние семь **строк**, а не обязательно семь календарных дней. Если даты пропущены, окно растянется.

Надёжный путь — сначала создать календарь и заполнить отсутствующие дни нулями, затем использовать `ROWS`. Или применить time-based `RANGE`, если СУБД корректно поддерживает нужный interval frame.

```sql
SELECT
    dt,
    AVG(revenue) OVER (
        ORDER BY dt
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS ma7
FROM complete_daily_calendar;
```

## 6. Top-N per group

Три самых высокооплачиваемых **уровня зарплаты** отдела:

```sql
WITH ranked AS (
    SELECT
        e.*,
        DENSE_RANK() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employee e
)
SELECT *
FROM ranked
WHERE salary_rank <= 3;
```

Выбор функции:

- `ROW_NUMBER()` — ровно N строк, ties разрываются;
- `RANK()` — места с пропусками: 1, 1, 3;
- `DENSE_RANK()` — уникальные уровни без пропусков: 1, 1, 2.

## 7. First/last event и dedup

Последнее состояние заказа:

```sql
WITH ranked AS (
    SELECT
        s.*,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY status_time DESC, ingested_at DESC
        ) AS rn
    FROM order_status_history s
)
SELECT *
FROM ranked
WHERE rn = 1;
```

Нужен deterministic tie-breaker. Иначе при равном `status_time` результат может быть нестабилен.

## 8. LAG/LEAD для последовательностей

Найти интервал между событиями:

```sql
SELECT
    user_id,
    event_time,
    event_name,
    LAG(event_time) OVER (
        PARTITION BY user_id ORDER BY event_time
    ) AS prev_time,
    LEAD(event_name) OVER (
        PARTITION BY user_id ORDER BY event_time
    ) AS next_event
FROM events;
```

`PARTITION BY` отвечает «для кого отдельная последовательность», `ORDER BY` — «в каком порядке». Без `ORDER BY` понятия следующего события нет.

## 9. Anti-join

Пользователи без покупок:

```sql
SELECT u.user_id
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.user_id
      AND o.status = 'completed'
);
```

`NOT EXISTS` безопаснее `NOT IN`, если подзапрос может вернуть NULL: `x NOT IN (1, NULL)` становится UNKNOWN.

## 10. Процент от группы

```sql
WITH category AS (
    SELECT category_id, SUM(revenue) AS revenue
    FROM orders
    GROUP BY category_id
)
SELECT
    category_id,
    revenue,
    revenue / SUM(revenue) OVER () AS revenue_share
FROM category;
```

Сначала агрегируем до категории, затем window считает total по уже агрегированным строкам.

## 11. Метрики классификации в SQL

```sql
SELECT
    threshold,
    SUM(CASE WHEN score >= threshold AND label = 1 THEN 1 ELSE 0 END) AS tp,
    SUM(CASE WHEN score >= threshold AND label = 0 THEN 1 ELSE 0 END) AS fp,
    SUM(CASE WHEN score <  threshold AND label = 0 THEN 1 ELSE 0 END) AS tn,
    SUM(CASE WHEN score <  threshold AND label = 1 THEN 1 ELSE 0 END) AS fn
FROM predictions
CROSS JOIN thresholds
GROUP BY threshold;
```

После этого precision, recall и expected cost считаются с `NULLIF(denominator, 0)`.

## 12. Признаки за окно без leakage

```sql
SELECT
    p.user_id,
    COUNT(t.transaction_id) AS tx_cnt_30d,
    COALESCE(SUM(t.amt), 0) AS sum_30d
FROM prediction_points p
LEFT JOIN transactions t
  ON t.user_id = p.user_id
 AND t.dt >= p.t0 - INTERVAL '30 days'
 AND t.dt <  p.t0
GROUP BY p.user_id, p.t0;
```

Здесь одна строка результата на `(user_id, t0)`. События в `t0` не включены: предполагается, что prediction делается непосредственно перед ним.

## Как объяснять сложность и производительность

До микротюнинга проверьте:

- фильтруем ли partitions по дате;
- есть ли индекс/partition key на join и filter columns;
- не читаем ли одну таблицу многократно;
- можно ли предварительно агрегировать;
- не создаёт ли `COUNT(DISTINCT)` огромный shuffle;
- можно ли заменить JOIN на EXISTS;
- соответствуют ли типы ключей;
- что показывает `EXPLAIN`.

Но правильность важнее «быстрого» неверного запроса.

## Как ответить на интервью

> [!question] Как решить SQL-задачу, если условие неоднозначно?
> Сначала проговорю assumptions: grain таблиц и результата, бизнес-статус, окно и поведение NULL/ties. Затем построю решение CTE-этапами, чтобы каждый CTE имел понятный grain. В конце проверю пустого пользователя, дубликаты, равные timestamps и границы окна.

> [!question] WHERE и HAVING?
> WHERE фильтрует строки до группировки, HAVING — группы после агрегатов. Если условие можно применить раньше без изменения смысла, WHERE обычно уменьшит объём. Условие по правой таблице после LEFT JOIN в WHERE может удалить NULL-строки и превратить его в INNER JOIN.

> [!question] Когда окно лучше GROUP BY?
> GROUP BY схлопывает строки до группы. Window сохраняет детали и добавляет контекст: rank, running total, previous event, долю от total. Часто сначала нужен GROUP BY до бизнес-grain, затем window поверх агрегата.

## Типичные ошибки

- Не фиксировать grain CTE и результата.
- Использовать `BETWEEN` для соседних временных периодов и дважды включать границу.
- Считать последние 7 строк последними 7 днями.
- Забывать alias подзапроса в MySQL.
- Писать `DENSE_RANK(salary)` вместо `DENSE_RANK()`.
- Удалять нулевых пользователей условием в WHERE после LEFT JOIN.
- Использовать `NOT IN` с NULL.
- Не задавать tie-breaker в `ROW_NUMBER()`.

## Проверка себя

- Как сохранить пользователей без транзакций?
- В чём разница между `ROWS BETWEEN 6 PRECEDING` и календарными семью днями?
- Как посчитать три уникальных уровня salary, сохранив ties?
- Почему шаги сложной воронки нельзя всегда находить независимыми `MIN`?

## Связано

- [[SQL/🏠 Главная|Раздел SQL]]
- [[SQL/Практика/SQL — практические задачи — агрегации, окна и JOIN|Практические задачи SQL]]
- [[Воронка и конверсия — от события до диагностики потерь]]
- [[Retention, churn и когортный анализ]]

---
[[Аналитика/🗺️ Индекс|Назад к индексу аналитики]]
