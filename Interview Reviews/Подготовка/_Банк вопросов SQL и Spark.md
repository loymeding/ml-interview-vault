---
tags: [interview-prep, question-bank, sql, spark, databases]
type: topic-question-bank
topic: SQL и Spark
status: active
created: 2026-07-13
cssclasses: [wide-page]
---

# 🗄️ Банк вопросов — SQL, базы данных и Spark

> [!success] Развёрнутые ответы
> [[Interview Reviews/Подготовка/SQL и Spark — развёрнутые ответы|Открыть карточку с готовыми объяснениями]]: ответы сгруппированы по темам ниже и покрывают все вопросы банка.

> [!tip] Фокус
> Здесь вопросы уровня “часто спрашивают на DS/ML/Data Engineer-собесах”: индексы, JOIN, оконные функции, транзакции, ORM-ловушки, Spark execution model, shuffle, partitioning и оптимизация PySpark. Практический разбор признаков для CVM и uplift: [[Big Data/PySpark — практический пайплайн признаков и оптимизация под ML|открыть карточку]].

---

## 🧭 Карта блоков

| Блок | Что проверяют | Приоритет |
|---|---|---|
| SQL-запросы | JOIN, оконные функции, агрегации, `NULL` | 🔥 высокий |
| Индексы и оптимизация | B-tree, composite index, `EXPLAIN` | 🔥 высокий |
| Транзакции | isolation levels, locks, ACID | 🟡 средний |
| PostgreSQL / ORM | JSONB, lazy loading, N+1 | 🟡 средний |
| NoSQL | BASE, CAP, денормализация | 🟡 средний |
| Spark / PySpark | lazy evaluation, transformations/actions, shuffle | 🔥 высокий |
| Оптимизация Spark | broadcast join, skew, caching, partitioning | 🔥 высокий |

---

## 1. SQL: JOIN, агрегации и оконные функции

- [ ] Чем отличаются `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN` и `CROSS JOIN`?
- [ ] Чем `WHERE` отличается от `HAVING`?
- [ ] Что такое оконная функция?
- [ ] Приведи пример использования `ROW_NUMBER()`.
- [ ] Чем отличаются `ROW_NUMBER()`, `RANK()` и `DENSE_RANK()`?
- [ ] Что делают `PARTITION BY` и `ORDER BY` внутри оконной функции?
- [ ] Как посчитать топ-N записей в каждой группе?
- [ ] Как удалить дубликаты через оконную функцию?
- [ ] Чем `UNION` отличается от `UNION ALL`?
- [ ] Как работает `NULL` в SQL и почему `column = NULL` не работает?
- [ ] Чем `COUNT(*)` отличается от `COUNT(column)`?
- [ ] Как найти пользователей, у которых нет заказов?
- [ ] Как посчитать rolling average или cumulative sum в SQL?

---

## 2. Индексы и оптимизация запросов

- [ ] Как B-tree индекс ускоряет запросы с оператором `BETWEEN`?
- [ ] Какие запросы хорошо ускоряются B-tree индексом?
- [ ] Когда индекс может не использоваться даже если он есть?
- [ ] Что такое composite index?
- [ ] Что такое leftmost prefix rule в составном индексе?
- [ ] Чем covering index отличается от обычного индекса?
- [ ] Почему индекс ускоряет чтение, но может замедлять запись?
- [ ] Что такое селективность индекса?
- [ ] Как читать `EXPLAIN` / `EXPLAIN ANALYZE` на базовом уровне?
- [ ] Что такое full table scan и всегда ли это плохо?
- [ ] Почему функция над колонкой в `WHERE` может мешать использованию индекса?

---

## 3. Транзакции и уровни изоляции

- [ ] Что такое ACID?
- [ ] Опишите различие уровней изоляции `Read Committed` и `Serializable`.
- [ ] Какие аномалии транзакций бывают: dirty read, non-repeatable read, phantom read?
- [ ] Что такое lock и deadlock?
- [ ] Чем optimistic locking отличается от pessimistic locking?
- [ ] Когда транзакция может стать узким местом производительности?
- [ ] Какие гарантии ACID дает Delta Lake поверх S3?

---

## 4. PostgreSQL, JSONB и ORM

- [ ] Какие преимущества `JSONB` перед `JSON` в PostgreSQL?
- [ ] Когда хранить данные в `JSONB` оправданно, а когда лучше нормализовать схему?
- [ ] Что такое lazy loading в ORM и когда оно вредно?
- [ ] Что такое проблема N+1 запросов?
- [ ] Как eager loading помогает бороться с N+1?
- [ ] Что такое миграции схемы БД и почему их важно версионировать?

---

## 5. NoSQL и распределенные хранилища

- [ ] Почему BASE-подход часто выбирают для NoSQL-систем?
- [ ] Чем BASE отличается от ACID?
- [ ] Что такое eventual consistency?
- [ ] Что такое CAP theorem на практическом уровне?
- [ ] Когда NoSQL лучше реляционной БД?
- [ ] Какие компромиссы появляются при денормализации данных?

---

## 6. Spark и PySpark: базовая модель вычислений

- [ ] Чем DataFrame в Spark отличается от RDD с точки зрения оптимизации?
- [ ] Два основных типа операций в PySpark: transformations и actions.
- [ ] Что такое lazy evaluation в контексте Spark?
- [ ] Зачем Spark строит logical plan и physical plan?
- [ ] DataFrame в PySpark vs NumPy: в чем принципиальная разница?
- [ ] Компоненты Spark-приложения: driver, executors, cluster manager.
- [ ] Где исполняется Python-код в PySpark?
- [ ] Что такое task, stage и job в Spark UI?
- [ ] Почему PySpark UDF может быть медленнее встроенных функций Spark SQL?
- [ ] Чем Pandas UDF отличается от обычного Python UDF?

---

## 7. Spark: JOIN, shuffle и оптимизация

- [ ] Как выполнить JOIN большой таблицы с маленькой в Spark?
- [ ] Что такое broadcast join?
- [ ] Что такое shuffle и почему он дорогой?
- [ ] Что такое data skew и как он ломает Spark jobs?
- [ ] Как бороться со skew при JOIN?
- [ ] Чем `repartition` отличается от `coalesce`?
- [ ] Зачем использовать partitioning?
- [ ] Зачем использовать bucketing, если уже есть partitioning?
- [ ] Когда стоит применять `cache()` / `persist()`?
- [ ] Почему `cache()` не всегда ускоряет пайплайн?
- [ ] Как выбрать количество partitions?
- [ ] Что такое Adaptive Query Execution?

---

## 8. Big Data инфраструктура

- [ ] Опишите архитектуру YARN: `ResourceManager` и `NodeManager`.
- [ ] Что происходит при потере одного из `DataNode` в HDFS?
- [ ] Как Kafka обеспечивает at-least-once доставку сообщений?
- [ ] Чем at-least-once отличается от exactly-once и at-most-once?
- [ ] Что такое offset в Kafka?
- [ ] Почему consumer group важна для масштабирования чтения из Kafka?

---

## 9. Практические SQL/Spark-сценарии

- [ ] Как найти вторую по величине зарплату в таблице?
- [ ] Как посчитать retention по когортам?
- [ ] Как найти дубликаты по нескольким колонкам?
- [ ] Как обновить витрину инкрементально, а не пересчитывать все данные?
- [ ] Как проверить, что после JOIN не размножились строки?
- [ ] Как диагностировать медленный Spark job?
- [ ] Что смотреть в Spark UI при долгом выполнении задачи?
- [ ] Как понять, что Spark job уперся в память?

---

## ✅ Следующий шаг

> [!success] Как превращать в ответы
> Для этой страницы хорошо подойдут отдельные карточки-ответы по блокам: SQL JOIN, оконные функции, индексы, транзакции, Spark execution model, Spark joins/shuffle, PySpark UDF и оптимизация.
