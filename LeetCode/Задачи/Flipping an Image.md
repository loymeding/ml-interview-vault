---
номер: 44
название: Flipping an Image
ссылка: https://leetcode.com/problems/flipping-an-image/
тип: list
сложность: Easy
алгоритм: O(N^2)
попытки: "1"
дата: 14 мая 2026г.
статус: решено
повторить: false
---

# Flipping an Image

## Решение

Можно использовать XOR еще и для инверсии бинарных значений

```python
class Solution:

    def flipAndInvertImage(self, image: List[List[int]]) -> List[List[int]]:

        res = []

        for i in image:

            i.reverse()

            res.append([x ^ 1 for x in i])

        return res
```

## Ключевой инсайт

## Где застрял
