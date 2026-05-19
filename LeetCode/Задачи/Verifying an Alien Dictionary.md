---
номер: 46
название: Verifying an Alien Dictionary
ссылка: https://leetcode.com/problems/verifying-an-alien-dictionary/
тип:
сложность: Easy
алгоритм: O(N)
попытки: "1"
дата: 14 мая 2026г.
статус: решено
повторить: false
---

# Verifying an Alien Dictionary

## Решение

```python
class Solution:

    def isAlienSorted(self, words: List[str], order: str) -> bool:

        order_map = {char: i for i , char in enumerate(order)}



        for i in range (len(words) - 1):

            w1 = words[i]

            w2 = words[i + 1]



            for j in range (min(len(w1),len(w2))):

                if w1[j] != w2[j]:

                    if order_map[w1[j]] > order_map[w2[j]]:

                        return False

                    break

            else:

                if len(w1) > len(w2):

                    return False




        return True
```

## Ключевой инсайт

## Где застрял
