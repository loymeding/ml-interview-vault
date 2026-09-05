---
tags: [big-data, spark, yarn, kafka, hdfs, delta-lake, s3, acid]
тип: теория и практика
уровень: middle
сложность: средняя
статус: готово
готовность: 90
создано: 2026-05-28
источники:
  - "Вопросы пользователя по Big Data"
предпосылки: []
связано:
  - "PySpark — DataFrame, lazy evaluation, joins и execution model"
  - "PySpark — практический пайплайн признаков и оптимизация под ML"
сравнить-с: []
---

# Big Data — Spark, YARN, Kafka, HDFS, Delta Lake

> [!abstract] Суть
> Big Data-системы решают разные части одной задачи: Spark считает распределённо, YARN выдаёт ресурсы, Kafka доставляет поток событий, HDFS хранит блоки с репликацией, а Delta Lake добавляет транзакционный слой поверх object storage вроде S3.

## Ответ

### 1. Spark DataFrame vs RDD с точки зрения оптимизации

RDD — низкоуровневый API Spark: распределённая коллекция объектов и операций над ними. Spark знает, что есть цепочка transformations, но плохо понимает смысл пользовательского кода.

DataFrame — табличный API со schema. Spark видит колонки, типы, фильтры, joins, aggregations и может оптимизировать план через Catalyst optimizer.

DataFrame лучше оптимизируется, потому что Spark может:
- pushdown фильтров;
- reorder joins;
- prune ненужные колонки;
- выбирать physical plan;
- использовать Tungsten/code generation;
- читать только нужные parquet/orc колонки.

RDD уместен, когда нужна низкоуровневая логика, нестандартные структуры данных или API, которого нет в DataFrame.

### 2. Архитектура YARN

YARN разделяет управление ресурсами и выполнение приложений.

Главные роли:
- **ResourceManager** — глобальный менеджер ресурсов кластера, принимает заявки и распределяет containers;
- **NodeManager** — агент на каждой машине, запускает containers и следит за ресурсами узла;
- **ApplicationMaster** — процесс конкретного приложения, договаривается с ResourceManager о ресурсах и координирует tasks.

Spark on YARN запускает driver/ApplicationMaster и executors в containers, выделенных YARN.

### 3. Kafka и at-least-once delivery

At-least-once означает: сообщение будет доставлено минимум один раз, но дубликаты возможны.

Kafka обеспечивает это через:
- запись сообщений в replicated log;
- offsets у consumer group;
- commit offset после обработки;
- retries producer/consumer;
- replication factor и leader/follower replicas.

Типичный сценарий дубликата: consumer обработал сообщение, но упал до commit offset. После рестарта он прочитает сообщение снова.

Вывод: downstream-обработка должна быть idempotent или дедуплицировать события по ключу/event_id.

### 4. Bucketing vs partitioning

Partitioning раскладывает данные по директориям по значениям колонки:

```text
date=2026-05-28/country=RU/...
```

Это помогает отбрасывать целые директории при фильтре.

Bucketing раскладывает строки по hash ключа в фиксированное число bucket-файлов. Это полезно для повторяющихся joins/aggregations по одному ключу.

Зачем bucketing, если есть partitioning:
- partitioning по высококардинальному ключу создаст слишком много мелких файлов;
- bucketing сохраняет фиксированное число buckets;
- bucketed join может уменьшить shuffle, если таблицы bucketed по одному ключу и совместимому числу buckets.

### 5. Что происходит при потере DataNode в HDFS

HDFS хранит файлы блоками, каждый блок обычно реплицируется на несколько DataNode.

Если один DataNode потерян:
1. NameNode перестаёт получать heartbeat от DataNode;
2. блоки на нём считаются under-replicated;
3. NameNode планирует восстановление реплик с surviving DataNodes;
4. кластер возвращает replication factor к норме.

Потеря одного DataNode обычно не означает потерю данных, если replication factor был достаточным и не потеряны все реплики блока.

### 6. Delta Lake поверх S3

S3 — object storage, не полноценная транзакционная БД. Delta Lake добавляет transaction log (`_delta_log`), который хранит историю атомарных изменений таблицы.

Delta Lake даёт:
- ACID transactions;
- schema enforcement/evolution;
- time travel;
- snapshot isolation;
- atomic commit на уровне transaction log;
- возможность безопасных concurrent reads/writes.

Для аналитики это значит: читатель видит консистентный snapshot таблицы, а не полузаписанное состояние из набора файлов.

## Формула / Схема

**Kafka at-least-once:**

```text
poll -> process -> commit offset
```

Если падение произошло после `process`, но до `commit`, сообщение будет обработано повторно.

**HDFS replication:**

$$\Large \text{storage overhead} \approx \text{file size} \times \text{replication factor}$$

где replication factor обычно равен 3 в классических HDFS-кластерах.

## Короткий пример

Если Spark job каждый день join-ит события с таблицей пользователей по `user_id`, partitioning по `user_id` может создать миллионы директорий. Bucketing по `user_id` в 1024 buckets даст управляемое число файлов и может уменьшить shuffle.

## Типичные ошибки

- **Думать, что DataFrame — просто красивый RDD:** DataFrame даёт Spark больше семантики для оптимизации.
- **Коммитить Kafka offset до обработки:** при падении можно потерять сообщение.
- **Партиционировать по высококардинальному ключу:** получите small files problem.
- **Считать S3 транзакционной таблицей:** Delta Lake добавляет transaction log именно для консистентности.

## Каверзные вопросы

> [!question] Когда RDD может быть уместнее DataFrame?
> Когда нужна кастомная низкоуровневая логика, которую сложно выразить через SQL/DataFrame API, или работа с нестандартными объектами.

> [!question] Почему at-least-once требует idempotency?
> Потому что повторная доставка допустима. Если обработчик не идемпотентен, один event может дважды изменить состояние.

## Проверка себя

- Чем ResourceManager отличается от NodeManager?
- Почему bucketing помогает join?
- Почему потеря одного DataNode обычно не теряет данные?
- Какие ACID-свойства Delta Lake добавляет поверх S3?

## Предпосылки

- Основы распределённых систем

## Связано

- [[Big Data/PySpark — DataFrame, lazy evaluation, joins и execution model]]
- [[Big Data/PySpark — практический пайплайн признаков и оптимизация под ML|PySpark для ML и задач X5]]

## Источники

- Вопросы пользователя по Big Data

---
[[🗺️ Индекс|Назад к разделу]]
