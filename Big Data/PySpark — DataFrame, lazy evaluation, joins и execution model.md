---
tags: [big-data, spark, pyspark, dataframe, lazy-evaluation, broadcast-join, driver, executor]
тип: теория и практика
уровень: middle
сложность: средняя
статус: готово
готовность: 90
создано: 2026-05-28
источники:
  - "Вопросы пользователя по PySpark"
предпосылки:
  - "Big Data — Spark, YARN, Kafka, HDFS, Delta Lake"
связано: []
сравнить-с:
  - "Pandas"
---

# PySpark — DataFrame, lazy evaluation, joins и execution model

> [!abstract] Суть
> PySpark DataFrame — распределённая таблица с ленивым планом выполнения. Transformations строят DAG, actions запускают вычисление, Spark оптимизирует план, а driver координирует работу executors. Для join большой таблицы с маленькой часто используют broadcast join.

## Ответ

### 1. Transformations и actions

В Spark операции делятся на два типа.

**Transformations** строят новый DataFrame, но не запускают вычисление:
- `select`;
- `filter`;
- `withColumn`;
- `join`;
- `groupBy`;
- `repartition`.

**Actions** запускают выполнение DAG:
- `count`;
- `collect`;
- `show`;
- `write`;
- `take`.

До action Spark только накапливает logical plan.

### 2. Lazy evaluation

Lazy evaluation означает, что Spark откладывает вычисления до action. Это нужно, чтобы увидеть весь pipeline целиком и оптимизировать его:
- объединить фильтры;
- убрать ненужные колонки;
- переставить операции;
- выбрать стратегию join;
- протолкнуть фильтр к источнику данных.

Пример:

```python
df2 = df.filter("country = 'RU'").select("user_id")
# пока ничего не посчитано
df2.count()
# action запускает job
```

### 3. PySpark DataFrame vs NumPy / Pandas

NumPy array:
- локальный объект в памяти одного процесса;
- eager execution;
- быстрые vectorized операции в C;
- нет встроенного распределённого execution.

PySpark DataFrame:
- распределённые partitions на executors;
- schema и logical plan;
- lazy execution;
- подходит для данных больше памяти одной машины;
- overhead выше, поэтому на маленьких данных может проигрывать Pandas/NumPy.

### 4. Broadcast join

Если есть большая таблица и маленький справочник, маленькую таблицу можно отправить на все executors:

```python
from pyspark.sql.functions import broadcast

result = events.join(broadcast(users_dim), on="user_id", how="left")
```

Плюс: не нужно shuffle большой таблицы. Каждый executor join-ит свою partition с локальной копией маленькой таблицы.

Риски:
- маленькая таблица должна помещаться в память executors;
- если справочник неожиданно вырос, можно получить OOM;
- skew ключей всё равно может создавать дисбаланс.

### 5. Компоненты Spark-приложения

**Driver**:
- запускает `SparkSession`;
- строит logical/physical plan;
- отправляет tasks executors;
- собирает metadata и результаты actions.

**Executors**:
- выполняют tasks;
- хранят partitions/cache;
- читают/пишут данные;
- возвращают результаты driver.

**Cluster manager**:
- YARN, Kubernetes, Standalone;
- выделяет ресурсы.

### 6. Где исполняется Python-код

PySpark — Python API поверх JVM Spark. Driver Python-процесс общается с JVM. На executors Python UDF выполняется в Python worker processes, а встроенные Spark SQL functions выполняются в JVM и оптимизируются Catalyst.

Почему Python UDF медленнее:
- serialization между JVM и Python;
- хуже оптимизация Catalyst;
- overhead на передачу данных;
- vectorized Pandas UDF часто быстрее обычных UDF, но всё равно требует аккуратности.

## Формула / Схема

**Spark execution:**

```text
DataFrame transformations
-> logical plan
-> optimized logical plan
-> physical plan
-> DAG stages/tasks
-> executors
```

**Broadcast join:**

```text
small table -> broadcast to executors
big table partitions stay distributed
local join on each executor
```

## Короткий пример

Если таблица событий 10 TB, а справочник стран 200 KB, shuffle всей таблицы событий ради join бессмысленен. Broadcast справочника дешевле и быстрее.

## Типичные ошибки

- **Вызывать `collect()` на большой таблице:** данные едут на driver и могут убить память.
- **Использовать Python UDF вместо built-in функций:** теряется оптимизация Spark SQL.
- **Не понимать lazy evaluation:** замеры времени вокруг transformations могут ничего не измерять.
- **Broadcast огромной таблицы:** вместо ускорения получите OOM.

## Каверзные вопросы

> [!question] Почему `df.filter(...).select(...).count()` может прочитать меньше данных, чем кажется?
> Catalyst может протолкнуть filter к источнику и прочитать только нужные колонки, если формат данных поддерживает column pruning/predicate pushdown.

> [!question] Что делает driver, а что executor?
> Driver планирует и координирует job, executors выполняют tasks на partitions.

## Проверка себя

- Чем transformation отличается от action?
- Почему PySpark DataFrame не равен NumPy array?
- Как выполнить join большой и маленькой таблиц?
- Почему Python UDF может быть bottleneck?

## Предпосылки

- [[Big Data/Big Data — Spark, YARN, Kafka, HDFS, Delta Lake]]

## Связано

- [[Вопросы для МОК-Собеседований/Interview Questions/Pandas и PySpark]]
- [[Big Data/PySpark — практический пайплайн признаков и оптимизация под ML|Практический PySpark для ML-пайплайна]]

## Источники

- Вопросы пользователя по PySpark

---
[[🗺️ Индекс|Назад к разделу]]
