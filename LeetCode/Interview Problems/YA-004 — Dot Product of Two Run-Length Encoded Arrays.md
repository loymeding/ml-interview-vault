---
tags: [interview-problem, algorithms, yandex, arrays]
cssclasses: [problem-card]
интервью: true
статус: не решено
источник: yandex-custom
платформа: Custom
номер: YA-004
название: Dot Product of Two Run-Length Encoded Arrays
ссылка:
сложность: Medium
попытки:
дата:
повторить: false
порядок: 904
---

# YA-004 — Dot Product of Two Run-Length Encoded Arrays

> [!important] 🎯 Режим решения
> Сначала реши задачу по условию. Подсказка, идея, метаданные и решение спрятаны ниже, чтобы не подглядывать паттерн раньше времени.

## 📌 Условие

Даны два вектора целых чисел одинаковой длины. Оба вектора заданы в сжатой форме: списками пар `(value, count)`.

Например, вектор `[4, 4, 5]` задается как:

```python
[(4, 2), (5, 1)]
```

Нужно посчитать скалярное произведение двух векторов, не разворачивая их полностью в память.

```python
assert dot_product([(1, 3)], [(1, 2), (10, 1)]) == 12
```

Сжатие сохраняет порядок элементов.

## ✅ Прогресс

- [ ] Решал сам
- [ ] Решил без подсказки
- [ ] Разобрал оптимальное решение
- [ ] Повторить

> [!info]- 🧭 Метаданные / спойлер
> Паттерн: два указателя по RLE-сегментам.
> Похожие темы: arrays, compression, two pointers.

> [!tip]- 💡 Подсказка
> Не нужно восстанавливать исходные массивы. На каждом шаге можно перемножить столько элементов, сколько одновременно осталось в текущих RLE-сегментах обоих массивов.

> [!abstract]- 🧠 Идея
> Держим два указателя на текущие пары `(value, count)` и остатки `rem_a`, `rem_b`. На каждом шаге берем `k = min(rem_a, rem_b)`, добавляем `k * value_a * value_b`, уменьшаем остатки и двигаем указатель там, где сегмент закончился.
>
> Время: `O(len(a) + len(b))`. Память: `O(1)`.

> [!success]- ✅ Решение
> ```python
> def dot_product(a: list[tuple[int, int]], b: list[tuple[int, int]]) -> int:
>     i = 0
>     j = 0
>     rem_a = 0
>     rem_b = 0
>     result = 0
>
>     while i < len(a) and j < len(b):
>         value_a, count_a = a[i]
>         value_b, count_b = b[j]
>
>         if rem_a == 0:
>             rem_a = count_a
>         if rem_b == 0:
>             rem_b = count_b
>
>         k = min(rem_a, rem_b)
>         result += k * value_a * value_b
>
>         rem_a -= k
>         rem_b -= k
>
>         if rem_a == 0:
>             i += 1
>         if rem_b == 0:
>             j += 1
>
>     return result
> ```
