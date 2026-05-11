---
номер: 56
название: Kids With the Greatest Number of Candies
ссылка: https://leetcode.com/problems/kids-with-the-greatest-number-of-candies/
тип: list
сложность: Easy
алгоритм: O(N)
попытки: "1"
дата: 2026-05-03
статус: решено
повторить: false
---

# Kids With the Greatest Number of Candies

## Решение

```python
class Solution:

    def kidsWithCandies(self, c: List[int], ec: int) -> List[bool]:

        l=[]

        d =max(c)

        for i in c:

            if i + ec >= d:

                l.append(True)

            else:

                l.append(False)

        return l
```

## Ключевой инсайт

Ничего нового не узнал, странная задача, надеялся, что в subbmits узнаю какой-то уникальный подход, позволяющий решить задачу менее примитивным и более быстрым способом, но там такие же решения.

## Где застрял
