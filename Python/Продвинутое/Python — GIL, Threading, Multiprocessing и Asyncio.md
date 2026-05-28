---
tags: [python, gil, threading, multiprocessing, asyncio, concurrency, parallelism]
тип: теория и практика
уровень: middle
сложность: средняя
статус: готово
готовность: 90
создано: 2026-05-28
источники:
  - "Вопросы пользователя по многопоточности"
  - "Python 3.14 docs: Python support for free threading"
предпосылки: []
связано:
  - "Python — Параллелизм и Конкурентность"
сравнить-с: []
---

# Python — GIL, Threading, Multiprocessing и Asyncio

> [!abstract] Суть
> В обычном CPython GIL не даёт нескольким Python-потокам одновременно исполнять байткод, поэтому threads подходят в основном для I/O-bound задач. Для CPU-bound задач используют multiprocessing, C/NumPy-код без GIL или free-threaded builds. Asyncio даёт конкурентность через event loop и cooperative multitasking.

## Ответ

### 1. GIL и ограничения до Python 3.14

GIL (Global Interpreter Lock) — глобальная блокировка интерпретатора в стандартном CPython. Она гарантирует, что в один момент времени только один поток исполняет Python bytecode.

Следствия:
- чистый Python CPU-bound код в threads обычно не ускоряется;
- I/O-bound код может ускориться, потому что поток отдаёт управление во время ожидания сети/диска;
- C extensions могут отпускать GIL, поэтому NumPy/Polars/часть ML-библиотек способны параллелиться внутри;
- multiprocessing обходит GIL, потому что каждый процесс имеет свой интерпретатор и свою память.

Начиная с Python 3.13 появились free-threaded builds CPython, где GIL можно отключать. В Python 3.14 это направление продолжает развиваться, но обычная практическая оговорка остаётся: не весь сторонний код и C extensions готовы к free-threading, а некоторые расширения могут включать GIL обратно.

### 2. Threading

Threading создаёт потоки внутри одного процесса. У потоков общая память, поэтому обмен данными дешёвый, но появляются race conditions.

Когда подходит:
- сетевые запросы;
- чтение/запись файлов;
- ожидание базы данных;
- много задач "ждём внешний ресурс".

Когда не подходит:
- тяжёлые Python-циклы;
- CPU-bound feature generation;
- чистый Python парсинг больших данных.

Инструменты:
- `threading.Thread`;
- `concurrent.futures.ThreadPoolExecutor`;
- `Lock`, `RLock`, `Semaphore`, `Queue`.

### 3. Multiprocessing

Multiprocessing запускает несколько процессов. У каждого процесса:
- свой Python-интерпретатор;
- своя память;
- свой GIL.

Поэтому CPU-bound Python-код может реально выполняться параллельно на разных ядрах.

Цена:
- данные нужно сериализовать/pickle;
- передача больших объектов дорогая;
- старт процесса дороже старта потока;
- shared state сложнее.

GPU-нюанс: для GPU часто лучше один главный процесс управляет батчированием, а тяжёлые операции идут в CUDA kernels. Несколько процессов на одну GPU могут конфликтовать за память и снижать throughput, если нет аккуратного scheduler.

### 4. Asyncio

`asyncio` — однопоточная конкурентность на event loop. Корутина добровольно отдаёт управление в точках `await`.

```python
async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()
```

Когда подходит:
- тысячи сетевых соединений;
- API gateway;
- асинхронные клиенты к БД/очередям;
- high-concurrency I/O.

Когда не подходит:
- CPU-bound код без выноса в process/thread pool;
- блокирующие библиотеки внутри event loop.

Если внутри coroutine вызвать тяжёлую синхронную функцию, event loop остановится и остальные задачи не будут выполняться.

### 5. Как выбрать подход

| Задача | Подход |
|---|---|
| 1000 HTTP-запросов | `asyncio` или threads |
| много блокирующих legacy SDK | threads |
| чистый Python CPU-bound | multiprocessing |
| NumPy/ML операции | часто достаточно vectorized/C/GPU |
| web-сервис с async DB client | asyncio |
| pipeline с большими объектами | осторожно с multiprocessing из-за serialization |

## Формула / Схема

**Concurrency vs parallelism:**

```text
Concurrency: задачи перекрываются по времени, но не обязательно исполняются одновременно.
Parallelism: задачи реально исполняются одновременно на разных ядрах/устройствах.
```

**Threading для I/O:**

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=20) as pool:
    results = list(pool.map(fetch_url, urls))
```

**Multiprocessing для CPU:**

```python
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor() as pool:
    results = list(pool.map(cpu_heavy_func, items))
```

## Короткий пример

Если функция делает HTTP-запрос и 95% времени ждёт сеть, threads/asyncio ускорят pipeline. Если функция считает миллион Python-операций в цикле, threads будут бороться за GIL, и лучше multiprocessing или векторизация.

## Типичные ошибки

- **Ждать ускорения CPU-bound кода от threads:** GIL почти всё съест.
- **Блокировать event loop синхронной функцией:** одна тяжёлая функция остановит все coroutine.
- **Передавать огромные DataFrame между процессами:** serialization может быть дороже вычислений.
- **Считать free-threading магической кнопкой:** сторонние расширения, thread-safety и overhead всё равно нужно проверять.

## Каверзные вопросы

> [!question] Когда NumPy может ускориться в threads несмотря на GIL?
> Когда вычисление уходит в C/BLAS/CUDA-код, который отпускает GIL. Тогда Python-поток ждёт, а тяжёлая работа выполняется вне байткода CPython.

> [!question] Чем coroutine отличается от thread?
> Coroutine не исполняется ОС как отдельный поток. Она выполняется в event loop и переключается кооперативно в `await`-точках.

## Проверка себя

- Почему multiprocessing обходит GIL?
- Почему `asyncio` не ускоряет CPU-bound код само по себе?
- Что опасно в общей изменяемой памяти потоков?
- Когда лучше `ThreadPoolExecutor`, а когда `ProcessPoolExecutor`?

## Предпосылки

- Базовый Python

## Связано

- [[Python/Продвинутое/Python — Параллелизм и Конкурентность]]

## Источники

- Вопросы пользователя по многопоточности
- [Python 3.14 docs: Python support for free threading](https://docs.python.org/3.14/howto/free-threading-python.html)

---
[[🗺️ Индекс|Назад к разделу]]
