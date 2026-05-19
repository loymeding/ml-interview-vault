---
topic: Python
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/Python|Python]]"
generated: 2026-05-19
source: "user-provided extra mock questions"
---

## Q001
type: fact
difficulty: middle
key_concepts: [self как ссылка на экземпляр, super и MRO, наследование, инкапсуляция поведения в классах]
optional_concepts: [multiple inheritance, dunder methods]
text: Основы Python: ООП, super и self.
follow_up:
  - Что именно возвращает super() в Python?
  - Почему self явно указан в методах экземпляра?

## Q002
type: scenario
difficulty: middle
key_concepts: [функция высшего порядка, wrapper, замыкание, functools.wraps]
optional_concepts: [lru_cache, параметризованный декоратор]
text: Декораторы в Python: кэширование, retry, логирование.
follow_up:
  - Зачем использовать functools.wraps?
  - Как написать retry-декоратор, который не скрывает исходное исключение?

## Q003
type: compare
difficulty: middle
key_concepts: [instance method, staticmethod без self/cls, classmethod получает cls, альтернативные конструкторы]
optional_concepts: [factory methods, наследование classmethod]
text: Staticmethod и classmethod: чем отличаются и когда применять?
follow_up:
  - Почему classmethod удобен для альтернативного конструктора?
  - Когда staticmethod лучше заменить обычной функцией модуля?

## Q004
type: fact
difficulty: middle
key_concepts: [DEBUG INFO WARNING ERROR CRITICAL, logger hierarchy, handlers, formatters]
optional_concepts: [structured logging, propagation]
text: Логирование в Python: уровни и практическое использование.
follow_up:
  - Почему в библиотеке не стоит использовать print?
  - Как настроить разные handlers для файла и консоли?
