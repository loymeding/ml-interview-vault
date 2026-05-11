---
номер: 55
название: Delete Node in a Linked List
ссылка: https://leetcode.com/problems/delete-node-in-a-linked-list/
тип: linked list
сложность: Medium
алгоритм: O(1)
попытки: "1"
дата: 2036-05-03
статус: решено
повторить: false
---

# Delete Node in a Linked List

## Решение

```python
# Definition for singly-linked list.

# class ListNode:

#     def __init__(self, x):

#         self.val = x

#         self.next = None

  

class Solution:

    def deleteNode(self, node):

        """

        :type node: ListNode

        :rtype: void Do not return anything, modify node in-place instead.

        """

        node.val = node.next.val

        node.next = node.next.next
```

## Ключевой инсайт

Удалить текущий элемент связного списка можно переписав val и next в нем на значения следующей ноды, то есть копируем в текущую ноду следующую и прокидываем ссылку на последующую.
## Где застрял
