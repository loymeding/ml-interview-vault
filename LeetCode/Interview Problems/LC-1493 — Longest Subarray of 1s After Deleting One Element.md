---
tags: [leetcode, interview-problem, algorithms, yandex]
cssclasses: [problem-card]
интервью: true
статус: не решено
источник: yandex-custom
платформа: LeetCode
номер: 1493
название: Longest Subarray of 1s After Deleting One Element
ссылка: https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/
сложность: Medium
попытки:
дата:
повторить: false
порядок: 905
---

# LC-1493 — Longest Subarray of 1s After Deleting One Element

> [!important] 🎯 Режим решения
> Сначала реши задачу по условию. Подсказка, идея, метаданные и решение спрятаны ниже, чтобы не подглядывать паттерн раньше времени.

## 📌 Условие

Дан непустой массив из нулей и единиц. Нужно определить максимальную длину подотрезка единиц, который можно получить, удалив ровно один элемент массива.

Удалить один элемент обязательно.

```python
assert max_ones([1, 1, 0, 1]) == 3
assert max_ones([1, 1, 0, 0, 1]) == 2
```

[Открыть условие на LeetCode](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/)

## ✅ Прогресс

- [ ] Решал сам
- [ ] Решил без подсказки
- [ ] Разобрал оптимальное решение
- [ ] Повторить

> [!info]- 🧭 Метаданные / спойлер
> Паттерн: sliding window, окно с максимум одним нулем.
> Похожие темы: arrays, sliding window.

> [!tip]- 💡 Подсказка
> Удаление одного элемента можно интерпретировать как поиск самого длинного окна, в котором не больше одного нуля.

> [!abstract]- 🧠 Идея
> Держим окно `[left, right]`, внутри которого не больше одного нуля. Если нулей стало больше одного, двигаем левую границу, пока ограничение снова не выполнится. Ответ обновляется как `window_size - 1`, потому что один элемент удаляем обязательно.
>
> Время: `O(n)`. Память: `O(1)`.

> [!success]- ✅ Решение
> ```python
> def max_ones(nums: list[int]) -> int:
>     left = 0
>     zeros = 0
>     best = 0
>
>     for right, value in enumerate(nums):
>         if value == 0:
>             zeros += 1
>
>         while zeros > 1:
>             if nums[left] == 0:
>                 zeros -= 1
>             left += 1
>
>         window_size = right - left + 1
>         best = max(best, window_size - 1)
>
>     return best
> ```
