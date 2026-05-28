---
tags: [interview-problem, algorithms, yandex, probability]
cssclasses: [problem-card]
интервью: true
статус: не решено
источник: yandex-custom
платформа: Custom
номер: YA-003
название: Sampling из дискретного распределения
ссылка:
сложность: Medium
попытки:
дата:
повторить: false
порядок: 903
---

# YA-003 — Sampling из дискретного распределения

> [!important] 🎯 Режим решения
> Сначала реши задачу по условию. Подсказка, идея, метаданные и решение спрятаны ниже, чтобы не подглядывать паттерн раньше времени.

## 📌 Условие

Есть дискретное распределение вероятностей, например:

```python
{"a": 0.1, "b": 0.3, "c": 0.4, "d": 0.2}
```

Нужно написать класс, который:

1. Инициализируется конкретным распределением.
2. Имеет метод `sample(n)`.
3. Возвращает `n` элементов, выбранных согласно заданным вероятностям.

Пример интервалов для распределения:

```text
элементы:  a    b    c    d
вероятн.:  0.1  0.3  0.4  0.2
cumsum:    0.1  0.4  0.8  1.0
```

## ✅ Прогресс

- [ ] Решал сам
- [ ] Решил без подсказки
- [ ] Разобрал оптимальное решение
- [ ] Повторить

> [!info]- 🧭 Метаданные / спойлер
> Паттерн: prefix sums, binary search, random sampling.
> Похожие темы: probability, bisect, design.

> [!tip]- 💡 Подсказка
> Преврати вероятности в отрезки на линии от `0` до общей суммы весов. Случайное число попадает в один из этих отрезков.

> [!abstract]- 🧠 Идея
> На этапе инициализации строим массив накопленных сумм `cumsum`. Чем больше вероятность элемента, тем длиннее его интервал. Для одного sample генерируем случайное число `r` и бинарным поиском находим первый накопленный порог, который не меньше `r`.
>
> Инициализация: `O(n)`. Один sample: `O(log n)`. Память: `O(n)`.

> [!success]- ✅ Решение
> ```python
> import bisect
> import random
>
>
> class DiscreteDistribution:
>     def __init__(self, distribution: dict[str, float]):
>         if not distribution:
>             raise ValueError("Distribution must not be empty")
>
>         self.elements = []
>         self.cumsum = []
>         total = 0.0
>
>         for elem, weight in distribution.items():
>             if weight < 0:
>                 raise ValueError("Weights must be non-negative")
>             if weight == 0:
>                 continue
>             total += weight
>             self.elements.append(elem)
>             self.cumsum.append(total)
>
>         if total <= 0:
>             raise ValueError("At least one weight must be positive")
>
>         self.total = total
>
>     def sample(self, n: int = 1):
>         if n <= 0:
>             return []
>
>         result = []
>         for _ in range(n):
>             r = random.random() * self.total
>             idx = bisect.bisect_left(self.cumsum, r)
>             result.append(self.elements[idx])
>
>         return result[0] if n == 1 else result
> ```
