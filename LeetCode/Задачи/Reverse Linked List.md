---
номер: 40
название: Reverse Linked List
ссылка: https://leetcode.com/problems/reverse-linked-list/
тип: 
сложность: Easy
алгоритм: 
попытки: 
дата: 
статус: решено
повторить: false
---

# Reverse Linked List


## Решение

```python
# встаclass Solution:

    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:

        node = None

  

        while head:

            temp = head.next

            head.next = node

            node = head

            head = temp

        return nodeвь свой код
```

## Ключевой инсайт

Освоил метод обращения ссылок в связном списке.

## Где застрял

В целом было тяжело прийти к идее как это сделать. Думал через рекурсию, но не получилось.
