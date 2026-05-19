---
номер: 42
название: Binary Tree Paths
ссылка: https://leetcode.com/problems/binary-tree-paths/
тип: tree
сложность: Easy
алгоритм: O(N)
попытки: "1"
дата: 13 мая 2026г.
статус: решено
повторить: false
---

# Binary Tree Paths

## Решение

```python
class Solution:

    def binaryTreePaths(self, root: Optional[TreeNode]) -> List[str]:

        def helper(node, path, result):

            if not node:

                return

            path += str(node.val)



            if not node.left and not node.right:

                result.append(path)    

            else:

                helper(node.left, path + '->', result)

                helper(node.right, path + '->', result)

        result = []

        helper(root, '', result)

        return result
```

## Ключевой инсайт

## Где застрял
