---
tags: [leetcode, interview-problem, algorithms, sber]
cssclasses: [problem-card]
интервью: true
статус: не решено
источник: sber-custom
платформа: LeetCode
номер: 977
название: Squares of a Sorted Array
ссылка: https://leetcode.com/problems/squares-of-a-sorted-array/
сложность: Easy
попытки:
дата:
повторить: false
порядок: 909
---

# LC-0977 — Squares of a Sorted Array

> [!important] 🎯 Режим решения
> Сначала реши задачу по условию. Подсказка, идея, метаданные и решение спрятаны ниже, чтобы не подглядывать паттерн раньше времени.

## 📌 Условие

На вход приходит отсортированный список чисел. Нужно вернуть отсортированный список квадратов этих чисел.

```python
assert sorted_squares([1, 2, 3, 4, 5]) == [1, 4, 9, 16, 25]
assert sorted_squares([2, 2]) == [4, 4]
assert sorted_squares([-2, -1, 4]) == [1, 4, 16]
```

[Открыть условие на LeetCode](https://leetcode.com/problems/squares-of-a-sorted-array/)

## ✅ Прогресс

- [ ] Решал сам
- [ ] Решил без подсказки
- [ ] Разобрал оптимальное решение
- [ ] Повторить

> [!info]- 🧭 Метаданные / спойлер
> Паттерн: two pointers, merge двух отсортированных потоков.
> Похожие темы: arrays, sorting, two pointers.

> [!tip]- 💡 Подсказка
> Отрицательная часть массива после возведения в квадрат становится отсортированной, если идти по ней справа налево. Неотрицательная часть уже дает квадраты в отсортированном порядке.

> [!abstract]- 🧠 Идея
> Найдем `edge` — индекс первого неотрицательного элемента. Левая часть `nums[:edge]` отрицательная, правая часть `nums[edge:]` неотрицательная. Идем по левой части справа налево и по правой слева направо, сливая два отсортированных потока квадратов.
>
> Время: `O(n)`, если `edge` искать линейно, или `O(n + log n)` с бинарным поиском. Память: `O(n)` под результат.

> [!success]- ✅ Решение
> ```python
> from typing import List
>
>
> def sorted_squares(nums: List[int]) -> List[int]:
>     n = len(nums)
>     edge = find_edge(nums)
>
>     i = edge - 1
>     j = edge
>     result = []
>
>     while i >= 0 and j < n:
>         left_square = nums[i] * nums[i]
>         right_square = nums[j] * nums[j]
>
>         if left_square <= right_square:
>             result.append(left_square)
>             i -= 1
>         else:
>             result.append(right_square)
>             j += 1
>
>     while i >= 0:
>         result.append(nums[i] * nums[i])
>         i -= 1
>
>     while j < n:
>         result.append(nums[j] * nums[j])
>         j += 1
>
>     return result
>
>
> def find_edge(nums: List[int]) -> int:
>     left = 0
>     right = len(nums)
>     while left < right:
>         mid = (left + right) // 2
>         if nums[mid] < 0:
>             left = mid + 1
>         else:
>             right = mid
>     return left
> ```
