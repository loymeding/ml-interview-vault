---
tags: [big-data, spark, pyspark, ml-system-design, feature-engineering, uplift, churn, personalization, performance]
тип: теория и практика
уровень: middle
сложность: высокая
статус: готово
готовность: 95
создано: 2026-09-03
источники:
  - "Apache Spark SQL, DataFrames and Datasets Guide"
  - "High Performance Spark — Holden Karau, Rachel Warren"
  - "Designing Machine Learning Systems — Chip Huyen"
предпосылки:
  - "PySpark — DataFrame, lazy evaluation, joins и execution model"
  - "Оконные функции SQL"
  - "Feature engineering и временная валидация"
связано:
  - "Big Data — Spark, YARN, Kafka, HDFS, Delta Lake"
  - "Как выбирать архитектуру нейросети — от задачи до production"
  - "Uplift-моделирование"
  - "Дизайн A-B-теста — гипотезы, метрики, выборка и решение"
сравнить-с:
  - "Pandas"
---

# PySpark на практике — признаки, оптимизация и ML-пайплайн

> [!abstract] Суть
> PySpark — это Python-интерфейс к распределённому движку Apache Spark. В задачах X5 он нужен не для того, чтобы «сделать Pandas на большем файле», а чтобы безопасно обработать историю покупок, промо и коммуникаций, которая не помещается на одной машине. Хороший PySpark-код учитывает распределение данных, стоимость shuffle, временную корректность признаков и то, где реально выполняется Python.
>
> Типичная цепочка для CVM (Customer Value Management, управления ценностью клиента):
>
> **сырые события → очистка и дедупликация → point-in-time признаки → target и treatment → временной split → датасет модели → рекомендации → A/B-тест.**

## 1. Контекст задачи X5

Представим таблицы:

- orders: заказ, пользователь, магазин, время, сумма и маржа;
- order_items: товары и категории в заказе;
- promotions: назначенные скидки и кэшбеки;
- communications: отправленные push, email и сообщения в приложении;
- users: регион, формат магазина и дата регистрации.

Нужно построить таблицу для uplift-модели:

| user_id | snapshot_dt | orders_30d | avg_check_90d | last_order_days | category_id | treatment | bought_7d | margin_7d |
|---|---|---:|---:|---:|---|---|---:|---:|

Одна строка означает:

> «Что мы знали о пользователе на дату snapshot_dt, какое воздействие он получил и что произошло в следующие 7 дней».

Главный риск — случайно использовать информацию из будущего. Например, посчитать orders_30d так, чтобы туда попал заказ после назначения промо. Такая утечка даст красивые offline-метрики и плохую работу в продакшене.

## 2. Ментальная модель Spark

### 2.1. Driver, executors и cluster manager

**Driver** — координирующая часть приложения:

- создаёт SparkSession;
- принимает код пользователя;
- строит план вычислений;
- разбивает работу на jobs, stages и tasks;
- отправляет задачи executors;
- получает небольшие результаты и метаданные.

**Executor** — процесс на рабочем узле:

- выполняет tasks;
- читает partitions;
- считает выражения;
- держит данные в памяти при cache;
- пишет результаты.

**Cluster manager** — система, которая выдаёт ресурсы: YARN, Kubernetes или Standalone.

Важно: driver не должен получать всю таблицу. Вызов collect на миллиардах строк пытается перенести данные в память driver и часто заканчивается ошибкой.

### 2.2. Partition, task, stage и job

- **Partition** — кусок распределённого набора данных.
- **Task** — работа одного executor над одной partition.
- **Stage** — группа tasks, которую можно выполнить без обмена данными между partitions.
- **Job** — вычисление, запущенное action.

Граница shuffle обычно создаёт новую stage. Например, groupBy по user_id должен собрать записи одного пользователя на одной partition, поэтому Spark перераспределяет данные по сети.

## 3. Lazy evaluation и план выполнения

Операции DataFrame делятся на два типа.

### Transformations

Они создают новый план, но не считают данные:

- select;
- filter;
- withColumn;
- join;
- groupBy;
- window;
- repartition.

### Actions

Они запускают вычисление:

- count;
- show;
- collect;
- take;
- write;
- foreach.

Пример:

~~~python
from pyspark.sql import functions as F

features = (
    orders
    .filter(F.col("status") == "completed")
    .select("user_id", "order_id", "order_dt", "amount")
)
# До action данные не прочитаны и не отфильтрованы.
features.write.mode("overwrite").parquet(output_path)
# write запускает job.
~~~

Ленивость позволяет Spark увидеть всю цепочку и оптимизировать её через Catalyst:

- убрать ненужные колонки;
- протолкнуть filter к источнику;
- выбрать стратегию join;
- объединить выражения;
- использовать column pruning и predicate pushdown.

Замер времени только вокруг transformations ничего не показывает:

~~~python
start = time.perf_counter()
result = df.filter(F.col("amount") > 0)
print(time.perf_counter() - start)  # почти только построение плана
result.count()                      # здесь начинается работа
~~~

## 4. DataFrame и встроенные функции

DataFrame — распределённая таблица со схемой и декларативным планом. Spark понимает выражения вроде col, when, sum и join и может оптимизировать их.

~~~python
from pyspark.sql import functions as F

clean = (
    orders
    .select(
        F.col("user_id").cast("long"),
        F.to_timestamp("order_dt").alias("order_ts"),
        F.col("amount").cast("double"),
        F.col("status"),
    )
    .filter(
        (F.col("status") == "completed")
        & F.col("user_id").isNotNull()
    )
)
~~~

Python UDF (пользовательская функция) менее предпочтительна:

~~~python
from pyspark.sql.types import StringType

@F.udf(returnType=StringType())
def normalize_category(value):
    return value.strip().lower() if value else None
~~~

Обычный Python UDF обычно медленнее встроенной функции, потому что:

- данные переходят из JVM Spark в Python worker;
- происходит сериализация;
- Catalyst хуже видит логику функции;
- нельзя полноценно протолкнуть выражение к источнику.

Вместо UDF:

~~~python
clean = clean.withColumn(
    "category_normalized",
    F.lower(F.trim(F.col("category_name")))
)
~~~

Если Python действительно нужен, можно рассмотреть Pandas UDF с Arrow. Он работает пакетами и уменьшает overhead, но требует контроля памяти и всё равно не всегда быстрее встроенных SQL-функций.

## 5. Фильтры, колонки и null

### Фильтрация

В PySpark для Column используются побитовые операторы:

- & для AND;
- | для OR;
- ~ для NOT.

Нельзя писать обычный Python and/or между Column-объектами.

~~~python
df.filter(
    (F.col("amount") > 100)
    & (F.col("status") == "completed")
)
~~~

### Null

Null — не то же самое, что пустая строка или ноль:

~~~python
df = df.withColumn(
    "amount_filled",
    F.coalesce(F.col("amount"), F.lit(0.0))
)
~~~

Но заменять все null нулём опасно: отсутствие суммы, отсутствие истории и настоящая сумма 0 имеют разный смысл. Для модели полезно создавать отдельный флаг:

~~~python
df = (
    df
    .withColumn("amount_is_null", F.col("amount").isNull().cast("int"))
    .withColumn("amount_filled", F.coalesce("amount", F.lit(0.0)))
)
~~~

## 6. Группировки и агрегации

Пример пользовательских признаков:

~~~python
user_agg = (
    orders
    .groupBy("user_id")
    .agg(
        F.countDistinct("order_id").alias("orders_cnt"),
        F.sum("amount").alias("revenue"),
        F.avg("amount").alias("avg_check"),
        F.max("order_ts").alias("last_order_ts"),
        F.countDistinct("category_id").alias("uniq_categories"),
    )
)
~~~

Агрегация обычно вызывает shuffle: записи с одним user_id должны оказаться вместе. Поэтому перед groupBy проверяем:

- достаточно ли данных;
- не создаём ли лишние колонки;
- не существует ли сильного skew по ключу;
- нужна ли предварительная фильтрация.

Если нужен только count, не стоит считать несколько дорогих статистик в разных проходах без необходимости. Часто выгоднее собрать их в одном agg.

## 7. Оконные функции

Оконная функция считает значение по связанному набору строк, не схлопывая их в одну строку.

~~~python
from pyspark.sql.window import Window

user_window = Window.partitionBy("user_id").orderBy("order_ts")

with_prev = orders.withColumn(
    "prev_order_ts",
    F.lag("order_ts").over(user_window)
)
~~~

### 7.1. Удаление дублей

Если события могут прийти повторно, оставляем последнюю версию:

~~~python
dedup_window = (
    Window
    .partitionBy("order_id")
    .orderBy(F.col("updated_at").desc())
)

orders_latest = (
    orders
    .withColumn("_rn", F.row_number().over(dedup_window))
    .filter(F.col("_rn") == 1)
    .drop("_rn")
)
~~~

Нельзя использовать dropDuplicates без понимания того, какую запись Spark оставит. Если важна последняя версия, задаём порядок явно.

### 7.2. Временные окна для признаков

В X5 часто нужно считать покупки в последние 30 дней относительно даты снимка. Для timestamp-окна:

~~~python
window_30d = (
    Window
    .partitionBy("user_id", "snapshot_dt")
    .orderBy(F.col("order_ts").cast("long"))
    .rangeBetween(-30 * 24 * 3600, -1)
)

features = events.withColumn(
    "orders_30d",
    F.count("order_id").over(window_30d)
)
~~~

На практике безопаснее сначала построить сетку user × snapshot, а затем сделать условный join:

~~~python
features = (
    snapshots.alias("s")
    .join(
        orders.alias("o"),
        on=(
            (F.col("s.user_id") == F.col("o.user_id"))
            & (F.col("o.order_ts") < F.col("s.snapshot_ts"))
            & (
                F.col("o.order_ts")
                >= F.col("s.snapshot_ts") - F.expr("INTERVAL 30 DAYS")
            )
        ),
        how="left",
    )
    .groupBy("s.user_id", "s.snapshot_ts")
    .agg(
        F.countDistinct("o.order_id").alias("orders_30d"),
        F.sum("o.amount").alias("revenue_30d"),
    )
)
~~~

Так условие «только прошлое» видно прямо в коде и проще проверяется.

### 7.3. rowsBetween и rangeBetween

- rowsBetween считает фиксированное число физических строк;
- rangeBetween считает диапазон значений orderBy.

Если события приходят с разными промежутками времени, rowsBetween(−30, −1) означает 30 предыдущих событий, а не 30 дней. Для временного окна это часто ошибка.

## 8. Дедупликация и качество событий

До feature engineering нужно определить:

- что является уникальным событием;
- могут ли события приходить повторно;
- какой статус считать завершённым;
- как обрабатывать отменённые заказы;
- учитывается ли возврат;
- какое время использовать — событие, обработку или оплату;
- как бороться с поздно пришедшими данными.

Пример бизнес-правила:

~~~text
один заказ = последняя запись по order_id
покупка = status = completed
выручка = сумма оплаченных позиций
возврат = отдельная корректировка маржи
~~~

Если эти правила не зафиксировать, две команды могут получить разные признаки из одной таблицы.

## 9. JOIN и стратегии выполнения

### 9.1. Обычный join

~~~python
result = orders.join(users, on="user_id", how="left")
~~~

Join может вызвать shuffle обеих таблиц. Проверяем:

- уникальность ключа в справочнике;
- типы ключей;
- дубли справа;
- долю unmatched;
- размер таблиц;
- skew по ключу.

### 9.2. Broadcast join

Если таблица маленькая, её можно разослать каждому executor:

~~~python
from pyspark.sql.functions import broadcast

result = orders.join(
    broadcast(users_dim),
    on="user_id",
    how="left",
)
~~~

Плюс — не нужно перераспределять большую таблицу. Риски:

- справочник должен помещаться в память каждого executor;
- размер может неожиданно вырасти;
- дубли ключей справа размножат строки;
- broadcast не спасает от логически неправильного join.

Перед broadcast желательно:

~~~python
users_small = (
    users
    .select("user_id", "region", "format")
    .dropDuplicates(["user_id"])
)
~~~

### 9.3. Sort-merge join

Для больших таблиц Spark часто сортирует и перераспределяет обе стороны по ключу, затем объединяет их. Это нормальная стратегия, но она требует сетевого обмена и места под shuffle.

Не нужно бездумно запрещать sort-merge join. Если обе таблицы большие, broadcast может быть хуже или невозможен.

### 9.4. Ошибка many-to-many

Если справа несколько строк на одного пользователя, join может увеличить число строк:

~~~text
orders: 1 строка на заказ
promotions: 5 строк на пользователя
join: один заказ превращается в 5 строк
~~~

Проверяем кардинальность:

~~~python
users.groupBy("user_id").count().filter(F.col("count") > 1).show()
~~~

Если нужен один актуальный статус, сначала выбираем его оконной функцией.

## 10. Data skew и salting

Skew — неравномерное распределение данных. Например, один технический user_id или один магазин содержит десятки миллионов событий. После shuffle одна task работает существенно дольше остальных.

Симптомы:

- большинство tasks завершились быстро, одна или несколько очень долго;
- большой разброс размера partitions;
- в Spark UI одна task имеет огромный shuffle read;
- растёт spill и GC time.

### Варианты решения

1. Отфильтровать ошибочные ключи.
2. Предагрегировать данные до join или groupBy.
3. Использовать broadcast, если одна сторона действительно маленькая.
4. Разделить heavy hitters отдельной логикой.
5. Использовать salting — добавить случайную соль к ключу и размножить маленькую сторону по солям.

Упрощённая схема salting:

~~~python
facts = facts.withColumn(
    "salt",
    (F.rand(seed=42) * 16).cast("int")
)

dim_salted = dim.crossJoin(
    spark.range(16).withColumnRenamed("id", "salt")
)

joined = facts.join(
    dim_salted,
    on=["user_id", "salt"],
    how="left",
)
~~~

Salting увеличивает объём данных и усложняет логику. Его используют только после подтверждения skew.

## 11. repartition и coalesce

### repartition

~~~python
df2 = df.repartition(200, "user_id")
~~~

- обычно вызывает полный shuffle;
- может увеличить или уменьшить число partitions;
- перераспределяет записи по ключу;
- полезен перед тяжёлой операцией, если данные плохо распределены;
- дорогой, потому что данные идут по сети.

### coalesce

~~~python
df2 = df.coalesce(20)
~~~

- обычно уменьшает число partitions без полного shuffle;
- полезен перед записью небольшого результата;
- не подходит для равномерного увеличения числа partitions;
- может создать очень крупные partitions, если уменьшить слишком сильно.

Формулировка для интервью:

> repartition перераспределяет данные и стоит shuffle; coalesce обычно только объединяет существующие partitions и дешевле, но может ухудшить баланс.

Нельзя считать, что больше partitions всегда быстрее. Слишком много маленьких tasks создаёт overhead планировщика, а слишком мало — длинные tasks и нехватку параллелизма.

## 12. Как выбрать число partitions

На него влияют:

- объём данных;
- число cores;
- память executor;
- сложность операции;
- skew;
- размер файлов;
- число запросов.

Часто начинают с настройки Spark и измеряют, а не используют магическое число. Полезно ориентироваться на partitions, которые достаточно велики для эффективной работы, но помещаются в память task.

После shuffle смотрим:

- средний и максимальный размер partition;
- spill;
- длительность tasks;
- число пустых partitions;
- shuffle read/write.

## 13. Чтение и запись данных

### Предпочтительные форматы

Parquet и ORC хранят колонки и поддерживают:

- column pruning;
- predicate pushdown;
- сжатие;
- типы данных;
- чтение только нужных полей.

CSV удобен для обмена, но хуже для больших ML-пайплайнов: нет надёжной схемы, больше parsing overhead и часто больше объём.

### Маленькие файлы

Если каждая маленькая задача пишет множество файлов, появляется small files problem:

- метаданных становится очень много;
- чтение запускает слишком много tasks;
- растёт scheduler overhead.

Перед записью можно уменьшить число partitions:

~~~python
features.coalesce(100).write.mode("overwrite").parquet(path)
~~~

Но не надо ставить coalesce(1) для большой таблицы: получится один узкий task и один файл, который может писать очень долго.

### Partitioning по колонкам

~~~python
features.write.partitionBy("snapshot_date", "region").parquet(path)
~~~

Partitioning по колонкам позволяет пропустить целые директории при фильтре. Но не стоит партиционировать по user_id: получится огромное количество мелких директорий.

Хорошие кандидаты:

- дата;
- крупный регион;
- тип данных;
- версия snapshot.

## 14. cache и persist

Cache нужен, если один и тот же DataFrame используется несколько раз:

~~~python
prepared = expensive_pipeline.cache()
prepared.count()  # материализация cache

metrics = prepared.groupBy("segment").count()
sample = prepared.sample(0.01)
~~~

Без action cache ещё не материализован.

Риски:

- cache может вытеснить полезные данные;
- память executor ограничена;
- хранение тяжёлой таблицы может быть дороже повторного вычисления;
- после использования нужно освобождать память:

~~~python
prepared.unpersist()
~~~

Используем persist с подходящим StorageLevel, если данных не хватает для полного хранения в памяти.

Не следует автоматически ставить cache после каждого шага. Сначала нужно увидеть повторное использование и профилировать время.

## 15. explain и Spark UI

### explain

~~~python
features.explain("formatted")
~~~

В плане смотрим:

- FileScan;
- фильтры и выбранные колонки;
- BroadcastHashJoin;
- SortMergeJoin;
- Exchange — границу shuffle;
- Window;
- HashAggregate;
- PythonUDF.

Если ожидали broadcast, а видим SortMergeJoin, проверяем размер, hint и конфигурацию.

### Spark UI

Полезные сигналы:

- сколько длится stage;
- shuffle read/write;
- spill memory/disk;
- размер partitions;
- GC time;
- failed/retried tasks;
- skew по отдельным tasks;
- input/output;
- число tasks.

Диагностику медленного job лучше формулировать от наблюдения:

~~~text
один task намного дольше остальных
→ проверяю skew и размер partition
→ смотрю shuffle read и ключи
→ затем выбираю broadcast, предагрегацию или salting
~~~

Увеличение памяти без понимания причины часто лишь маскирует проблему.

## 16. Полный пример: признаки для uplift

Ниже — упрощённая схема, которую можно обсуждать на собеседовании.

~~~python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# 1. Чистим заказы и оставляем только завершённые события.
orders_clean = (
    orders
    .select(
        "order_id",
        "user_id",
        F.to_timestamp("order_ts").alias("order_ts"),
        F.col("amount").cast("double").alias("amount"),
        F.col("margin").cast("double").alias("margin"),
        "status",
    )
    .filter(F.col("status") == "completed")
    .filter(F.col("user_id").isNotNull())
)

# 2. Убираем повторные версии заказа.
latest_window = (
    Window
    .partitionBy("order_id")
    .orderBy(F.col("updated_at").desc())
)

orders_clean = (
    orders_clean
    .withColumn("_rn", F.row_number().over(latest_window))
    .filter(F.col("_rn") == 1)
    .drop("_rn")
)

# 3. Строим user × snapshot, то есть момент принятия решения.
snapshots = (
    assignment
    .select(
        "user_id",
        F.to_timestamp("assignment_ts").alias("snapshot_ts"),
        "treatment",
    )
    .dropDuplicates(["user_id", "snapshot_ts"])
)

# 4. Присоединяем только прошлые заказы за 90 дней.
history = (
    snapshots.alias("s")
    .join(
        orders_clean.alias("o"),
        on=(
            (F.col("s.user_id") == F.col("o.user_id"))
            & (F.col("o.order_ts") < F.col("s.snapshot_ts"))
            & (
                F.col("o.order_ts")
                >= F.col("s.snapshot_ts") - F.expr("INTERVAL 90 DAYS")
            )
        ),
        how="left",
    )
)

# 5. Считаем признаки в окнах 30 и 90 дней.
features = (
    history
    .groupBy(
        F.col("s.user_id").alias("user_id"),
        F.col("s.snapshot_ts").alias("snapshot_ts"),
        F.col("s.treatment").alias("treatment"),
    )
    .agg(
        F.countDistinct(
            F.when(
                F.col("o.order_ts")
                >= F.col("s.snapshot_ts") - F.expr("INTERVAL 30 DAYS"),
                F.col("o.order_id"),
            )
        ).alias("orders_30d"),
        F.sum(
            F.when(
                F.col("o.order_ts")
                >= F.col("s.snapshot_ts") - F.expr("INTERVAL 30 DAYS"),
                F.col("o.amount"),
            )
        ).alias("revenue_30d"),
        F.sum("o.margin").alias("margin_90d"),
        F.max("o.order_ts").alias("last_order_ts"),
    )
    .withColumn(
        "days_since_last_order",
        F.datediff(
            F.to_date("snapshot_ts"),
            F.to_date("last_order_ts"),
        ),
    )
)

# 6. Target считаем только в будущем окне после snapshot.
future_orders = (
    snapshots.alias("s")
    .join(
        orders_clean.alias("o"),
        on=(
            (F.col("s.user_id") == F.col("o.user_id"))
            & (F.col("o.order_ts") > F.col("s.snapshot_ts"))
            & (
                F.col("o.order_ts")
                <= F.col("s.snapshot_ts") + F.expr("INTERVAL 7 DAYS")
            )
        ),
        how="left",
    )
    .groupBy("s.user_id", "s.snapshot_ts")
    .agg(
        F.max(F.lit(1)).alias("bought_7d"),
        F.sum("o.margin").alias("margin_7d"),
    )
)

dataset = (
    features
    .join(future_orders, ["user_id", "snapshot_ts"], "left")
    .fillna({"bought_7d": 0, "margin_7d": 0.0})
)
~~~

В production к этому добавляются:

- контроль дублей;
- проверка временных границ;
- качество treatment;
- versioned snapshot;
- сохранение схемы;
- тесты на небольших данных;
- контроль числа строк после каждого join.

## 17. Временная корректность и leakage

### 17.1. Point-in-time join

Point-in-time join — соединение, при котором к объекту присоединяются только данные, доступные на момент прогноза.

Нужно явно проверять:

~~~text
feature_event_time < prediction_time
target_event_time > prediction_time
~~~

Не стоит рассчитывать признаки на текущей полной таблице, если затем использовать их для старых snapshot: так будущие события «протекут» в прошлое.

### 17.2. Временной split

Для покупок обычно разумнее:

~~~text
train: январь–июнь
validation: июль
test: август
~~~

Случайный split может положить покупки одного пользователя из будущего в train и прошлое в test. Offline-метрика будет завышена.

### 17.3. Treatment

Если модель выбирает, кому дать скидку, признаки должны быть рассчитаны до назначения:

- покупки до snapshot;
- история промо до snapshot;
- активность до snapshot;
- признаки пользователя на snapshot.

Нельзя использовать:

- факт открытия текущего промо;
- клики после отправки;
- покупки в будущем;
- итоговую сумму заказа, который возник после воздействия.

## 18. PySpark и подготовка данных для PyTorch

Spark хорошо подходит для больших joins и агрегаций, а PyTorch — для обучения модели.

Практическая схема:

~~~text
PySpark
  → расчёт point-in-time признаков
  → временной split
  → Parquet shards
  → PyTorch Dataset/DataLoader
  → обучение
~~~

Не следует:

- делать collect всей обучающей выборки;
- превращать десятки терабайт в один Pandas DataFrame;
- загружать модель PyTorch заново на каждой строке Spark UDF;
- запускать тяжёлую модель через обычный Python UDF без пакетной обработки.

Если нужен batch-инференс, лучше обрабатывать данные крупными партиями и сохранять предсказания обратно в Parquet или таблицу feature store.

## 19. Structured Streaming и события

Если признаки должны обновляться почти в реальном времени, используют Structured Streaming, например события из Kafka:

~~~text
Kafka → Spark Structured Streaming
     → watermark и дедупликация
     → агрегаты
     → таблица свежих признаков
~~~

Нужно понимать:

- event time — время события;
- processing time — время обработки;
- watermark — граница, до которой ждём опоздавшие события;
- checkpoint — состояние для восстановления;
- at-least-once может привести к повторной обработке;
- операция должна быть идемпотентной или иметь дедупликацию.

Для ежедневного churn или промо не всегда нужен streaming. Если решение обновляется раз в сутки, надёжный batch-пайплайн может быть дешевле и проще.

## 20. Чек-лист оптимизации

Перед оптимизацией задаём вопрос: где фактическое узкое место?

### Данные

- читаем только нужные колонки;
- фильтруем как можно раньше;
- используем Parquet/ORC;
- проверяем pushdown;
- убираем дубли до join.

### Join

- знаем размеры сторон;
- проверили уникальность ключей;
- broadcast только действительно маленькой таблицы;
- измерили skew;
- избегаем many-to-many без причины.

### Shuffle

- не делаем лишние repartition;
- предагрегируем до join;
- не используем coalesce(1) на большой таблице;
- выбираем число partitions по измерениям.

### Python

- предпочитаем встроенные functions;
- UDF используем только при необходимости;
- для Python-логики рассматриваем Pandas UDF;
- не вызываем collect без доказательства, что результат маленький.

### Память

- не cache-им всё подряд;
- используем persist с подходящим уровнем;
- освобождаем cache;
- контролируем broadcast и размер partition.

### ML-корректность

- временной split;
- point-in-time признаки;
- правильный treatment;
- стабильная единица наблюдения;
- delayed labels;
- метрики по сегментам.

## 21. Типовые ошибки

### Ошибка 1. collect большой таблицы

Данные собираются на driver, память заканчивается. Используем write, limit, sample или агрегируем до маленького результата.

### Ошибка 2. count ради проверки на каждом шаге

Каждый count запускает отдельный job. Для диагностики лучше продумать точки материализации, использовать explain и Spark UI.

### Ошибка 3. withColumn в огромном цикле

Много последовательных withColumn может раздувать план. Объединяем выражения в один select или аккуратно группируем преобразования.

### Ошибка 4. broadcast без проверки размера

«Маленькая» таблица выросла, каждый executor получил большую копию и упал по памяти.

### Ошибка 5. repartition перед каждой операцией

Каждый repartition — потенциальный shuffle. Нужен только там, где изменение распределения помогает следующей операции.

### Ошибка 6. coalesce(1)

Вся запись превращается в один task. Это допустимо только для маленьких служебных файлов.

### Ошибка 7. UDF для простой логики

lower, trim, when, regexp_extract и date functions уже есть во встроенном API. UDF здесь обычно только замедляет job.

### Ошибка 8. неверное временное окно

rowsBetween по числу строк принимают за окно в днях. Для времени используем rangeBetween по timestamp или условный join.

### Ошибка 9. join размножил строки

После join число строк неожиданно выросло в 5 раз. Проверяем кардинальность и уникальность ключа справа.

### Ошибка 10. cache не материализован

После cache сразу измеряют время следующей операции, но первый action всё ещё строит и загружает данные. Для честного сравнения материализуем cache.

### Ошибка 11. Spark-метрика высокая, модель бесполезна

Признаки могли содержать leakage или быть рассчитаны не в тот момент. Data engineering корректен только тогда, когда корректен с точки зрения принятия ML-решения.

## 22. Что могут спросить на собеседовании

### «Чем PySpark DataFrame отличается от Pandas DataFrame?»

PySpark DataFrame распределён по partitions, ленив и обрабатывается executors. Pandas DataFrame живёт в памяти одного процесса и считает операции сразу. На маленьких данных Pandas обычно проще и быстрее, а PySpark оправдан размером данных и распределёнными вычислениями.

### «Что такое shuffle?»

Shuffle — перераспределение записей между executors по ключу или другой границе. Он нужен для groupBy, join и repartition, но дорог из-за сети, диска и сортировки.

### «repartition и coalesce?»

repartition делает полное перераспределение и может менять число partitions вверх или вниз. coalesce обычно уменьшает число partitions без полного shuffle. repartition используют для баланса и ключа, coalesce — чаще перед записью небольшого результата.

### «Когда broadcast join?»

Когда одна сторона гарантированно мала и помещается в память каждого executor. Проверяем размер и уникальность ключей; при росте справочника broadcast может привести к OOM.

### «Как найти skew?»

Смотрю Spark UI: разброс длительности tasks, shuffle read и размер partition. Затем исследую частоты ключей, ищу heavy hitters, пробую предагрегацию, broadcast или salting.

### «Почему UDF медленный?»

Обычный Python UDF требует обмена JVM и Python worker и хуже оптимизируется Catalyst. Встроенные SQL-функции предпочтительнее; Pandas UDF может уменьшить overhead за счёт пакетной обработки.

### «Как строить признаки для uplift?»

Фиксирую snapshot, считаю признаки только до него, отдельно сохраняю treatment и target в будущем окне, проверяю overlap и временной split. Spark использую для больших joins и агрегатов, а не для причинного вывода сам по себе.

### «Что смотреть в explain?»

Ищу лишние Exchange, неожиданное отсутствие broadcast, PythonUDF, чтение лишних колонок и отсутствие predicate pushdown. Затем подтверждаю гипотезу в Spark UI.

### «Что такое lazy evaluation?»

Transformations только строят план, а action запускает расчёт. Благодаря этому Spark оптимизирует всю цепочку до выполнения.

## 23. Практические задания

1. Построить признаки orders_7d, orders_30d и avg_check_90d для user × snapshot без утечки будущих заказов.
2. Реализовать deduplication заказов через row_number и сравнить с dropDuplicates.
3. Сравнить join большой таблицы с маленьким справочником обычным способом и через broadcast.
4. Создать искусственный skew по одному user_id и объяснить, как обнаружить его в Spark UI.
5. Реализовать last_order_days через оконную функцию.
6. Показать на примере разницу rowsBetween и rangeBetween.
7. Найти в explain плане Exchange и объяснить, какая операция его вызвала.
8. Сравнить обычный Python UDF и встроенную regexp-функцию.
9. Сформировать таблицу признаков, сохранить её в Parquet и проверить column pruning.
10. Объяснить, почему случайный split опасен для churn и uplift.
11. Спроектировать ежедневный batch-пайплайн признаков и вариант near-real-time через Kafka.
12. Ответить, как обработать неизвестного пользователя и позднее событие.

## 24. Главная формулировка для интервью

> «Я рассматриваю PySpark как распределённый слой подготовки данных, а не как замену Pandas один к одному. Сначала фиксирую единицу наблюдения и момент принятия решения, чтобы признаки не использовали будущее. Затем фильтрую и дедуплицирую события, строю point-in-time признаки через агрегации и окна, аккуратно соединяю справочники, проверяя кардинальность и skew. Для оптимизации предпочитаю встроенные Spark SQL-функции, использую broadcast только для гарантированно маленькой таблицы, понимаю цену shuffle и различие repartition/coalesce. План проверяю через explain и Spark UI. После этого сохраняю версионированные Parquet-данные для PyTorch, а качество модели проверяю не только offline, но и на временном holdout и A/B-тесте».

## 25. Что запомнить

PySpark-код для ML считается хорошим, если он одновременно:

- распределённо обрабатывает данные;
- не собирает большие таблицы на driver;
- не создаёт лишний shuffle;
- правильно работает с joins и partitions;
- использует встроенные функции вместо лишнего Python;
- сохраняет временную причинность признаков;
- воспроизводим и версионируем;
- выдаёт данные в формате, удобном для обучения и инференса.

В задачах CVM особенно важно помнить:

> Быстрый job с leakage — хуже, чем более медленный, но честный job. Сначала правильность момента времени и target, затем оптимизация исполнения.
