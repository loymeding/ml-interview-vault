---
topic: Pandas и PySpark
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/Pandas и PySpark|Pandas и PySpark]]"
generated: 2026-05-19
source: "user-provided extra mock questions"
---

## Q001
type: fact
difficulty: middle
key_concepts: [transformations, actions, lazy evaluation, DAG выполнения]
optional_concepts: [narrow/wide transformations]
text: Два основных типа операций в PySpark? Что такое lazy evaluation?
follow_up:
  - Почему transformation не запускает вычисления сразу?
  - Какие примеры actions ты можешь назвать?

## Q002
type: compare
difficulty: middle
key_concepts: [distributed DataFrame, локальный ndarray, schema, lazy execution]
optional_concepts: [Catalyst optimizer, row/columnar execution]
text: DataFrame в PySpark vs NumPy: в чём разница?
follow_up:
  - Почему NumPy быстрее на маленьком локальном массиве?
  - Когда PySpark DataFrame начинает выигрывать?

## Q003
type: fact
difficulty: middle
key_concepts: [ленивые transformations, action как trigger, DAG, оптимизация плана]
optional_concepts: [lineage, Catalyst optimizer]
text: Что такое lazy evaluation в контексте Spark?
follow_up:
  - Как lazy evaluation помогает оптимизировать pipeline?
  - Почему ошибка может проявиться только на collect или write?

## Q004
type: scenario
difficulty: middle
key_concepts: [broadcast join, маленькая таблица в память executors, shuffle avoidance, broadcast threshold]
optional_concepts: [broadcast hint, skew]
text: Как выполнить JOIN большой таблицы с маленькой в Spark?
follow_up:
  - Что произойдёт, если маленькая таблица на самом деле не помещается в память?
  - Как проверить физический план join?

## Q005
type: system_design
difficulty: middle
key_concepts: [Driver, Executors, Cluster Manager, Python worker processes]
optional_concepts: [JVM, Py4J]
text: Компоненты Spark-приложения? Где исполняется Python-код?
follow_up:
  - Что делает driver, а что executor?
  - Почему Python UDF может быть медленнее built-in Spark SQL функций?
