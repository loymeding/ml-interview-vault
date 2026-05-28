---
tags: [interview-problem, algorithms, yandex, strings]
cssclasses: [problem-card]
интервью: true
статус: не решено
источник: yandex-custom
платформа: Custom
номер: YA-001
название: Кратчайшее расстояние между X и Y
ссылка:
сложность: Easy
попытки:
дата:
повторить: false
порядок: 901
---

# YA-001 — Кратчайшее расстояние между X и Y

> [!important] 🎯 Режим решения
> Сначала реши задачу по условию. Подсказка, идея, метаданные и решение спрятаны ниже, чтобы не подглядывать паттерн раньше времени.

## 📌 Условие

Дана строка, состоящая из символов `"X"`, `"Y"` и `"0"`. Нужно найти кратчайшее расстояние между символами `"X"` и `"Y"`.

Если в строке нет хотя бы одного из символов `"X"` или `"Y"`, вернуть `0`.

```python
assert distance("XY") == 1
assert distance("X0Y") == 2
assert distance("X00Y") == 3
assert distance("YX") == 1
assert distance("Y00X0X") == 3
assert distance("XYYY") == 1
assert distance("X") == 0
assert distance("Y") == 0
assert distance("000") == 0
assert distance("X0000Y0X") == 1
```

## ✅ Прогресс

- [ ] Решал сам
- [ ] Решил без подсказки
- [ ] Разобрал оптимальное решение
- [ ] Повторить

> [!info]- 🧭 Метаданные / спойлер
> Паттерн: один проход, последние позиции.
> Похожие темы: строки, two pointers.

> [!tip]- 💡 Подсказка
> Достаточно помнить последнюю позицию `"X"` и последнюю позицию `"Y"`. Когда встречаешь один символ, можно сразу посчитать расстояние до последнего противоположного.

> [!abstract]- 🧠 Идея
> Проходим строку один раз. Храним `last_x`, `last_y` и текущий минимум. При встрече `"X"` обновляем `last_x`; если раньше уже видели `"Y"`, обновляем минимум расстоянием `i - last_y`. Аналогично для `"Y"`.
>
> Время: `O(n)`. Память: `O(1)`.

> [!success]- ✅ Решение
> ```python
> def distance(string: str) -> int:
>     last_x = -1
>     last_y = -1
>     min_dist = float("inf")
>
>     for i, char in enumerate(string):
>         if char == "X":
>             last_x = i
>             if last_y != -1:
>                 min_dist = min(min_dist, i - last_y)
>         elif char == "Y":
>             last_y = i
>             if last_x != -1:
>                 min_dist = min(min_dist, i - last_x)
>
>     return 0 if min_dist == float("inf") else min_dist
> ```
