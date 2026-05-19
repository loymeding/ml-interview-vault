---
topic: Базы данных SQL и NoSQL
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/Базы данных SQL и NoSQL|Базы данных SQL и NoSQL]]"
generated: 2026-05-19
source: "user-provided extra mock questions"
---

## Q001
type: fact
difficulty: middle
key_concepts: [B-tree индекс, диапазонный scan, сортированный порядок ключей, селективность условия]
optional_concepts: [covering index, leftmost prefix]
text: Как B-tree индекс ускоряет запросы с оператором BETWEEN?
follow_up:
  - Почему hash-index хуже подходит для BETWEEN?
  - Когда оптимизатор всё равно выберет sequential scan вместо индекса?

## Q002
type: compare
difficulty: middle
key_concepts: [уровни изоляции транзакций, dirty/non-repeatable/phantom reads, Read Committed, Serializable]
optional_concepts: [MVCC, serialization failure]
text: Опишите различие уровней изоляции Read Committed и Serializable.
follow_up:
  - Какой аномалии Read Committed не предотвращает?
  - Почему Serializable может приводить к abort/retry транзакций?

## Q003
type: scenario
difficulty: middle
key_concepts: [оконная функция, PARTITION BY, ORDER BY, нумерация строк внутри группы]
optional_concepts: [deduplication, top-N per group]
text: Приведите пример использования оконной функции ROW_NUMBER().
follow_up:
  - Чем ROW_NUMBER отличается от RANK и DENSE_RANK?
  - Как с помощью ROW_NUMBER оставить последнюю запись по каждому пользователю?

## Q004
type: compare
difficulty: middle
key_concepts: [бинарное хранение JSONB, индексация GIN, нормализация структуры, быстрый поиск по полям]
optional_concepts: [потеря порядка ключей, стоимость записи]
text: Какие преимущества JSONB перед JSON в PostgreSQL?
follow_up:
  - Когда JSON может быть предпочтительнее JSONB?
  - Какой индекс обычно используют для поиска по JSONB-полям?

## Q005
type: scenario
difficulty: middle
key_concepts: [ORM relationship loading, lazy loading, N+1 query problem, eager loading]
optional_concepts: [join/load strategies, select_related/prefetch_related]
text: Что такое lazy loading в ORM и когда оно вредно?
follow_up:
  - Как распознать N+1 проблему по логам SQL?
  - Когда lazy loading всё-таки удобен и безопасен?

## Q006
type: compare
difficulty: middle
key_concepts: [BASE, eventual consistency, availability, partition tolerance]
optional_concepts: [CAP theorem, soft state]
text: Почему BASE-подход часто выбирают для NoSQL-систем?
follow_up:
  - Как BASE связан с CAP theorem?
  - Как объяснить бизнесу риск eventual consistency?

## Q007
type: compare
difficulty: middle
key_concepts: [INNER JOIN, LEFT/RIGHT JOIN, FULL OUTER JOIN, CROSS JOIN]
optional_concepts: [semi/anti join, NULL в join key]
text: Чем отличаются разные типы JOIN?
follow_up:
  - Что произойдёт с unmatched строками при LEFT JOIN?
  - Как получить строки из A, которым нет пары в B?

## Q008
type: fact
difficulty: middle
key_concepts: [оконная функция, OVER, PARTITION BY, ORDER BY]
optional_concepts: [running total, lag/lead]
text: Что такое оконная функция? Примеры использования.
follow_up:
  - Чем оконная функция отличается от GROUP BY?
  - Когда полезны LAG и LEAD?
