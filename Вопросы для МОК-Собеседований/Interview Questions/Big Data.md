---
topic: Big Data
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/Big Data|Big Data]]"
generated: 2026-05-19
source: "user-provided extra mock questions"
---

## Q001
type: compare
difficulty: middle
key_concepts: [DataFrame schema, Catalyst optimizer, logical/physical plan, RDD низкоуровневый API]
optional_concepts: [Tungsten, type safety Dataset]
text: Чем DataFrame в Spark отличается от RDD с точки зрения оптимизации?
follow_up:
  - Почему Spark SQL может оптимизировать DataFrame лучше, чем произвольный RDD-код?
  - Когда RDD всё ещё может быть уместен?

## Q002
type: system_design
difficulty: middle
key_concepts: [ResourceManager, NodeManager, ApplicationMaster, containers]
optional_concepts: [scheduler, heartbeats]
text: Опишите архитектуру YARN: ResourceManager и NodeManager.
follow_up:
  - Кто принимает решение о выделении ресурсов приложению?
  - Что произойдёт, если NodeManager перестанет отвечать?

## Q003
type: fact
difficulty: middle
key_concepts: [consumer offsets, commit after processing, acks, повторная обработка]
optional_concepts: [idempotent producer, consumer group]
text: Как Kafka обеспечивает at-least-once доставку сообщений?
follow_up:
  - Почему at-least-once может приводить к дублям?
  - Что нужно сделать на стороне consumer, чтобы безопасно переживать дубли?

## Q004
type: compare
difficulty: middle
key_concepts: [partitioning по директориям, bucketing по hash, уменьшение shuffle, bucketed joins]
optional_concepts: [количество buckets, sort within bucket]
text: Зачем использовать bucketing, если уже есть partitioning?
follow_up:
  - Когда partitioning создаёт слишком много мелких файлов?
  - Как bucketing помогает при повторяющихся join по одному ключу?

## Q005
type: system_design
difficulty: middle
key_concepts: [replication factor, NameNode metadata, block reports, re-replication]
optional_concepts: [rack awareness, heartbeat timeout]
text: Что происходит при потере одного из DataNode в HDFS?
follow_up:
  - Почему потеря одного DataNode обычно не означает потерю данных?
  - Какая роль NameNode при восстановлении реплик?

## Q006
type: fact
difficulty: middle
key_concepts: [transaction log, atomic commits, snapshot isolation, schema enforcement]
optional_concepts: [time travel, optimistic concurrency]
text: Какие гарантии ACID даёт Delta Lake поверх S3?
follow_up:
  - Как transaction log помогает получить consistency на object storage?
  - Что такое time travel в Delta Lake?
