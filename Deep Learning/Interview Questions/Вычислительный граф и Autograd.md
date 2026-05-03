---
topic: Вычислительный граф и Autograd
card_ref: "[[Вычислительный граф и Autograd]]"
generated: 2026-04-29
---

## Q001
type: fact
difficulty: junior
key_concepts: [requires_grad, флаг отслеживания градиентов, листовой тензор, backward]
optional_concepts: [retain_graph]
text: Что произойдёт, если вызвать `.backward()` на тензоре, у которого `requires_grad=False`? Объясните, зачем вообще нужен этот флаг.
follow_up:
  - Можно ли включить `requires_grad` у тензора после того, как он уже участвовал в операциях? Что это изменит?

## Q002
type: trap
difficulty: junior
key_concepts: [torch.no_grad, утечка памяти, вычислительный граф на инференсе, OOM]
optional_concepts: [model.eval]
text: Коллега говорит: «На валидации мы не вызываем `.backward()`, значит градиенты не считаются и память не тратится — `torch.no_grad()` не нужен». Он прав?
follow_up:
  - Чем `torch.no_grad()` отличается от `model.eval()`? Взаимозаменяемы ли они при инференсе?

## Q003
type: compare
difficulty: middle
key_concepts: [динамический граф, статический граф, define-by-run, define-and-run]
optional_concepts: [torch.jit.trace, XLA-компиляция]
text: PyTorch строит вычислительный граф динамически (define-by-run), а TensorFlow 1.x — статически (define-and-run). Сравните оба подхода: в каких сценариях каждый из них даёт преимущество, а в каких проигрывает?
follow_up:
  - Как появление `torch.compile` и `tf.function` стирает эту границу? Какие компромиссы они вносят?

## Q004
type: chain
difficulty: middle
key_concepts: [chain rule, локальные производные, backward pass, accumulate gradients]
optional_concepts: [якобиан, grad_fn]
text: Объясните, как именно PyTorch считает `x.grad` и `y.grad` для выражения `f = (x + y) * y`. Пройдитесь по шагам backward-прохода.
follow_up:
  - Что случится, если вызвать `f.backward()` дважды подряд без `retain_graph=True`? Почему граф уничтожается после первого вызова?

## Q005
type: debug
difficulty: middle
key_concepts: [накопление градиентов, optimizer.zero_grad, grad accumulation, неверные обновления весов]
optional_concepts: [gradient clipping]
text: Инженер написал цикл обучения. Лосс убывает, но модель сходится значительно медленнее ожидаемого и финальное качество ниже бейзлайна. Найдите ошибку:
```python
for epoch in range(10):
    for x_batch, y_batch in dataloader:
        pred = model(x_batch)
        loss = criterion(pred, y_batch)
        loss.backward()
        optimizer.step()
```
follow_up:
  - Есть паттерн gradient accumulation, где `zero_grad` намеренно вызывается раз в N шагов. Как отличить баг в коде выше от корректного gradient accumulation?

## Q006
type: scenario
difficulty: middle
key_concepts: [кастомный backward, Function.apply, числовое дифференцирование, autograd hook]
optional_concepts: [torch.autograd.gradcheck]
text: Команда реализует слой с нестандартной операцией поверх CUDA-кернела, у которой нет готового `grad_fn` в PyTorch. Менеджер требует, чтобы слой обучался как обычно. Как вы решите задачу, и как убедитесь, что производная реализована правильно?
follow_up:
  - Если нет возможности написать аналитическую производную, какой запасной вариант существует? Какова его вычислительная стоимость?

## Q007
type: tradeoff
difficulty: senior
key_concepts: [gradient checkpointing, потребление памяти, повторное вычисление forward, скорость обучения]
optional_concepts: [mixed precision training, activation offloading]
text: Вы обучаете большую трансформер-модель. GPU упирается в память из-за хранения промежуточных активаций для backward-прохода. Перед вами два варианта: уменьшить batch size вдвое или включить gradient checkpointing. Аргументируйте выбор с учётом trade-off по памяти, скорости и стабильности обучения.
follow_up:
  - Gradient checkpointing освобождает память за счёт повторного вычисления активаций. Как вы оцените, для каких слоёв модели checkpointing даст наибольший выигрыш по памяти при наименьших потерях в скорости?

## Q008
type: system_design
difficulty: senior
key_concepts: [inference без графа, torchscript или ONNX экспорт, отсутствие autograd на проде, latency vs гибкость]
optional_concepts: [torch.fx, quantization awareness]
text: Вы деплоите модель в real-time сервис с требованием p99-латентности < 20 мс. Исследователи продолжают эксперименты в динамическом PyTorch с кастомными операциями. Спроектируйте пайплайн экспорта и сервинга так, чтобы production-версия не тащила за собой накладные расходы autograd, а итерации исследователей при этом не замедлялись.
follow_up:
  - Один из кастомных слоёв не поддаётся трассировке `torch.jit.trace` из-за data-dependent control flow. Как вы его обработаете, не жертвуя целевой латентностью?
