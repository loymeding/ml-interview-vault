---
номер: 41
название: Missing Number
ссылка: https://leetcode.com/problems/missing-number/
тип: xor
сложность: Easy
алгоритм: O(N)
попытки: "1"
дата: 13 мая 2026г.
статус: решено
повторить: false
---

# Missing Number

## Решение 1(дополнительный список)

Сложность O(N) по скорости, O(N) по памяти

```python
class Solution:
	def missingNumber(self, nums: List[int]) -> int:
		n = len(nums)
		v = [-1] * (n + 1)
		for num in nums: v[num] = num
			for i in range(len(v)):
				if v[i] == -1:
					return i
		return 0
```
## Решение 2 (XOR)

Сложность O(N) по скорости, O(1) по памяти

```python
class Solution:
	def missingNumber(self, nums: List[int]) -> int:
		n = len(nums)
		ans = 0
		for i in range(1, n + 1):
			ans ^= i
		for num in nums:
			ans ^= num
		return ans
```
## Ключевой инсайт

Использование XOR для поиска пропуска.
У нас len(nums) чисел из последовательности [0; len(nums) + 1] и одно значение пропущено.
Решение строится на двух свойствах:
1) X ^ X = 0
2) X ^ 0 = X
То есть если вся наша последовательность [0; len(nums) + 1], то можем выписать:
0 ^ 1 ^ 2 ^ 3 ... ^ len(nums) + 1 и добавить в эту операцию все элементы из nums. Тогда по первому свойству все дубликаты занулятся, а пропущенное значение ^ 0 даст само пропущенное значение.

![[Pasted image 20260513154940.png]]

![[Pasted image 20260513154946.png]]
## Где застрял
