---
tags: [leetcode, interview-problem, algorithms]
cssclasses: [problem-card]
интервью: true
статус: не решено
источник: yandex-excel, sber
платформа: LeetCode
номер: 200
название: Number of Islands
ссылка: https://leetcode.com/problems/number-of-islands/
сложность: Medium
попытки:
дата:
повторить: false
порядок: 602
---

# LC-0200 — Number of Islands

> [!important] 🎯 Режим решения
> Сначала реши задачу по условию. Подсказка, идея, метаданные и решение спрятаны ниже, чтобы не подглядывать паттерн раньше времени.

## 📌 Условие

Есть карта, представленная матрицей из `"0"` и `"1"`, где `"1"` — суша, `"0"` — вода. Нужно посчитать количество островов. Ходить можно по четырем направлениям: вверх, вниз, влево, вправо.

```python
grid = [
    ["1", "1", "1", "1", "0"],
    ["1", "1", "0", "1", "0"],
    ["1", "1", "0", "0", "0"],
    ["0", "0", "0", "0", "0"],
]

assert num_islands(grid) == 1
```

```python
grid = [
    ["1", "1", "0", "0", "0"],
    ["1", "1", "0", "0", "0"],
    ["0", "0", "1", "0", "0"],
    ["0", "0", "0", "1", "1"],
]

assert num_islands(grid) == 3
```

[Открыть условие на LeetCode](https://leetcode.com/problems/number-of-islands/)

## ✅ Прогресс

- [ ] Решал сам
- [ ] Решил без подсказки
- [ ] Разобрал оптимальное решение
- [ ] Повторить

> [!info]- 🧭 Метаданные / спойлер
> Классификация из Excel: dfs/bfs
> LeetCode tags: Array, Depth-First Search, Breadth-First Search, Union Find, Matrix

> [!tip]- 💡 Подсказка
> Свести матрицу к графу: клетки `"1"` — вершины, соседство по четырем направлениям — ребра. Каждый запуск DFS/BFS из непосещенной суши находит один остров.

> [!abstract]- 🧠 Идея
> Проходим по всем клеткам. Если встретили `"1"`, увеличиваем счетчик островов и запускаем DFS/BFS, который "топит" весь остров, заменяя посещенные `"1"` на `"0"`. Отдельная матрица `visited` не нужна.
>
> Время: `O(m * n)`, потому что каждая клетка обрабатывается один раз. Память: `O(m * n)` в худшем случае из-за стека DFS/BFS.

> [!success]- ✅ Решение
> ```python
> from typing import List
>
> def num_islands(grid: List[List[str]]) -> int:
>     if not grid or not grid[0]:
>         return 0
>
>     m, n = len(grid), len(grid[0])
>     count = 0
>
>     def dfs(r: int, c: int) -> None:
>         stack = [(r, c)]
>         grid[r][c] = "0"
>
>         while stack:
>             cr, cc = stack.pop()
>             for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
>                 nr, nc = cr + dr, cc + dc
>                 if 0 <= nr < m and 0 <= nc < n and grid[nr][nc] == "1":
>                     grid[nr][nc] = "0"
>                     stack.append((nr, nc))
>
>     for i in range(m):
>         for j in range(n):
>             if grid[i][j] == "1":
>                 count += 1
>                 dfs(i, j)
>
>     return count
> ```

