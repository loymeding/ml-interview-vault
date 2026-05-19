---
номер: 57
название: Reverse Vowels of a String
ссылка: https://leetcode.com/problems/reverse-vowels-of-a-string/
тип: two pointers
сложность: Easy
алгоритм: O(N)
попытки: "2"
дата: 18 мая 2026г.
статус: решено
повторить: false
---

# Reverse Vowels of a String

## Решение

```python
class Solution:
    def reverseVowels(self, s: str) -> str:
        vowels_set = {'a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U'}

        l = 0
        r = len(s) - 1
        chars = list(s)

        while l < r:
            while l < r and chars[l] not in vowels_set:
                l += 1
            while l < r and chars[r] not in vowels_set:
                r -= 1

            if l < r:
                chars[l], chars[r] = chars[r], chars[l]
                l += 1
                r -= 1

        return ''.join(chars)
```

## Ключевой инсайт

## Где застрял
