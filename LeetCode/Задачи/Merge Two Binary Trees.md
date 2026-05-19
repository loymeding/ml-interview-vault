---
номер: 43
название: Merge Two Binary Trees
ссылка: https://leetcode.com/problems/merge-two-binary-trees/
тип: tree
сложность: Easy
алгоритм: O(N)
попытки: "2"
дата: 14 мая 2026г.
статус: решено
повторить: false
---

# Merge Two Binary Trees

## Решение

```python
class Solution:

    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:

        # Если оба узла пустые — возвращаем None

        if not root1 and not root2:

            return None

        # Если один из них пуст — возвращаем другой (его поддерево)

        if not root1:

            return root2

        if not root2:

            return root1

        # Создаём новый узел с суммой значений

        merged = TreeNode(root1.val + root2.val)

        # Рекурсивно сливаем левых и правых детей

        merged.left = self.mergeTrees(root1.left, root2.left)

        merged.right = self.mergeTrees(root1.right, root2.right)

        return merged
```

## Ключевой инсайт

## Где застрял
