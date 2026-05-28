---
topic: Python — Структуры данных — Словари (Dict)
card_ref: "[[Python — Структуры данных — Словари (Dict)]]"
domain: python
generated: 2026-05-22
---

Failed to authenticate. API Error: 401 {"type":"error","error":{"type":"authentication_error","message":"Invalid authentication credentials"},"request_id":"req_011CbJ9Wdegq3eiKpPJeepfR"}

## T001
type: implement
difficulty: junior
tags: [python, dict, частоты]
time_limit: 15
text: Найти первый уникальный класс в списке меток.

### Условие
Дан список строк `labels` — метки классов, предсказаний или категорий. Нужно вернуть первую метку, которая встречается в списке ровно один раз.

Если такой метки нет, верните `None`.

Edge cases:
- пустой список;
- все метки повторяются;
- уникальных меток несколько — вернуть самую раннюю по исходному порядку;
- список может содержать одну метку.

### Примеры
```
Вход: ["cat", "dog", "cat", "fish"]
Выход: "dog"

Вход: ["spam", "spam"]
Выход: None
```

### Сигнатура
```python
def first_unique_label(labels: list[str]) -> str | None:
    pass
```

### Видимые тесты
```python
assert first_unique_label(["cat", "dog", "cat", "fish"]) == "dog"
assert first_unique_label(["spam", "spam"]) is None
assert first_unique_label([]) is None
```

### Скрытые тесты
```python
assert first_unique_label(["a"]) == "a"
assert first_unique_label(["a", "b", "c"]) == "a"
assert first_unique_label(["x", "y", "x", "z", "y"]) == "z"
assert first_unique_label(["m", "n", "m", "n", "p", "q"]) == "p"
```

### Setup SQL
```sql

```

### Expected JSON
```json
[]
```

### Эталонное решение
```python
def first_unique_label(labels: list[str]) -> str | None:
    counts = {}

    for label in labels:
        counts[label] = counts.get(label, 0) + 1

    for label in labels:
        if counts[label] == 1:
            return label

    return None
```

### Критерии оценки
- optimal_complexity: O(n)
- key_concepts: [словарь, подсчёт частот, сохранение порядка обхода]
- common_mistakes: [вернуть любую уникальную метку вместо первой, использовать labels.count внутри цикла и получить O(n^2)]
- allowed_imports: []
- interview_focus: [correctness, edge_cases, complexity]
- follow_up: Как изменить функцию, чтобы она возвращала все уникальные метки в исходном порядке?
