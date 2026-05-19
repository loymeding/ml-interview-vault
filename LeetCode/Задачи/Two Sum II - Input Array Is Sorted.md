---
номер: 45
название: Two Sum II - Input Array Is Sorted
ссылка: https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/
тип: binary search
сложность: Medium
алгоритм: O(1)
попытки: "1"
дата: 14 мая 2026г.
статус: решено
повторить: false
---

# Two Sum II - Input Array Is Sorted

## Решение 1. Бинарный поиск с полным проходом

```python
class Solution:

    def twoSum(self, numbers: List[int], target: int) -> List[int]:

        l = 0

        r = len(numbers) - 1



        while l < r:

            curr_sum = numbers[l] + numbers[r]

            if curr_sum == target:

                return [l + 1, r + 1]

            elif curr_sum > target:

                r -= 1

            else:

                l += 1
```

## Ключевой инсайт

## Где застрял
