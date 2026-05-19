---
номер: 51
название: Majority Element
ссылка: https://leetcode.com/problems/majority-element/
тип: array
сложность: Easy
алгоритм: O(N)
попытки: "1"
дата: 15 мая 2026г.
статус: решено
повторить: false
---

# Majority Element

## Решение 1

```python
class Solution:

    def majorityElement(self, nums: List[int]) -> int:

        n = len(nums)

        m = defaultdict(int)

        for num in nums:

            m[num] += 1

        n = n // 2

        for key, value in m.items():

            if value > n:

                return key

        return 0
```

## Решение 2

```python
class Solution:
	def majorityElement(self, nums: List[int]) -> int:
		nums.sort()
		n = len(nums)
		return nums[n//2]
```

## Ключевой инсайт

## Где застрял
