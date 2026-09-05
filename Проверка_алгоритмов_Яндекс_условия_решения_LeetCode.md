# Задачи Яндекса: условия, решения и соответствия LeetCode

## Сводная таблица

|  ID | Задача                                                        | Сложность сайта | LeetCode                                                                                                                                             |
| --: | ------------------------------------------------------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
|   4 | Переместить нули в конец                                      | Easy            | [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/)                                                                                       |
|   5 | Отражение относительно прямой                                 | Easy            | [356. Line Reflection](https://leetcode.com/problems/line-reflection/)                                                                               |
|   6 | Сумма двух чисел                                              | Easy            | [1. Two Sum](https://leetcode.com/problems/two-sum/)                                                                                                 |
|   7 | Палиндром                                                     | Easy            | [9. Palindrome Number](https://leetcode.com/problems/palindrome-number/)                                                                             |
|  10 | Самая длинная подстрока без повторяющихся символов            | Medium          | [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)                   |
|  12 | Слияние двух отсортированных списков                          | Easy            | [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)                                                                  |
|  15 | Наибольшая возрастающая подпоследовательность                 | Easy            | [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)                                                 |
|  16 | Удаление лишних скобок                                        | Hard            | [301. Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/)                                                         |
|  17 | Произведение двух RLE-массивов                                | Medium          | [1868. Product of Two Run-Length Encoded Arrays](https://leetcode.com/problems/product-of-two-run-length-encoded-arrays/)                            |
|  18 | Максимальная длина последовательности единиц                  | Easy            | [485. Max Consecutive Ones](https://leetcode.com/problems/max-consecutive-ones/)                                                                     |
|  19 | Симметричное дерево                                           | Hard            | [101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)                                                                                 |
|  21 | Максимальная длина подмассива из единиц после одного удаления | Medium          | [1493. Longest Subarray of 1's After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/)         |
|  24 | Количество островов                                           | Hard            | [200. Number of Islands](https://leetcode.com/problems/number-of-islands/)                                                                           |
|  25 | Дождевая вода                                                 | Hard            | [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)                                                                        |
|  26 | Минимальная подстрока окна                                    | Hard            | [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)                                                              |
|  27 | sort-the-matrix-diagonally                                    | Hard            | [1329. Sort the Matrix Diagonally](https://leetcode.com/problems/sort-the-matrix-diagonally/)                                                        |
|  28 | Суммирование диапазонов                                       | Medium          | [228. Summary Ranges](https://leetcode.com/problems/summary-ranges/)                                                                                 |
|  30 | Группировка анаграмм                                          | Medium          | [49. Group Anagrams](https://leetcode.com/problems/group-anagrams/)                                                                                  |
|  36 | Минимальное количество перемещений ящиков                     | Hard            | не подтверждён                                                                                                                                       |
|  37 | Слияние интервалов                                            | Easy            | [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)                                                                                |
|  40 | Максимальное количество подряд идущих одинаковых символов     | Easy            | [1446. Consecutive Characters](https://leetcode.com/problems/consecutive-characters/)                                                                |
|  41 | Покупка и продажа акций                                       | Medium          | [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)                                               |
|  42 | Максимальная длина подстроки из двух уникальных символов      | Easy            | [159. Longest Substring with At Most Two Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-two-distinct-characters/) |
|  68 | Скалярное произведение сжатых векторов                        | Medium          | [1868. Product of Two Run-Length Encoded Arrays](https://leetcode.com/problems/product-of-two-run-length-encoded-arrays/)                            |
|  69 | Сжатие последовательных пробелов                              | Medium          | не подтверждён                                                                                                                                       |

## Массивы и два указателя

### Дождевая вода

- ID: 25, slug: `trapping-rain-water`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Hard**
- Темы: Два указателя, Массивы
- Страница сайта: [https://botayinterview.site/algorithm/trapping-rain-water](https://botayinterview.site/algorithm/trapping-rain-water)
- Публичный detail API: `GET /api/algorithms/public/25` → **HTTP 200**, 3131 байт
- LeetCode: [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) — **точное**

**Условие**

Дан массив неотрицательных целых чисел, представляющих карту высот, где ширина каждого стержня равна 1. Необходимо вычислить, сколько воды может задержать эта карта высот после дождя.

**Открытые тесты**

Тест 1:

```text
Вход:
height = [0,1,0,2,1,0,1,3,2,1,2,1]
Выход:
6
```

Тест 2:

```text
Вход:
height = [4,2,0,3,2,5]
Выход:
9
```

**Starter code**

````python
def trap(height):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

Для решения этой задачи необходимо найти максимальное количество воды, которое может быть задержано между стержнями. Для этого мы используем два массива, `left` и `right`, для хранения максимальных высот слева и справа от каждого стержня соответственно.

ШАГИ:
1. Инициализируйте два массива, `left` и `right`, с одинаковой длиной, что и входной массив `height`. Заполните первые элементы `left` и последние элементы `right` значениями из `height`.
2. Пройдите по массиву `height` слева направо и для каждого элемента вычислите максимальную высоту слева, сохраняя результат в `left`.
3. Пройдите по массиву `height` справа налево и для каждого элемента вычислите максимальную высоту справа, сохраняя результат в `right`.
4. Рассчитайте количество воды, которое может быть задержано для каждого стержня, как минимальную разницу между максимальными высотами слева и справа и текущим значением высоты. Просуммируйте эти значения, чтобы получить общее количество задержанной воды.

СЛОЖНОСТЬ: O(n) время, O(n) память

```python
def trap(height):
    n = len(height)
    left = [height[0]] * n
    right = [height[-1]] * n
    for i in range(1, n):
        left[i] = max(left[i - 1], height[i])
        right[n - i - 1] = max(right[n - i], height[n - i - 1])
    return sum(min(l, r) - h for l, r, h in zip(left, right, height))
```

---

### Максимальная длина подмассива из единиц после одного удаления

- ID: 21, slug: `longest-subarray-of-1s-after-deleting-one-element`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Medium**
- Темы: Массивы, Два указателя, Динамическое программирование
- Страница сайта: [https://botayinterview.site/algorithm/longest-subarray-of-1s-after-deleting-one-element](https://botayinterview.site/algorithm/longest-subarray-of-1s-after-deleting-one-element)
- Публичный detail API: `GET /api/algorithms/public/21` → **HTTP 200**, 2915 байт
- LeetCode: [1493. Longest Subarray of 1's After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/) — **точное**

**Условие**

Дан массив бинарных чисел `nums`. Необходимо удалить один элемент из массива.

Верните размер наибольшего непустого подмассива, содержащего только единицы, в результирующем массиве. Если такого подмассива нет, верните 0.

**Примеры**:

Пример 1:

Вход: `nums = [1,1,0,1]`
Выход: `3`
Объяснение: После удаления числа в позиции 2, `[1,1,1]` содержит 3 числа со значением 1.

Example 2:

Вход: `nums = [0,1,1,1,0,1,1,0,1]`
Выход: `5`
Объяснение: После удаления числа в позиции 4, `[0,1,1,1,1,1,0,1]` наибольший подмассив со значением 1 равен `[1,1,1,1,1]`.

Example 3:

Вход: `nums = [1,1,1]`
Выход: `2`
Объяснение: Вы должны удалить один элемент.

**Открытые тесты**

Тест 1:

```text
Вход:
nums = [1,1,1]
Выход:
2
```

**Starter code**

````python
def longestSubarray(nums):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

ИДЕЯ: Нам нужно найти наибольший подмассив из единиц после удаления одного элемента.

ШАГИ:
1. Создаем два массива `left` и `right`, где `left[i]` и `right[i]` хранят количество единиц слева и справа от позиции `i` соответственно.
2. Заполняем массивы `left` и `right` в соответствии с входным массивом `nums`.
3. Ищем максимум суммы `left[i]` и `right[i + 1]` для всех `i`.

СЛОЖНОСТЬ: $O(n)$ время, $O(n)$ память

```python
def longestSubarray(nums):
    n = len(nums)
    left = [0] * (n + 1)
    right = [0] * (n + 1)
    for i, x in enumerate(nums, 1):
        if x:
            left[i] = left[i - 1] + 1
    for i in range(n - 1, -1, -1):
        if nums[i]:
            right[i] = right[i + 1] + 1
    return max(left[i] + right[i + 1] for i in range(n))
```

---

### Максимальная длина последовательности единиц

- ID: 18, slug: `max-consecutive-ones`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Easy**
- Темы: Массивы
- Страница сайта: [https://botayinterview.site/algorithm/max-consecutive-ones](https://botayinterview.site/algorithm/max-consecutive-ones)
- Публичный detail API: `GET /api/algorithms/public/18` → **HTTP 200**, 3260 байт
- LeetCode: [485. Max Consecutive Ones](https://leetcode.com/problems/max-consecutive-ones/) — **точное**

**Условие**

**Описание:** 
Дан массив из бинарных чисел (`0` и `1`). Верните максимальное количество последовательных единиц в массиве. 


**Пример 1:** 
Вход: `nums = [1,1,0,1,1,1]` 
Выход: `3` 
Объяснение: Первые два или последние три элемента являются последовательными единицами. Максимальное количество последовательных единиц равно `3`. 

**Пример 2:** 
Вход: `nums = [1,0,1,1,0,1]` 
Выход: `2` 

**Ограничения:** `1 <= nums.length <= 10^5`, `nums[i]` — либо `0`, либо `1`.

**Открытые тесты**

Тест 1:

```text
Вход:
nums = [0, 1, 1, 1, 1, 0]
Выход:
4
```

Тест 2:

```text
Вход:
nums = [0]
Выход:
0
```

Тест 3:

```text
Вход:
nums = [1, 0, 1, 0, 1, 0, 1]
Выход:
1
```

**Starter code**

````python
def findMaxConsecutiveOnes(nums: List[int]) -> int:
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**1. ИДЕЯ:** 
Мы проходим по массиву и поддерживаем две переменные: `ans` (для хранения максимального количества последовательных единиц) и `cnt` (для хранения текущего количества последовательных единиц). 
**2. ШАГИ:** 
**Шаг 1:** Инициализируем `ans` и `cnt` нулями. 
**Шаг 2:** Проходим по массиву. Если текущий элемент равен `1`, увеличиваем `cnt` на `1` и обновляем `ans`, если `cnt` больше текущего `ans`. 
**Шаг 3:** Если текущий элемент равен `0`, сбрасываем `cnt` до `0`. 

**3. ПРИМЕР:** Для массива `[1,1,0,1,1,1]` мы получаем: 
Инициализация: `ans = 0`, `cnt = 0`; 
Первый элемент (`1`): `cnt = 1`, `ans = 1`; 
Второй элемент (`1`): `cnt = 2`, `ans = 2`; 
Третий элемент (`0`): `cnt = 0`; 
Четвертый элемент (`1`): `cnt = 1`; 
Пятый элемент (`1`): `cnt = 2`; 
Шестой элемент (`1`): `cnt = 3`, `ans = 3`. 

**4. СЛОЖНОСТЬ:** 
`O(n)` время, `O(1)` память, где `n` — длина входного массива. 

**5. КОД:** 
```python
def findMaxConsecutiveOnes(nums: List[int]) -> int: 
    ans = cnt = 0 
    for x in nums: 
        if x: 
            cnt += 1 
            ans = max(ans, cnt) 
        else: 
            cnt = 0
    return ans
```

---

### Переместить нули в конец

- ID: 4, slug: `move-zeroes`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Easy**
- Темы: Два указателя, Массивы
- Страница сайта: [https://botayinterview.site/algorithm/move-zeroes](https://botayinterview.site/algorithm/move-zeroes)
- Публичный detail API: `GET /api/algorithms/public/4` → **HTTP 200**, 6885 байт
- LeetCode: [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/) — **точное**

**Условие**

Дан целочисленный массив `nums`. Переместите все нули в конец массива, сохраняя относительный порядок ненулевых элементов.

**Важно**: необходимо изменить исходный массив (in-place), не создавая его копии.

**Пример 1**:

Входные данные: `nums = [0,1,0,3,12]`
Выходные данные: `[1,3,12,0,0]`

**Пример 2**:

Входные данные: `nums = [0]`
Выходные данные: `[0]`

**Открытые тесты**

Тест 1:

```text
Вход:
nums = [0, 1, 0, 3, 12]
Выход:
[1, 3, 12, 0, 0]
```

Тест 2:

```text
Вход:
nums = [0, 0, 0, 1, 2]
Выход:
[1, 2, 0, 0, 0]
```

**Starter code**

````python
def moveZeroes(nums: list) -> list:
    pass
````

````javascript
function solution() {
    
}
````

**Решение из публичного API**

Мы будем использовать подход с двумя указателями:

1. **Указатель для записи (write_pointer)** - указывает, куда нужно записать следующий ненулевой элемент
2. **Указатель для чтения (read_pointer)** - проходит по всем элементам массива

**Шаги алгоритма:**
1. Инициализируем `write_pointer = 0`
2. Проходим по массиву с помощью `read_pointer`
3. Если текущий элемент не равен нулю, записываем его на позицию `write_pointer` и увеличиваем `write_pointer`
4. После прохода по всем элементам, заполняем оставшуюся часть массива нулями

**Пример работы**

Рассмотрим массив `nums = [0, 1, 0, 3, 12]`:

**Шаг 1:** `read_pointer = 0`, `write_pointer = 0`
- `nums[0] = 0` → пропускаем

**Шаг 2:** `read_pointer = 1`, `write_pointer = 0`
- `nums[1] = 1` ≠ 0 → `nums[0] = 1`, `write_pointer = 1`

**Шаг 3:** `read_pointer = 2`, `write_pointer = 1`
- `nums[2] = 0` → пропускаем

**Шаг 4:** `read_pointer = 3`, `write_pointer = 1`
- `nums[3] = 3` ≠ 0 → `nums[1] = 3`, `write_pointer = 2`

**Шаг 5:** `read_pointer = 4`, `write_pointer = 2`
- `nums[4] = 12` ≠ 0 → `nums[2] = 12`, `write_pointer = 3`

**Шаг 6:** Заполняем оставшиеся позиции нулями:
- `nums[3] = 0`, `nums[4] = 0`

Результат: `[1, 3, 12, 0, 0]`

**Реализация на Python**

```python
def moveZeroes(nums):
    """
    Перемещает все нули в конец массива, сохраняя порядок ненулевых элементов.
    Изменяет исходный массив in-place.
    
    Args:
        nums: List[int] - исходный массив чисел
    """
    # Указатель для записи ненулевых элементов
    write_pointer = 0
    
    # Первый проход: перемещаем все ненулевые элементы в начало
    for read_pointer in range(len(nums)):
        if nums[read_pointer] != 0:
            nums[write_pointer] = nums[read_pointer]
            write_pointer += 1
    
    # Второй проход: заполняем оставшуюся часть нулями
    for i in range(write_pointer, len(nums)):
        nums[i] = 0
    return nums
```

```python
# Примеры использования
if __name__ == "__main__":
    # Пример 1
    nums1 = [0, 1, 0, 3, 12]
    print(f"Исходный массив: {nums1}")
    moveZeroes(nums1)
    print(f"После перемещения нулей: {nums1}")
    print()
    
    # Пример 2
    nums2 = [0]
    print(f"Исходный массив: {nums2}")
    moveZeroes(nums2)
    print(f"После перемещения нулей: {nums2}")
    print()
    
    # Пример 3
    nums3 = [1, 2, 3, 4, 5]
    print(f"Исходный массив: {nums3}")
    moveZeroes(nums3)
    print(f"После перемещения нулей: {nums3}")
    print()
    
    # Пример 4
    nums4 = [0, 0, 0, 1, 2, 3]
    print(f"Исходный массив: {nums4}")
    moveZeroes(nums4)
    print(f"После перемещения нулей: {nums4}")
```

**Альтернативная оптимизированная реализация**

```python
def move_zeroes_optimized(nums):
    """
    Оптимизированная версия с одним проходом и обменом элементов.
    """
    # Указатель для записи ненулевых элементов
    write_pointer = 0
    
    for read_pointer in range(len(nums)):
        if nums[read_pointer] != 0:
            # Меняем местами текущий элемент и элемент на позиции write_pointer
            nums[write_pointer], nums[read_pointer] = nums[read_pointer], nums[write_pointer]
            write_pointer += 1
```

**Анализ сложности**

**Временная сложность:** O(n), где n - длина массива
- Мы проходим по массиву один или два раза, что дает линейную сложность

**Пространственная сложность:** O(1)
- Мы используем только константное количество дополнительной памяти

**Ключевые моменты**

1. Алгоритм работает за линейное время
2. Не требует дополнительной памяти (in-place)
3. Сохраняет относительный порядок ненулевых элементов
4. Работает корректно для всех краевых случаев:
   - Массив из одного нуля
   - Массив без нулей
   - Массив из одних нулей
   - Пустой массив

---

### Покупка и продажа акций

- ID: 41, slug: `best-time-to-buy-sell-stock`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Medium**
- Темы: Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/best-time-to-buy-sell-stock](https://botayinterview.site/algorithm/best-time-to-buy-sell-stock)
- Публичный detail API: `GET /api/algorithms/public/41` → **HTTP 200**, 2690 байт
- LeetCode: [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) — **точное**

**Условие**

Вам дан массив `prices`, где `prices[i]` - цена акций в $i$-й день.

Вы хотите максимизировать свою прибыль, выбрав один день для покупки акций и другой день в будущем для их продажи.

Верните максимальную прибыль, которую вы можете получить от этой транзакции. Если вы не можете получить прибыль, верните `0`.

**Открытые тесты**

Тест 1:

```text
Вход:
prices = [7,1,5,3,6,4]
Выход:
5
Пояснение:
Купить на 2-й день (цена = `1`) и продать на 5-й день (цена = `6`), прибыль = `6-1 = 5`.
Обратите внимание, что покупка на 2-й день и продажа на 1-й день не разрешены, потому что вы должны купить до продажи
```

Тест 2:

```text
Вход:
prices = [7,6,4,3,1]
Выход:
0
Пояснение:
В этом случае не проводятся транзакции, и максимальная прибыль = `0`
```

**Starter code**

````python
def maxProfit(prices):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

ИДЕЯ: Мы хотим найти максимальную разницу между двумя ценами, где цена покупки меньше цены продажи.

ШАГИ:
1. Инициализируйте переменную `mi` как положительную бесконечность и `ans` как `0`.
2. Переберите массив цен.
3. Для каждой цены обновите `ans` как максимальную из текущей `ans` и разницы между текущей ценой и `mi`.
4. Обновите `mi` как минимальную из текущей `mi` и текущей цены.
5. Верните `ans` как максимальную прибыль.

СЛОЖНОСТЬ: O(n) время, O(1) память

```python
def maxProfit(prices):
    ans, mi = 0, float('inf')
    for v in prices:
        ans = max(ans, v - mi)
        mi = min(mi, v)
    return ans
```

---

### Произведение двух RLE-массивов

- ID: 17, slug: `product-of-two-run-length-encoded-arrays`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Medium**
- Темы: Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/product-of-two-run-length-encoded-arrays](https://botayinterview.site/algorithm/product-of-two-run-length-encoded-arrays)
- Публичный detail API: `GET /api/algorithms/public/17` → **HTTP 200**, 4042 байт
- LeetCode: [1868. Product of Two Run-Length Encoded Arrays](https://leetcode.com/problems/product-of-two-run-length-encoded-arrays/) — **точное**

**Условие**

**Кодирование длин серий (Run-length encoding)** — это алгоритм сжатия, который позволяет представить массив целых чисел `nums` с множеством сегментов из **последовательно повторяющихся** чисел в виде (обычно меньшего) двумерного массива `encoded`. Каждая запись `encoded[i] = [val_i, freq_i]` описывает `i`-й сегмент повторяющихся чисел в `nums`, где `val_i` — это значение, которое повторяется `freq_i` раз.

* Например, `nums = [1,1,1,2,2,2,2,2]` представляется как массив `encoded = [[1,3],[2,5]]`. Это можно прочитать как «три `1`, за которыми следуют пять `2`».

**Произведение** двух массивов, кодированных по длине серий, `encoded1` и `encoded2`, может быть вычислено с помощью следующих шагов:

1. **Разверните** оба массива `encoded1` и `encoded2` в полные массивы `nums1` и `nums2` соответственно.
2. Создайте новый массив `prodNums` длиной `nums1.length` и установите `prodNums[i] = nums1[i] * nums2[i]`.
3. **Сожмите** `prodNums` обратно в массив формата кодирования длин серий и верните его.

Вам даны два массива `encoded1` и `encoded2`, представляющие полные массивы `nums1` и `nums2` соответственно. Оба массива `nums1` и `nums2` имеют **одинаковую длину**. Каждая запись `encoded1[i] = [val_i, freq_i]` описывает `i`-й сегмент `nums1`, а каждая запись `encoded2[j] = [val_j, freq_j]` описывает `j`-й сегмент `nums2`.

Верните **произведение** `encoded1` и `encoded2`.

**Примечание:** Сжатие должно быть выполнено таким образом, чтобы результирующий массив имел **минимально возможную** длину.

**Открытые тесты**

Тест 1:

```text
Вход:
encoded1 = [[2,5]]
encoded2 = [[3,2],[1,3]]
Выход:
[[6,2],[2,3]]
```

Тест 2:

```text
Вход:
encoded1 = [[1,2],[2,2]]
encoded2 = [[4,2],[2,2]]
Выход:
[[4,4]]
```

Тест 3:

```text
Вход:
encoded1 = [[5,10]]
encoded2 = [[0,5],[3,5]]
Выход:
[[0,5],[15,5]]
```

Тест 4:

```text
Вход:
encoded1 = [[1,1],[2,1],[1,1],[2,1]]
encoded2 = [[2,1],[1,1],[2,1],[1,1]]
Выход:
[[2,4]]
```

**Starter code**

````python
def findRLEArray(
    encoded1: List[List[int]], encoded2: List[List[int]]
) -> List[List[int]]:
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

```python
def findRLEArray(
    encoded1: List[List[int]], encoded2: List[List[int]]
) -> List[List[int]]:
    ans = []
    j = 0
    for vi, fi in encoded1:
        while fi:
            f = min(fi, encoded2[j][1])
            v = vi * encoded2[j][0]
            if ans and ans[-1][0] == v:
                ans[-1][1] += f
            else:
                ans.append([v, f])
            fi -= f
            encoded2[j][1] -= f
            if encoded2[j][1] == 0:
                j += 1
    return ans
```

---

### Скалярное произведение сжатых векторов

- ID: 68, slug: `dot-product-of-run-length-encoded-vectors`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Medium**
- Темы: Массив, Два указателя, Run-length encoding
- Страница сайта: [https://botayinterview.site/algorithm/dot-product-of-run-length-encoded-vectors](https://botayinterview.site/algorithm/dot-product-of-run-length-encoded-vectors)
- Публичный detail API: `GET /api/algorithms/public/68` → **HTTP 200**, 4374 байт
- LeetCode: [1868. Product of Two Run-Length Encoded Arrays](https://leetcode.com/problems/product-of-two-run-length-encoded-arrays/) — **близкая вариация, не точное совпадение**

**Условие**

# Скалярное произведение сжатых векторов

Даны два вектора целых чисел одинаковой длины. Каждый вектор задан в **сжатом формате** — списком пар `[value, count]`, где `value` — значение подряд идущих элементов, а `count` — их количество.

Например, вектор `[4, 4, 5]` записывается как `[[4, 2], [5, 1]]`. Сжатие сохраняет исходный порядок элементов.

Верните скалярное произведение исходных векторов:

`a[0] * b[0] + a[1] * b[1] + ... + a[n - 1] * b[n - 1]`.

Не разворачивайте сжатые векторы целиком: решение должно работать за время, пропорциональное числу пар.

**Открытые тесты**

Тест 1:

```text
Вход:
vec1 = [[1,3]]
vec2 = [[1,2],[10,1]]
Выход:
12
Пояснение:
Векторы равны [1, 1, 1] и [1, 1, 10], поэтому 1 + 1 + 10 = 12.
```

Тест 2:

```text
Вход:
vec1 = [[4,2],[3,2],[5,2]]
vec2 = [[1,1],[2,3],[4,2]]
Выход:
64
Пояснение:
Исходные векторы: [4, 4, 3, 3, 5, 5] и [1, 2, 2, 2, 4, 4].
```

Тест 3:

```text
Вход:
vec1 = [[2,1],[-3,2],[4,1]]
vec2 = [[5,2],[-1,2]]
Выход:
-6
Пояснение:
Проверка отрицательных значений: 2·5 + (-3)·5 + (-3)·(-1) + 4·(-1) = -6.
```

**Starter code**

````python
def dotProduct(vec1, vec2):
    pass
````

````javascript
function dotProduct(vec1, vec2) {
    
}
````

**Решение из публичного API**

ИДЕЯ: идти по обоим сжатым векторам двумя указателями. Для текущих пар `[value1, count1]` и `[value2, count2]` одинаковое число позиций равно `min(count1, count2)`. Их вклад в ответ — `value1 * value2 * min(count1, count2)`. Затем уменьшаем оставшиеся количества и переходим к следующей паре там, где количество закончилось.

ШАГИ:
1. Поставить указатели на первые пары обоих векторов.
2. Взять количество совпадающих позиций `used = min(rem1, rem2)`.
3. Добавить `value1 * value2 * used` к ответу.
4. Уменьшить остатки текущих серий на `used`.
5. Если остаток серии стал нулём, сдвинуть соответствующий указатель.

СЛОЖНОСТЬ: O(m + n) времени, где m и n — числа пар в векторах; O(1) дополнительной памяти.

```python
def dotProduct(vec1, vec2):
    i = j = 0
    rem1 = rem2 = 0
    result = 0

    while i < len(vec1) and j < len(vec2):
        if rem1 == 0:
            value1, rem1 = vec1[i]
        if rem2 == 0:
            value2, rem2 = vec2[j]

        used = min(rem1, rem2)
        result += value1 * value2 * used
        rem1 -= used
        rem2 -= used

        if rem1 == 0:
            i += 1
        if rem2 == 0:
            j += 1

    return result
```

---

### Слияние интервалов

- ID: 37, slug: `merge-x-intervals`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Easy**
- Темы: Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/merge-x-intervals](https://botayinterview.site/algorithm/merge-x-intervals)
- Публичный detail API: `GET /api/algorithms/public/37` → **HTTP 200**, 2059 байт
- LeetCode: [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/) — **точное**

**Условие**

Дан массив intervals, где intervals[i] = [starti, endi]. Объедините все перекрывающиеся интервалы и верните массив неперекрывающихся интервалов, покрывающих все интервалы из входных данных.

**Открытые тесты**

Тест 1:

```text
Вход:
intervals = [[1,3],[2,6],[8,10],[15,18]]
Выход:
[[1,6],[8,10],[15,18]]
```

Тест 2:

```text
Вход:
intervals = [[1,4],[4,5]]
Выход:
[[1,5]]
```

**Starter code**

````python
def merge(intervals):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

ИДЕЯ: Сортируем интервалы по началу, затем последовательно объединяем перекрывающиеся интервалы.

ШАГИ:
- Сортируем массив intervals по значению starti.
- Инициализируем результат первым интервалом.
- Для каждого следующего интервала проверяем, перекрывается ли он с последним добавленным в результат.
- Если перекрывается — расширяем конец последнего интервала. Если нет — добавляем новый интервал в результат.
```python
def merge(intervals):
    intervals.sort()
    res = [intervals[0]]
    for s, e in intervals[1:]:
        if s <= res[-1][1]:
            res[-1][1] = max(res[-1][1], e)
        else:
            res.append([s, e])
    return res
```

---

### Суммирование диапазонов

- ID: 28, slug: `sum-ranges`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Medium**
- Темы: Массивы, Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/sum-ranges](https://botayinterview.site/algorithm/sum-ranges)
- Публичный detail API: `GET /api/algorithms/public/28` → **HTTP 200**, 3004 байт
- LeetCode: [228. Summary Ranges](https://leetcode.com/problems/summary-ranges/) — **точное**

**Условие**

Вам дан отсортированный уникальный массив целых чисел `nums`.

Диапазон `[a, b]` - это набор всех целых чисел от `a` до `b` (включительно).

Верните **наименьший** отсортированный список диапазонов, которые покрывают все числа в массиве точно. То есть каждый элемент `nums` должен быть покрыт ровно одним из диапазонов, и не должно быть целого числа `x`, такого что `x` находится в одном из диапазонов, но не в `nums`.

Каждый диапазон `[a, b]` в списке должен быть выведен как:

"a->b" если `a != b`
"a" если `a == b`

**Открытые тесты**

Тест 1:

```text
Вход:
nums = [0,1,2,4,5,7]
Выход:
["0->2","4->5","7"]
```

Тест 2:

```text
Вход:
nums = [0,2,3,4,6,8,9]
Выход:
["0","2->4","6","8->9"]
```

**Starter code**

````python
def summaryRanges(nums):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

ИДЕЯ: Нам нужно найти все непрерывные диапазоны в отсортированном массиве `nums` и вывести их в заданном формате.

ШАГИ:
1. Инициализируем пустой список `ans` для хранения результатов и индекс `i` для прохода по `nums`.
2. Проходим по `nums`, проверяя для каждого элемента, является ли он началом нового диапазона или продолжением текущего.
3. Если элемент является началом нового диапазона, то мы нашли новый диапазон.
4. Когда мы нашли конец диапазона, добавляем его в `ans` в соответствующем формате.

СЛОЖНОСТЬ: O(n) время, O(n) память, где n - длина `nums`.

```python
def summaryRanges(nums):
    def f(i, j):
        return str(nums[i]) if i == j else f'{nums[i]}->{nums[j]}'
    i = 0
    n = len(nums)
    ans = []
    while i < n:
        j = i
        while j + 1 < n and nums[j + 1] == nums[j] + 1:
            j += 1
        ans.append(f(i, j))
        i = j + 1
    return ans
```

---

## Строки и скользящее окно

### Максимальная длина подстроки из двух уникальных символов

- ID: 42, slug: `max-len-substrin-two-symbols`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Easy**
- Темы: Строки
- Страница сайта: [https://botayinterview.site/algorithm/max-len-substrin-two-symbols](https://botayinterview.site/algorithm/max-len-substrin-two-symbols)
- Публичный detail API: `GET /api/algorithms/public/42` → **HTTP 200**, 2194 байт
- LeetCode: [159. Longest Substring with At Most Two Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-two-distinct-characters/) — **точное**

**Условие**

Дана строка `s`.

Необходимо найти максимальную длину непрерывной подстроки, в которой встречается не более двух различных символов.

**Открытые тесты**

Тест 1:

```text
Вход:
s = "eceba"
Выход:
3
```

Тест 2:

```text
Вход:
s = "ccaabbb"
Выход:
5
```

**Starter code**

````python
def longest_two_unique(s):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**ИДЕЯ:**  
Используем два указателя и поддерживаем текущее окно, в котором не больше двух уникальных символов.

**ШАГИ:**
1. Заводим левую границу окна `left`.
2. Идем правой границей `right` по строке.
3. Храним частоты символов внутри текущего окна.
4. Если уникальных символов стало больше двух, двигаем `left`, уменьшая частоты.
5. После каждого шага обновляем максимальную длину окна.

**СЛОЖНОСТЬ:**  
O(n) время, O(1) память

```python
def longest_two_unique(s):
    left = 0
    freq = {}
    best = 0

    for right, ch in enumerate(s):
        freq[ch] = freq.get(ch, 0) + 1

        while len(freq) > 2:
            left_ch = s[left]
            freq[left_ch] -= 1

            if freq[left_ch] == 0:
                del freq[left_ch]

            left += 1

        best = max(best, right - left + 1)

    return best
```

---

### Максимальное количество подряд идущих одинаковых символов

- ID: 40, slug: `max-sequential-same-synbols`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Easy**
- Темы: Строки
- Страница сайта: [https://botayinterview.site/algorithm/max-sequential-same-synbols](https://botayinterview.site/algorithm/max-sequential-same-synbols)
- Публичный detail API: `GET /api/algorithms/public/40` → **HTTP 200**, 2199 байт
- LeetCode: [1446. Consecutive Characters](https://leetcode.com/problems/consecutive-characters/) — **точное**

**Условие**

Дана строка `s`.

Необходимо найти максимальную длину непрерывной последовательности, состоящей из одинаковых символов.

**Открытые тесты**

Тест 1:

```text
Вход:
s = "aaabbc"
Выход:
3
```

Тест 2:

```text
Вход:
s = "abbcccdddd
Выход:
4
```

Тест 3:

```text
Вход:
s = "abc"
Выход:
1
```

Тест 4:

```text
Вход:
s = ""
Выход:
0
```

**Starter code**

````python
def max_consecutive_chars(s):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**ИДЕЯ:**  
Идем по строке и считаем длину текущей последовательности одинаковых символов.

**ШАГИ:**
1. Если строка пустая, возвращаем `0`.
2. Заводим переменные `current` и `best`.
3. Проходим по строке со второго символа.
4. Если текущий символ равен предыдущему, увеличиваем `current`.
5. Иначе начинаем новую последовательность длины `1`.
6. После каждого шага обновляем максимум `best`.

**СЛОЖНОСТЬ:**  
O(n) время, O(1) память

```python
def max_consecutive_chars(s):
    if not s:
        return 0

    current = 1
    best = 1

    for i in range(1, len(s)):
        if s[i] == s[i - 1]:
            current += 1
        else:
            current = 1

        best = max(best, current)

    return best
```

---

### Минимальная подстрока окна

- ID: 26, slug: `minimal-window-substring`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Hard**
- Темы: Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/minimal-window-substring](https://botayinterview.site/algorithm/minimal-window-substring)
- Публичный detail API: `GET /api/algorithms/public/26` → **HTTP 200**, 2417 байт
- LeetCode: [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) — **точное**

**Условие**

Даны две строки `s` и `t` длин `m` и `n` соответственно. Верните минимальную подстроку окна `s` такую, что каждый символ в `t` (включая дубликаты) включен в окно. Если такой подстроки нет, верните пустую строку `"".`

**Открытые тесты**

Тест 1:

```text
Вход:
s = "acbbaca"
t = "aba"
Выход:
"baca"
```

Тест 2:

```text
Вход:
s = "aabbcc"
t = "abc"
Выход:
"abbc"
```

**Starter code**

````python
def minWindow(s, t):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

Используется подход скользящего окна для поиска минимальной подстроки `s`, содержащей все символы `t`.

ШАГИ:
1. Создаём словарь `need` для хранения частоты символов в строке `t`.
2. Создаём словарь `window` для хранения частоты символов в текущем окне.
3. Инициализируем переменные `cnt`, `l`, `k` и `mi`.
4. Перебираем строку `s` и расширяем окно вправо.
5. Если в окне есть все символы `t`, то сужаем окно влево.

СЛОЖНОСТЬ: O(m + n) время, O(m + n) память

```python
def minWindow(s, t):
    need = Counter(t)
    window = Counter()
    cnt = l = 0
    k, mi = -1, float('inf')
    for r, c in enumerate(s):
        window[c] += 1
        if need[c] >= window[c]:
            cnt += 1
        while cnt == len(t):
            if r - l + 1 < mi:
                mi = r - l + 1
                k = l
            if need[s[l]] >= window[s[l]]:
                cnt -= 1
            window[s[l]] -= 1
            l += 1
    return "" if k < 0 else s[k : k + mi]
```

---

### Самая длинная подстрока без повторяющихся символов

- ID: 10, slug: `longest-substring-without-repeating-characters`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Medium**
- Темы: Строки, Хэш-таблица, Скользящее окно
- Страница сайта: [https://botayinterview.site/algorithm/longest-substring-without-repeating-characters](https://botayinterview.site/algorithm/longest-substring-without-repeating-characters)
- Публичный detail API: `GET /api/algorithms/public/10` → **HTTP 200**, 6130 байт
- LeetCode: [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) — **точное**

**Условие**

**Наибольшая подстрока без повторяющихся символов**

Дана строка `s`. Найдите длину **наибольшей подстроки**, не содержащей повторяющихся символов.

**Пример 1**:
**Ввод:** s = "abcabcbb"
**Вывод:** 3
**Объяснение:** Ответ — "abc", длина равна 3.

**Пример 2**:
**Ввод:** s = "bbbbb"
**Вывод:** 1
**Объяснение:** Ответ — "b", длина равна 1.

**Пример 3**:
**Ввод:** s = "pwwkew"
**Вывод:** 3
**Объяснение:** Ответ — "wke", длина равна 3.
Заметим, что ответ должен быть подстрокой. "pwke" — это подпоследовательность, а не подстрока.

**Открытые тесты**

Тест 1:

```text
Вход:
s = "pwwkew"
Выход:
3
```

Тест 2:

```text
Вход:
s = ""
Выход:
0
```

Тест 3:

```text
Вход:
s = "dvdf"
Выход:
3
```

**Starter code**

````python
def lengthOfLongestSubstring(s: str) -> int:
    pass
````

````javascript
function lengthOfLongestSubstring(s) {
    
}
````

**Решение из публичного API**

**Подход 1**: Брутфорс

Проверить все подстроки и найти самую длинную без повторений.

**Сложность:** $O(n^3)$ - неоптимально


**Подход 2**: Скользящее окно (Sliding Window)

Используем технику **скользящего окна** с двумя указателями.

**Идея:**
- Поддерживаем окно `[left, right]` без повторяющихся символов
- Расширяем окно вправо, пока не встретим повтор
- При повторе сдвигаем `left` до тех пор, пока повтор не исчезнет
- Отслеживаем максимальную длину окна

**Структура данных:**
- Hash Set для быстрой проверки наличия символа в окне
- Или Hash Map для хранения последней позиции каждого символа

**Алгоритм (с Hash Map):**

```python
def lengthOfLongestSubstring(s: str) -> int:
    char_index = {}  # символ -> последний индекс
    max_length = 0
    left = 0

    for right, char in enumerate(s):
        # Если символ уже есть в окне, сдвигаем left
        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1

        # Обновляем позицию символа
        char_index[char] = right

        # Обновляем максимальную длину
        max_length = max(max_length, right - left + 1)

    return max_length
```

**Сложность:**
- Временная: $O(n)$ - один проход
- Пространственная: $O(\min(n, m))$, где $m$ - размер алфавита

```python
s = a b c a b c b b
    0 1 2 3 4 5 6 7

Итерация 0 (char='a'): [a]               длина=1
Итерация 1 (char='b'): [a b]             длина=2
Итерация 2 (char='c'): [a b c]           длина=3 ← max
Итерация 3 (char='a'):   [b c a]         длина=3
Итерация 4 (char='b'):     [c a b]       длина=3
Итерация 5 (char='c'):       [a b c]     длина=3
Итерация 6 (char='b'):           [c b]   длина=2
Итерация 7 (char='b'):             [b]   длина=1

Каждый раз, когда встречается дубликат:
1. Сдвигаем левую границу за предыдущее вхождение
2. Обновляем позицию символа в словаре
3. Вычисляем текущую длину окна

Наибольшая длина: 3
```

Ответ: **3** (подстрока "abc")

**Ключевая идея:**

Когда встречаем повторяющийся символ, не нужно сдвигать `left` по одному - можно сразу прыгнуть на позицию после предыдущего вхождения этого символа.

**Альтернативная реализация** (с Set):

```python
def lengthOfLongestSubstring(s: str) -> int:
    chars = set()
    max_length = 0
    left = 0

    for right in range(len(s)):
        # Сдвигаем left, пока есть дубликаты
        while s[right] in chars:
            chars.remove(s[left])
            left += 1

        chars.add(s[right])
        max_length = max(max_length, right - left + 1)

    return max_length
```

Обе реализации работают за $O(n)$, но версия с HashMap обычно быстрее на практике.

---

### Сжатие последовательных пробелов

- ID: 69, slug: `compress-consecutive-spaces`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Medium**
- Темы: Строки, Массивы, Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/compress-consecutive-spaces](https://botayinterview.site/algorithm/compress-consecutive-spaces)
- Публичный detail API: `GET /api/algorithms/public/69` → **HTTP 200**, 4325 байт
- LeetCode: точное соответствие не подтверждено

**Условие**

# Сжатие последовательных пробелов

Вам дана строка, представленная в виде списка, где каждый элемент списка соответствует одному символу строки. Ваша задача — сжать все группы последовательных пробелов до одного пробела и вернуть результат в виде списка символов.

**Пример:**
Вход: ['h', 'e', 'l', 'l', 'o', ' ', ' ', ' ', ' ', ' ', 'w', 'o', 'r', 'l', 'd', ' ', '!', ' ', '!', ' ']
Выход: ['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd', ' ', '!', ' ', '!', ' ']

**Открытые тесты**

Тест 1:

```text
Вход:
chars = ["h","e","l","l","o"," "," "," "," "," ","w","o","r","l","d"," ","!"," ","!"," "]
Выход:
["h","e","l","l","o"," ","w","o","r","l","d"," ","!"," ","!"," "]
Пояснение:
Сжатие нескольких последовательных пробелов между 'hello' и 'world', а также после 'world ! !'.
```

Тест 2:

```text
Вход:
chars = [" "," "," ","a"," "," ","b"," "," "," ","c"]
Выход:
[" ","a"," ","b"," ","c"]
Пояснение:
Сжатие пробелов в начале, между символами и в конце.
```

Тест 3:

```text
Вход:
chars = ["a","b","c"]
Выход:
["a","b","c"]
Пояснение:
Отсутствие пробелов в строке.
```

**Starter code**

````python
def compress_spaces(chars):
    pass
````

````javascript
function compress_spaces(chars) {
    
}
````

**Решение из публичного API**

ИДЕЯ: использовать два указателя для эффективного сжатия последовательных пробелов.

ШАГИ:
1. Инициализировать два указателя: один для текущей позиции в исходном списке (current), другой для позиции вставки в результирующий список (insert).
2. Проходить по списку chars с помощью указателя current.
3. Если текущий символ не является пробелом, вставлять его в результирующий список по позиции insert и увеличивать оба указателя.
4. Если текущий символ является пробелом, проверять, является ли он первым пробелом в группе. Если да, вставлять его в результирующий список и увеличивать insert.
5. Возвращать результирующий список.

СЛОЖНОСТЬ: O(n) время, O(n) память, где n — длина списка chars.

```python
def compress_spaces(chars):
    result = []
    insert = 0
    current = 0
    while current < len(chars):
        if chars[current] != ' ':
            result.insert(insert, chars[current])
            insert += 1
        elif insert == 0 or result[insert - 1] != ' ':
            result.insert(insert, chars[current])
            insert += 1
        current += 1
    return result
```

---

### Удаление лишних скобок

- ID: 16, slug: `remove-invalid-parentheses`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Hard**
- Темы: Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/remove-invalid-parentheses](https://botayinterview.site/algorithm/remove-invalid-parentheses)
- Публичный detail API: `GET /api/algorithms/public/16` → **HTTP 200**, 4197 байт
- LeetCode: [301. Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/) — **точное**

**Условие**

Дана строка s, содержащая скобки и буквы. Удалите минимальное количество неправильно расставленных скобок, чтобы исходная строка стала валидной.

Верните список всех уникальных строк, которые становятся валидными после удаления минимального количества скобок. Ответ можно вернуть в любом порядке.

**Пример 1**: 
Ввод: `s = "()())()"` 
Вывод: `["(())()","()()()"]`

**Пример 2**: 
Ввод: `s = "(a)())()"`
Вывод: `["(a())()","(a)()()"]`

**Пример 3**: 
Ввод: `s = ")("` 
Вывод: `[""]`

Ограничения:

1 <= `s.length` <= 25

s состоит из строчных латинских букв и скобок '(' и ')'.

В строке s будет не более 20 скобок.

**Открытые тесты**

Тест 1:

```text
Вход:
s = "abc"
Выход:
["abc"]
Пояснение:
Скобок нет, строка уже валидна
```

Тест 2:

```text
Вход:
s = "((()"
Выход:
["()"]
Пояснение:
Нужно удалить две лишние открывающие скобки
```

Тест 3:

```text
Вход:
s = "n)())()"
Выход:
["n()()"]
Пояснение:
Минимум удалений — 2 скобки.
```

**Starter code**

````python
def removeInvalidParentheses(s):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

Для решения задачи "минимального количества удалений" идеально подходит поиск в ширину (BFS). Мы проверяем все возможные варианты строки, удаляя по одной скобке за раз, пока не найдем первый уровень, на котором есть валидные строки.

```python
from collections import deque


def removeInvalidParentheses(s: str) -> list[str]:
    # Функция для проверки, является ли строка валидной
    def isValid(string):
        count = 0
        for char in string:
            if char == '(':
                count += 1
            elif char == ')':
                count -= 1
                if count < 0:
                    return False
        return count == 0

    # Очередь для BFS и множество для хранения посещенных строк
    queue = deque([s])
    visited = {s}
    result = []
    found = False

    while queue:
        current_str = queue.popleft()

        # Если строка валидна
        if isValid(current_str):
            result.append(current_str)
            found = True

        # Если мы уже нашли валидные строки на этом уровне, 
        # дальше удалять скобки (уходить вглубь) нет смысла
        if found:
            continue

        # Генерируем все возможные варианты, удаляя одну скобку
        for i in range(len(current_str)):
            if current_str[i] not in "()":
                continue
            
            # Создаем новую строку без текущего символа
            next_str = current_str[:i] + current_str[i+1:]
            
            if next_str not in visited:
                visited.add(next_str)
                queue.append(next_str)

    return result
```

---

## Хэш-таблицы

### Группировка анаграмм

- ID: 30, slug: `group-anagramm`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Medium**
- Темы: Массивы, Хэш-таблица
- Страница сайта: [https://botayinterview.site/algorithm/group-anagramm](https://botayinterview.site/algorithm/group-anagramm)
- Публичный detail API: `GET /api/algorithms/public/30` → **HTTP 200**, 1699 байт
- LeetCode: [49. Group Anagrams](https://leetcode.com/problems/group-anagrams/) — **точное**

**Условие**

Дан массив английских слов `arr`. 

Задача состоит в группировке строк, являющихся анаграммами. Анаграмма — это слово или фраза, образованная перестановкой букв другого слова, при этом все исходные буквы используются ровно один раз.

**Открытые тесты**

Тест 1:

```text
Вход:
arr = ["act", "god", "cat", "dog", "tac"]
Выход:
[["act", "cat", "tac"], ["god", "dog"]]
```

Тест 2:

```text
Вход:
arr = []
Выход:
[]
```

**Starter code**

````python
def group_anagrams(arr):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

```python
from typing import List

def group_anagrams(arr):
    group = {}
    
    for word in arr:
        key = "".join(sorted(word))
        
        # Если такого ключа еще нет, создаем пустой список
        if key not in group:
            group[key] = []
            
        # Добавляем именно исходное слово (word), а не ключ
        group[key].append(word)
    
    return list(group.values())
```

---

### Отражение относительно прямой

- ID: 5, slug: `line-reflection`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Easy**
- Темы: Математика, Хэш-таблица
- Страница сайта: [https://botayinterview.site/algorithm/line-reflection](https://botayinterview.site/algorithm/line-reflection)
- Публичный detail API: `GET /api/algorithms/public/5` → **HTTP 200**, 12441 байт
- LeetCode: [356. Line Reflection](https://leetcode.com/problems/line-reflection/) — **точное**

**Условие**

Даны `n` точек на 2D плоскости. Определите, существует ли вертикальная линия (параллельная оси Y), относительно которой данные точки симметричны.

**Другими словами**: Можно ли найти такую вертикальную линию `x = c`, чтобы каждая точка `(x, y)` имела свою "зеркальную" пару `(2c - x, y)` среди данных точек.

**Пример 1**:
Вход: `points = [[1,1],[-1,1]]`
Выход: `true`
Объяснение: Линия `x = 0` отражает точку `(1,1)` в `(-1,1)`

**Пример 2**:
Вход: `points = [[1,1],[-1,-1]]`
Выход: `false`
Объяснение: Нет такой вертикальной линии

**Открытые тесты**

Тест 1:

```text
Вход:
points = [[1, 1], [-1, 1]]
Выход:
true
```

Тест 2:

```text
Вход:
points = [[1, 1], [-1, -1]]
Выход:
false
```

**Starter code**

````python
def isReflected(points):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**Постановка задачи**

Даны точки на плоскости в виде `[[x1, y1], [x2, y2], ..., [xn, yn]]`. Нужно определить, существует ли вертикальная линия `x = c`, такая что для каждой точки `(x, y)` существует симметричная относительно этой линии точка `(2c - x, y)` среди данных точек.

**Ключевая идея решения**

1. **Симметричные точки должны иметь одинаковую Y-координату**
2. **Центральная линия `x = c` должна быть средней точкой между парой симметричных точек**
3. **Все точки можно отразить относительно предполагаемой линии и проверить, есть ли их отражения в исходном наборе**


**Основные шаги решения:**
1. Найти потенциальную линию симметрии как среднее значение минимальной и максимальной X-координат
2. Использовать множество (set) для хранения всех точек в строковом формате для быстрого поиска
3. Для каждой точки вычислять ее отражение относительно линии и проверять, есть ли оно в множестве
4. Учитывать, что точки на самой линии симметрии являются сами себе симметричными

**Пример работы**

**Пример 1**: `points = [[1,1],[-1,1]]`

**Шаг 1:** Находим min_x = -1, max_x = 1
**Шаг 2:** Вычисляем линию симметрии: `c = (min_x + max_x) / 2 = (-1 + 1) / 2 = 0`
**Шаг 3:** Проверяем каждую точку:
- Для точки `(1,1)` отражение: `(2*0 - 1, 1) = (-1, 1)` - есть в наборе
- Для точки `(-1,1)` отражение: `(2*0 - (-1), 1) = (1, 1)` - есть в наборе

**Результат:** `True` (линия x = 0 существует)

**Пример 2**: `points = [[1,1],[-1,-1]]`

**Шаг 1:** min_x = -1, max_x = 1
**Шаг 2:** `c = (-1 + 1) / 2 = 0`
**Шаг 3:** Проверяем:
- `(1,1)` → отражение `(-1, 1)` - нет в наборе
- `(-1,-1)` → отражение `(1, -1)` - нет в наборе

**Результат:** `False`


```python
def isReflected(points):
    """
    Проверяет, существует ли вертикальная линия, относительно которой точки симметричны.
    
    Args:
        points: List[List[int]] - список точек [x, y]
    
    Returns:
        bool: True если линия симметрии существует, иначе False
    """
    if not points:
        return True
    
    # Создаем множество для быстрого поиска точек
    point_set = set()
    min_x = float('inf')
    max_x = float('-inf')
    
    # Находим минимальный и максимальный x, сохраняем точки в множестве
    for x, y in points:
        point_set.add((x, y))
        min_x = min(min_x, x)
        max_x = max(max_x, x)
    
    # Вычисляем линию симметрии
    line_x = (min_x + max_x) / 2
    
    # Проверяем каждую точку
    for x, y in points:
        # Вычисляем отраженную точку
        reflected_x = 2 * line_x - x
        reflected_point = (reflected_x, y)
        
        # Если отраженной точки нет в множестве, симметрии нет
        if reflected_point not in point_set:
            return False
    
    return True
```

Примеры использования
```python
if __name__ == "__main__":
    # Пример 1: симметрия относительно x = 0
    points1 = [[1, 1], [-1, 1]]
    print(f"Точки: {points1}")
    print(f"Существует ли линия симметрии? {isReflected(points1)}")
    print(f"Линия симметрии: x = {(min(p[0] for p in points1) + max(p[0] for p in points1)) / 2}")
    print()
    
    # Пример 2: нет симметрии
    points2 = [[1, 1], [-1, -1]]
    print(f"Точки: {points2}")
    print(f"Существует ли линия симметрии? {isReflected(points2)}")
    print()
    
    # Пример 3: симметрия с несколькими точками
    points3 = [[0, 0], [1, 2], [-1, 2], [2, 3], [-2, 3]]
    print(f"Точки: {points3}")
    print(f"Существует ли линия симметрии? {isReflected(points3)}")
    print(f"Линия симметрии: x = {(min(p[0] for p in points3) + max(p[0] for p in points3)) / 2}")
    print()
    
    # Пример 4: все точки на линии симметрии
    points4 = [[0, 1], [0, 2], [0, 3]]
    print(f"Точки: {points4}")
    print(f"Существует ли линия симметрии? {isReflected(points4)}")
    print(f"Линия симметрии: x = {(min(p[0] for p in points4) + max(p[0] for p in points4)) / 2}")
    print()
    
    # Пример 5: точки с дубликатами
    points5 = [[1, 1], [-1, 1], [1, 1]]  # Дубликат (1,1)
    print(f"Точки: {points5}")
    print(f"Существует ли линия симметрии? {isReflected(points5)}")
    print()
```

**Альтернативная реализация с обработкой дубликатов**

```python
def is_reflected_with_duplicates(points):
    """
    Решение с учетом возможных дубликатов точек.
    """
    if not points:
        return True
    
    # Используем словарь для подсчета точек
    from collections import defaultdict
    point_count = defaultdict(int)
    
    min_x = float('inf')
    max_x = float('-inf')
    
    # Считаем точки и находим min/max x
    for x, y in points:
        point_count[(x, y)] += 1
        min_x = min(min_x, x)
        max_x = max(max_x, x)
    
    line_x = (min_x + max_x) / 2
    
    # Проверяем каждую уникальную точку
    for (x, y), count in point_count.items():
        reflected_x = 2 * line_x - x
        reflected_point = (reflected_x, y)
        
        # Если это точка на линии симметрии
        if x == reflected_x:
            continue
        
        # Проверяем, есть ли отраженная точка с тем же количеством
        if point_count[reflected_point] != count:
            return False
    
    return True
```

**Визуализация алгоритма**

```python
def visualize_reflection(points):
    """
    Визуализирует точки и потенциальную линию симметрии.
    """
    if not points:
        print("Нет точек для визуализации")
        return
    
    # Находим границы для визуализации
    all_x = [p[0] for p in points]
    all_y = [p[1] for p in points]
    
    min_x, max_x = min(all_x), max(all_x)
    min_y, max_y = min(all_y), max(all_y)
    
    # Вычисляем линию симметрии
    line_x = (min_x + max_x) / 2
    
    print(f"Линия симметрии: x = {line_x}")
    print("Точки и их отражения:")
    
    for x, y in points:
        reflected_x = 2 * line_x - x
        print(f"  ({x}, {y}) -> ({reflected_x}, {y})")
    
    # Проверяем симметрию
    if is_reflected(points):
        print("✓ Точки симметричны относительно линии")
    else:
        print("✗ Точки НЕ симметричны относительно линии")


# Пример визуализации
if __name__ == "__main__":
    print("=== Визуализация примера 1 ===")
    points = [[1, 1], [-1, 1], [2, 3], [-2, 3]]
    visualize_reflection(points)
```

**Анализ сложности**

**Временная сложность:** O(n)
- Мы проходим по всем точкам два раза: для создания множества и для проверки
- Операции с множеством (добавление и поиск) в среднем O(1)

**Пространственная сложность:** O(n)
- Мы храним все точки в множестве для быстрого поиска

**Особые случаи**

1. **Пустой список:** считается симметричным (возвращаем True)
2. **Одна точка:** всегда симметрична (любая вертикальная линия, проходящая через точку)
3. **Точки на одной вертикальной линии:** симметричны относительно себя
4. **Дубликаты точек:** должны иметь дубликаты симметричных точек
5. **Точки с одинаковыми координатами:** обрабатываются корректно

**Проверка симметрии вручную**

Чтобы проверить симметрию вручную:
1. Найдите среднюю точку по X между самой левой и самой правой точкой
2. Для каждой точки (x, y) найдите ее отражение (2c - x, y)
3. Убедитесь, что все отраженные точки присутствуют в исходном наборе

Этот алгоритм эффективно решает задачу проверки вертикальной симметрии точек на плоскости.

---

### Сумма двух чисел

- ID: 6, slug: `two-sum`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Easy**
- Темы: Массивы, Хэш-таблица
- Страница сайта: [https://botayinterview.site/algorithm/two-sum](https://botayinterview.site/algorithm/two-sum)
- Публичный detail API: `GET /api/algorithms/public/6` → **HTTP 200**, 4350 байт
- LeetCode: [1. Two Sum](https://leetcode.com/problems/two-sum/) — **точное**

**Условие**

**Two Sum**

Дан массив целых чисел `nums` и целое число `target`. Верните **индексы** двух чисел, сумма которых равна `target`.  
Можно считать, что для каждого входного набора существует **ровно одно решение**, и один и тот же элемент нельзя использовать дважды.

**Пример 1**:

**Вход:** `nums = [2,7,11,15], target = 9`
**Выход:** `[0,1]`
**Объяснение:** Потому, что nums[0] + nums[1] == 9, и мы возвращаем список индексов [0, 1].

**Пример 2**:

**Вход:** `nums = [3,2,4], target = 6`
**Выход:** `[1,2]`

**Пример 3**:

**Вход:** `nums = [3,3], target = 6`
**Выход:** `[0,1]`

**Открытые тесты**

Тест 1:

```text
Вход:
nums = [3,3]
target = 6
Выход:
[0,1]
```

Тест 2:

```text
Вход:
nums = [1,2,3,4,5]
target = 9
Выход:
[3,4]
```

**Starter code**

````python
def twoSum(nums: list[int], target: int) -> list[int]:
    pass
````

````javascript
function twoSum(nums, target) {
    
}
````

**Решение из публичного API**

**Решение задачи Two Sum**

**Подход 1: Брутфорс (наивный подход)**

Самое простое решение - перебрать все пары чисел и проверить, дают ли они нужную сумму.

**Сложность:**
- Временная: $O(n^2)$
- Пространственная: $O(1)$

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []
```

**Подход 2: Hash Map (оптимальное решение)**

Мы можем использовать хеш-таблицу для хранения чисел, которые мы уже видели. Для каждого числа `num` проверяем, существует ли в таблице число `target - num`.

**Алгоритм:**
1. Создаем пустую хеш-таблицу `seen`
2. Для каждого числа `num` на позиции `i`:
   - Вычисляем `complement = target - num`
   - Если `complement` есть в `seen`, возвращаем `[seen[complement], i]`
   - Иначе добавляем `num` в `seen` с индексом `i`

**Сложность:**
- Временная: $O(n)$ - один проход по массиву
- Пространственная: $O(n)$ - хеш-таблица

**Код:**

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    seen = {}  # число -> индекс

    for i, num in enumerate(nums):
        complement = target - num

        if complement in seen:
            return [seen[complement], i]

        seen[num] = i

    return []
```

**Пример работы:**

Для `nums = [2, 7, 11, 15]`, `target = 9`:

1. `i=0, num=2`: complement=7, seen={}, добавляем: seen={2: 0}
2. `i=1, num=7`: complement=2, 2 есть в seen, возвращаем [0, 1] ✓

**Почему это работает:**

Если пара `(i, j)` дает нужную сумму, то когда мы дойдем до второго элемента пары, первый уже будет в хеш-таблице.

---

## Связные списки

### Слияние двух отсортированных списков

- ID: 12, slug: `merge-two-sorted-lists`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Easy**
- Темы: Связный список
- Страница сайта: [https://botayinterview.site/algorithm/merge-two-sorted-lists](https://botayinterview.site/algorithm/merge-two-sorted-lists)
- Публичный detail API: `GET /api/algorithms/public/12` → **HTTP 200**, 4874 байт
- LeetCode: [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) — **точное**

**Условие**

Задача: Объединение двух отсортированных списков

Вам даны головы двух отсортированных связанных списков `list1` и `list2`. 
Объедините два списка в один отсортированный список. Список должен быть создан 
путем соединения узлов первых двух списков. Верните голову объединенного связанного списка.


**Пример 1:**
    * Вход: `list1 = [1,2,4]`, `list2 = [1,3,4]`
    * Выход: `[1,1,2,3,4,4]`

**Пример 2:**
    * Вход: `list1 = []`, `list2 = []`
    * Выход: `[]`

**Пример 3:**
    * Вход: `list1 = []`, `list2 = [0]`
    * Выход: `[0]`

**Ограничения**:
- Количество узлов в обоих списках находится в диапазоне [0, 50].
- -100 <= Node.val <= 100
- Оба списка list1 и list2 отсортированы в неубывающем порядке.

**Открытые тесты**

Тест 1:

```text
Вход:
list1 = []
list2 = [0]
Выход:
[0]
```

Тест 2:

```text
Вход:
list1 = [1,1,1]
list2 = [1,1,1]
Выход:
[1, 1, 1, 1, 1, 1]
```

Тест 3:

```text
Вход:
list1 = [10, 20, 30]
list2 = [5, 15, 25, 35]
Выход:
[5, 10, 15, 20, 25, 30, 35]
```

Тест 4:

```text
Вход:
list1 = [-1,0,2]
list2 = [-2,-1,0]
Выход:
[-2, -1, -1, 0, 0, 2]
```

**Starter code**

````python
# Определение одногосвязного списка.
class ListNode:
     def __init__(self, val=0, next=None):
         self.val = val
         self.next = next

# Конвертация обычного списка в связный список
def list_to_linked(arr):
    if not arr:
        return None
    head = ListNode(arr[0])
    current = head
    for val in arr[1:]:
        current.next = ListNode(val)
        current = current.next
    return head

# Конвертация связного списка обратно в обычный
def linked_to_list(head):
    result = []
    while head:
        result.append(head.val)
        head = head.next
    return result

def mergeTwoLists(list1: list, list2: list) -> ListNode:
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**ИДЕЯ**: Рекурсивное объединение двух отсортированных связанных списков.

**СЛОЖНОСТЬ**:
- Время: O(n + m) — проходим по каждому узлу обоих списков один раз.
- Память: O(n + m) — глубина стека рекурсии пропорциональна сумме длин списков.

```python
# Определение одногосвязного списка.
class ListNode:
     def __init__(self, val=0, next=None):
         self.val = val
         self.next = next
# Конвертация обычного списка в связный список
def list_to_linked(arr):
    if not arr:
        return None
    head = ListNode(arr[0])
    current = head
    for val in arr[1:]:
        current.next = ListNode(val)
        current = current.next
    return head

# Конвертация связного списка обратно в обычный
def linked_to_list(head):
    result = []
    while head:
        result.append(head.val)
        head = head.next
    return result

# Само решение задачи
def mergeTwoLists(list1: list, list2: list) -> ListNode:
    list1 = list_to_linked(list1)
    list2 = list_to_linked(list2) 
    dummy = ListNode()  # фиктивный узел для удобства
    current = dummy
    
    while list1 and list2:
        if list1.val <= list2.val:
            current.next = list1
            list1 = list1.next
        else:
            current.next = list2
            list2 = list2.next
        current = current.next
    
    # Добавляем оставшийся хвост
    current.next = list1 if list1 else list2
    
    return linked_to_list(dummy.next)
```

---

## Динамическое программирование и бинарный поиск

### Наибольшая возрастающая подпоследовательность

- ID: 15, slug: `longest-increasing-subsequence`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Easy**
- Темы: Два указателя
- Страница сайта: [https://botayinterview.site/algorithm/longest-increasing-subsequence](https://botayinterview.site/algorithm/longest-increasing-subsequence)
- Публичный detail API: `GET /api/algorithms/public/15` → **HTTP 200**, 5710 байт
- LeetCode: [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) — **точное**

**Условие**

Задача: Самая длинная возрастающая подпоследовательность

Дан целочисленный массив nums. Верните длину самой длинной строго возрастающей подпоследовательности.

**Пример 1**:
Ввод: `nums = [10, 9, 2, 5, 3, 7, 101, 18]`
Вывод: `4`
Объяснение: Самая длинная возрастающая подпоследовательность — `[2, 3, 7, 101]`, поэтому её длина равна `4`.

**Пример 2**:
Ввод: `nums = [0, 1, 0, 3, 2, 3]`
Вывод: `4`

Пример 3:
Ввод: `nums = [7, 7, 7, 7, 7, 7, 7]`
Вывод: `1`

Ограничения:
* 1 <= `nums.length` <= 2500
* -10^4 <= `nums[i]` <= 10^4

Дополнительно: сможете ли вы придумать алгоритм со сложностью по времени O(n log(n))?

**Открытые тесты**

Тест 1:

```text
Вход:
nums = [1, 2, 3, 4, 5]
Выход:
5
Пояснение:
Весь массив является возрастающей подпоследовательностью
```

Тест 2:

```text
Вход:
nums = [-5, -2, 0, -1, 4, 2]
Выход:
4
Пояснение:
Самая длинная возрастающая подпоследовательность — [-5, -2, 0, 4] или [-5, -2, 0, 2].
```

Тест 3:

```text
Вход:
nums = [10, 5, 8, 3, 9, 4, 12, 11]
Выход:
4
Пояснение:
Возможная подпоследовательность — [5, 8, 9, 12] или [3, 4, 12] (но она короче)
```

**Starter code**

````python
def lengthOfLIS(nums):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**Подход: Жадный алгоритм + Бинарный поиск ()**

Этот метод является наиболее эффективным. Мы создаем список `sub`, который будет хранить "лучшую" возрастающую подпоследовательность на текущий момент.

**Логика:**

1. Мы перебираем каждое число `x` из массива `nums`.
2. Если `x` больше последнего элемента в `sub`, мы просто добавляем его в конец. Это значит, мы нашли способ удлинить нашу последовательность.
3. Если `x` меньше или равно последнему элементу, мы находим в `sub` первый элемент, который больше или равен `x`, и заменяем его на `x`.
* **Зачем мы это делаем?** Замена не меняет длину `sub`, но она делает значения в списке меньше. Чем меньше числа в нашей последовательности, тем больше шансов, что в будущем мы встретим еще большие числа и сможем удлинить список.


4. В итоге длина списка `sub` и будет ответом.

**Сложность:**

* **Временная:** , так как мы проходим по массиву один раз (), и для каждого элемента выполняем бинарный поиск ().
* **Пространственная:**  для хранения списка `sub`.

```python

def lengthOfLIS(nums: list[int]) -> int:
    # sub будет хранить элементы нашей возрастающей последовательности
    sub = []
    
    for x in nums:
        # Используем бинарный поиск, чтобы найти место для x
        # В Python bisect_left делает это максимально быстро
        import bisect
        idx = bisect.bisect_left(sub, x)
        
        # Если x больше всех элементов в sub, добавляем его
        if idx == len(sub):
            sub.append(x)
        # Иначе заменяем найденный элемент на x, чтобы сделать sub "лучше"
        else:
            sub[idx] = x
    
    # Длина sub соответствует длине самой длинной подпоследовательности
    return len(sub)

```

**Пример работы:**
`nums = [10, 9, 2, 5, 3, 7]`

1. `x = 10`: `sub = [10]`
2. `x = 9`: заменяем 10 на 9. `sub = [9]` (длина та же, но 9 выгоднее)
3. `x = 2`: заменяем 9 на 2. `sub = [2]`
4. `x = 5`: добавляем 5. `sub = [2, 5]`
5. `x = 3`: заменяем 5 на 3. `sub = [2, 3]`
6. `x = 7`: добавляем 7. `sub = [2, 3, 7]`
**Ответ: 3**

Хотите, чтобы я добавил еще вариант решения через классическое динамическое программирование (через массив `dp`), если этот кажется слишком сложным?

---

## Деревья и графы

### Количество островов

- ID: 24, slug: `islands`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Hard**
- Темы: Графы
- Страница сайта: [https://botayinterview.site/algorithm/islands](https://botayinterview.site/algorithm/islands)
- Публичный detail API: `GET /api/algorithms/public/24` → **HTTP 200**, 2644 байт
- LeetCode: [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) — **точное**

**Условие**

Дан двумерный массив `grid` размером $m \times n$, заполненный символами '1' (суша) и '0' (вода). Необходимо найти количество островов.

Остров - это область, окружённая водой и образованная соединением соседних участков суши горизонтально или вертикально. Можно предположить, что все четыре края сетки окружены водой.

**Открытые тесты**

Тест 1:

```text
Вход:
grid = [["1","1","1","1","0"],["1","1","0","1","0"],["1","1","0","0","0"],["0","0","0","0","0"]]
Выход:
1
```

**Starter code**

````python
def numIslands(grid):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

ИДЕЯ: Используется алгоритм поиска в глубину (DFS) для обхода островов и их пометки как посещённых.

ШАГИ:
1. Инициализируем переменную `ans` для хранения количества островов.
2. Определяем функцию `dfs(i, j)` для поиска в глубину, которая помечает текущую ячейку как посещённую (`grid[i][j] = '0'`) и рекурсивно вызывает себя для всех соседних ячеек, содержащих сушу.
3. Итерируем по всем ячейкам сетки. Если ячейка содержит сушу (`grid[i][j] == '1'`), вызываем `dfs(i, j)` и увеличиваем `ans` на 1.
СЛОЖНОСТЬ: O(m x n) время, O(m x n) память

```python
def numIslands(grid):
    def dfs(i, j):
        grid[i][j] = '0'
        dirs = [(-1, 0), (1, 0), (0, -1), (0, 1)]
        for a, b in dirs:
            x, y = i + a, j + b
            if 0 <= x < len(grid) and 0 <= y < len(grid[0]) and grid[x][y] == '1':
                dfs(x, y)
    ans = 0
    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] == '1':
                dfs(i, j)
                ans += 1
    return ans
```

---

### Симметричное дерево

- ID: 19, slug: `symmetric-tree`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Hard**
- Темы: Деревья
- Страница сайта: [https://botayinterview.site/algorithm/symmetric-tree](https://botayinterview.site/algorithm/symmetric-tree)
- Публичный detail API: `GET /api/algorithms/public/19` → **HTTP 200**, 5306 байт
- LeetCode: [101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/) — **точное**

**Условие**

Дан корень бинарного дерева, проверьте, является ли оно зеркальным отражением самого себя (т.е. симметричным относительно своего центра).

**Пример 1:**

Вход: `root = [1,2,2,3,4,4,3]`, Выход: `true`

**Пример 2:**

Вход: `root = [1,2,2,null,3,null,3]`, Выход: `false`


**Дополнительная задача:** Можете ли вы решить эту задачу как рекурсивно, так и итеративно?

**Открытые тесты**

Тест 1:

```text
Вход:
root = [1, 2, 3, 4, 5, 6, 7]
Выход:
false
```

Тест 2:

```text
Вход:
root = [1, 2, 2, 3, None, None, 3]
Выход:
true
```

Тест 3:

```text
Вход:
root = [1, 2, 2, 3, 4, 4, 3, 5, 6, 7, 8, 8, 7, 6, 5]
Выход:
true
```

**Starter code**

````python
class TreeNode:
   def __init__(self, val=0, left=None, right=None):
       self.val = val
       self.left = left
       self.right = right

# Создание дерева из списка
def list_to_tree(arr):
    if not arr:
        return None
    
    root = TreeNode(arr[0])
    queue = [root]
    i = 1
    
    while queue and i < len(arr):
        node = queue.pop(0)
        
        if i < len(arr) and arr[i] is not None:
            node.left = TreeNode(arr[i])
            queue.append(node.left)
        i += 1
        
        if i < len(arr) and arr[i] is not None:
            node.right = TreeNode(arr[i])
            queue.append(node.right)
        i += 1
    
    return root

def isSymmetric(root):
    root = list_to_tree(root)
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**Проверка симметричности бинарного дерева**

**ИДЕЯ**:
Для проверки симметричности бинарного дерева можно использовать рекурсивный подход, сравнивая левое и правое поддеревья корня. При сравнении левое поддерево левого узла должно быть зеркально правому поддереву правого узла.

**ШАГИ**:
1. Если дерево пустое (`root` равен `None`), то оно симметрично.
2. Запускаем рекурсивную функцию сравнения для левого и правого поддеревьев корня.
3. В рекурсивной функции:
   - Если оба узла `None` → возвращаем `True`
   - Если один из узлов `None` или их значения не равны → возвращаем `False`
   - Рекурсивно проверяем: `left.left` с `right.right` И `left.right` с `right.left`
4. Возвращаем результат рекурсивной проверки.

**Сложность**:
- **Время:** $O(n)$, где `n` - количество узлов (посещаем каждый узел один раз)
- **Память:** $O(h)$, где `h` - высота дерева (глубина рекурсии)

**Код решения**:

```python
class TreeNode:
   def __init__(self, val=0, left=None, right=None):
       self.val = val
       self.left = left
       self.right = right

# Создание дерева из списка
def list_to_tree(arr):
    if not arr:
        return None
    
    root = TreeNode(arr[0])
    queue = [root]
    i = 1
    
    while queue and i < len(arr):
        node = queue.pop(0)
        
        if i < len(arr) and arr[i] is not None:
            node.left = TreeNode(arr[i])
            queue.append(node.left)
        i += 1
        
        if i < len(arr) and arr[i] is not None:
            node.right = TreeNode(arr[i])
            queue.append(node.right)
        i += 1
    
    return root

def isSymmetric(root):
    root = list_to_tree(root)
    def dfs(root1, root2):
        # Базовые случаи
        if root1 is None and root2 is None:
            return True
        if root1 is None or root2 is None or root1.val != root2.val:
            return False
        
        # Рекурсивная проверка зеркальных поддеревьев
        return dfs(root1.left, root2.right) and dfs(root1.right, root2.left)
    
    # Пустое дерево симметрично
    if root is None:
        return True
    
    # Сравниваем левое и правое поддеревья
    return dfs(root.left, root.right)
```

---

## Матрицы

### sort-the-matrix-diagonally

- ID: 27, slug: `Сортировка матрицы по диагоналям`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Hard**
- Темы: Массивы, Математика
- Страница сайта: [https://botayinterview.site/algorithm/%D0%A1%D0%BE%D1%80%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0%20%D0%BC%D0%B0%D1%82%D1%80%D0%B8%D1%86%D1%8B%20%D0%BF%D0%BE%20%D0%B4%D0%B8%D0%B0%D0%B3%D0%BE%D0%BD%D0%B0%D0%BB%D1%8F%D0%BC](https://botayinterview.site/algorithm/%D0%A1%D0%BE%D1%80%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0%20%D0%BC%D0%B0%D1%82%D1%80%D0%B8%D1%86%D1%8B%20%D0%BF%D0%BE%20%D0%B4%D0%B8%D0%B0%D0%B3%D0%BE%D0%BD%D0%B0%D0%BB%D1%8F%D0%BC)
- Публичный detail API: `GET /api/algorithms/public/27` → **HTTP 200**, 3317 байт
- LeetCode: [1329. Sort the Matrix Diagonally](https://leetcode.com/problems/sort-the-matrix-diagonally/) — **точное**

**Условие**

Дан матрица `mat` размером `m x n`, содержащая целые числа. Требуется отсортировать каждую диагональ матрицы в порядке возрастания и вернуть полученную матрицу.

**Диагональ матрицы** - это линия ячеек, начинающаяся от некоторой ячейки в верхней строке или левом столбце и идущая в направлении снизу справа до конца матрицы.

Например, диагональ матрицы, начинающаяся от `mat[2][0]`, где `mat` - матрица размером `6 x 3`, включает ячейки `mat[2][0]`, `mat[3][1]` и `mat[4][2]`.

**Открытые тесты**

Тест 1:

```text
Вход:
mat = [[3, 3, 1, 1], [2, 2, 1, 2], [1, 1, 1, 2]]
Выход:
[[1, 1, 1, 1], [1, 2, 2, 2], [1, 2, 3, 3]]
```

Тест 2:

```text
Вход:
[[11, 25, 66, 1, 69, 7], [23, 55, 17, 45, 15, 52], [75, 31, 36, 44, 58, 8], [22, 27, 33, 25, 68, 4], [84, 28, 14, 11, 5, 50]]
Выход:
mat = [[5, 17, 4, 1, 52, 7], [11, 11, 25, 45, 8, 69], [14, 23, 25, 44, 58, 15], [22, 27, 31, 36, 50, 66], [84, 28, 75, 33, 55, 68]]
```

**Starter code**

````python
def diagonalSort(mat):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

Сортировка матрицы по диагоналям может быть осуществлена путем группировки элементов диагоналей в отдельные списки, сортировки этих списков и последующего присвоения отсортированных значений обратно в матрицу.

ШАГИ:
1. Создаем список списков `g` для хранения элементов диагоналей, где индекс списка соответствует диагонали.
2. Итерируемся по матрице и добавляем каждый элемент в соответствующий список `g`.
3. Сортируем каждый список в `g` в порядке возрастания.
4. Итерируемся по матрице снова и присваиваем отсортированные значения из `g` обратно в матрицу.

СЛОЖНОСТЬ: O(m * n * log(min(m, n))) время, O(m + n) память

```python
def diagonalSort(mat):
    m, n = len(mat), len(mat[0])
    g = [[] for _ in range(m + n)]
    for i, row in enumerate(mat):
        for j, x in enumerate(row):
            g[m - i + j].append(x)
    for e in g:
        e.sort()
    for i in range(m):
        for j in range(n):
            mat[i][j] = g[m - i + j].pop(0)
    return mat
```

---

## Математика

### Палиндром

- ID: 7, slug: `palindrome-number`
- Доступ: **платная** (`is_free: false`)
- Сложность на сайте: **Easy**
- Темы: Математика, Строки
- Страница сайта: [https://botayinterview.site/algorithm/palindrome-number](https://botayinterview.site/algorithm/palindrome-number)
- Публичный detail API: `GET /api/algorithms/public/7` → **HTTP 200**, 4157 байт
- LeetCode: [9. Palindrome Number](https://leetcode.com/problems/palindrome-number/) — **точное**

**Условие**

**Проверка на палиндром**

Для заданного целого числа `x` верните `true`, если `x` является **палиндромом**, и `false` в противном случае.

**Пример 1:**

**Ввод:** x = 121
**Вывод:** true
**Объяснение:** 121 читается как 121 слева направо и справа налево.

**Пример 2:**

**Ввод:** x = -121
**Вывод:** false
**Объяснение:** Слева направо читается как -121. Справа налево становится 121-. Следовательно, это не палиндром.

**Пример 3:**

**Ввод:** x = 10
**Вывод:** false
**Объяснение:** Справа налево читается как 01. Следовательно, это не палиндром.

**Открытые тесты**

Тест 1:

```text
Вход:
x = 10
Выход:
false
```

Тест 2:

```text
Вход:
x = 12321
Выход:
true
```

**Starter code**

````python
def isPalindrome(x: int) -> bool:
    pass
````

````javascript
function isPalindrome(x) {
    
}
````

**Решение из публичного API**

**Решение задачи про палиндром**

**Подход 1: Преобразование в строку**

Самый простой способ - преобразовать число в строку и проверить, является ли она палиндромом.

**Сложность:**
- Временная: $O(\log n)$ - количество цифр
- Пространственная: $O(\log n)$ - строка

```python
def isPalindrome(x: int) -> bool:
    if x < 0:
        return False

    s = str(x)
    return s == s[::-1]
```

## Подход 2: Переворот числа (без строк)

Более интересное решение - перевернуть число математически и сравнить с оригиналом.

**Особые случаи:**
- Отрицательные числа - не палиндромы
- Числа, оканчивающиеся на 0 (кроме самого 0) - не палиндромы

**Алгоритм:**
1. Отсекаем отрицательные числа и числа, оканчивающиеся на 0
2. Переворачиваем вторую половину числа
3. Сравниваем первую половину с перевернутой второй

**Сложность:**
- Временная: $O(\log n)$
- Пространственная: $O(1)$

**Код:**

```python
def isPalindrome(x: int) -> bool:
    # Отрицательные числа и числа, оканчивающиеся на 0 (кроме 0)
    if x < 0 or (x % 10 == 0 and x != 0):
        return False

    # Переворачиваем вторую половину числа
    reversed_half = 0
    while x > reversed_half:
        reversed_half = reversed_half * 10 + x % 10
        x //= 10

    # Для нечетного количества цифр: 12321 -> x=12, reversed=123
    # Для четного: 1221 -> x=12, reversed=12
    return x == reversed_half or x == reversed_half // 10
```

**Пример работы для x = 12321:**

1. `x=12321, reversed=0`
2. `x=1232, reversed=1`
3. `x=123, reversed=12`
4. `x=12, reversed=123` (стоп, x ≤ reversed)
5. Проверка: `12 == 123 // 10` → `12 == 12` ✓

---

## Другие и авторские задачи

### Минимальное количество перемещений ящиков

- ID: 36, slug: `minimax-boxes-moves`
- Доступ: **бесплатная** (`is_free: true`)
- Сложность на сайте: **Hard**
- Темы: Строки
- Страница сайта: [https://botayinterview.site/algorithm/minimax-boxes-moves](https://botayinterview.site/algorithm/minimax-boxes-moves)
- Публичный detail API: `GET /api/algorithms/public/36` → **HTTP 200**, 3063 байт
- LeetCode: точное соответствие не подтверждено

**Условие**

Дана строка `s`, представляющая тоннель, где:
`#` обозначает ящик
`.` обозначает пустую клетку
Разрешена операция "передвинуть ящик на 1 клетку" - при этом целевая клетка обязательно должна быть свободной.
Необходимо написать функцию, которая вернёт минимальное количество таких передвижений, которые надо совершить, чтобы все ящики оказались с одной стороны тоннеля (либо все слева, либо все справа).

**Открытые тесты**

Тест 1:

```text
Вход:
s = "#.#.#"
Выход:
3
```

Тест 2:

```text
Вход:
s = "##.."
Выход:
0
```

Тест 3:

```text
Вход:
s = ".#.#."
Выход:
1
```

**Starter code**

````python
def minMoves(s: str):
    pass
````

````javascript
function solution(nums) {
    
}
````

**Решение из публичного API**

**ИДЕЯ**: 
Использовать жадный алгоритм с подсчётом расстояний. Для каждого ящика вычисляем, сколько ходов нужно, чтобы переместить его на следующую доступную позицию с нужной стороны. Сравниваем два варианта: сбор всех ящиков слева и сбор всех ящиков справа.

**ШАГИ**:
Создаём вспомогательную функцию `f1(s)`, которая считает ходы для сбора ящиков слева.
Используем переменную `start` для отслеживания следующей целевой позиции (начинаем с 0).
Проходим по строке: когда находим ящик `#`, вычисляем расстояние end - start и добавляем к сумме.
Увеличиваем `start` на 1 (следующая целевая позиция).
Возвращаем минимум между `f1(s)` и `f1(s[::-1])` (оригинальная строка и перевёрнутая).

```python
def minMoves(s: str) -> int:
    def f1(s: str) -> int:
        window_sum = 0
        start = 0
        
        for end in range(len(s)):
            if s[end] == '#':
                r = end - start
                window_sum += r
                start += 1
        
        return window_sum
    
    return min(f1(s), f1(s[::-1]))
```

---

## Задачи без точного совпадения LeetCode

- **Минимальное количество перемещений ящиков (ID 36):** условие про одномерный тоннель не совпадает с LeetCode 1263 и LeetCode 1769; ссылка не добавлена.
- **Скалярное произведение сжатых векторов (ID 68):** точного совпадения не найдено. По технике двух указателей это близкая вариация LeetCode 1868, но результатом является скаляр, а не RLE-массив.
- **Сжатие последовательных пробелов (ID 69):** точное соответствие официальной задаче LeetCode не подтверждено.

