---
topic: Batch Normalization
card_ref: "[[Batch Normalization]]"
generated: 2026-04-29
---

## Q001
type: fact
difficulty: junior
key_concepts: [нормализация по среднему, параметр beta как сдвиг, избыточность bias]
optional_concepts: [экономия памяти и вычислений]
text: Объясните, почему в слое перед Batch Normalization принято отключать параметр `bias` (`bias=False`). Что произойдёт, если `bias` всё-таки оставить?
follow_up:
  - Если BN убирает смещение через вычитание среднего, то зачем тогда в самом BN есть обучаемый параметр β? Чем он отличается от bias предыдущего слоя?

---

## Q002
type: trap
difficulty: junior
key_concepts: [running statistics, model.eval(), статистика батча vs накопленная статистика]
optional_concepts: [train/test distribution mismatch]
text: Коллега обучил модель, получил отличный loss на валидации во время обучения, но при деплое на тех же данных метрика резко упала. Код инференса выглядит так: `model.train(); output = model(x)`. В чём проблема?
follow_up:
  - Допустим, коллега исправил на `model.eval()`, но метрика всё равно хуже, чем на валидации во время обучения. Какой ещё сценарий мог привести к расхождению статистик BN?

---

## Q003
type: compare
difficulty: middle
key_concepts: [нормализация по батчу vs нормализация по признакам, зависимость от batch_size, применимость в RNN/Transformer]
optional_concepts: [Instance Normalization, Group Normalization]
text: Сравните Batch Normalization и Layer Normalization. В каких архитектурах и задачах каждый из методов предпочтительнее и почему?
follow_up:
  - Вы разрабатываете модель для задачи NLP с переменной длиной последовательности и batch_size=1 (например, авторегрессивная генерация). Какую нормализацию выберете и что потеряете по сравнению с BN?

---

## Q004
type: chain
difficulty: middle
key_concepts: [обучаемые параметры γ и β, восстановление выразительности сети, градиентный поток]
optional_concepts: [identity mapping при γ=1 β=0]
text: Зачем после нормализации в BN применяются обучаемые параметры γ (scale) и β (shift)? Разве нормализация не обнуляет их эффект?
follow_up:
  - Что произойдёт с обучением, если инициализировать γ=0 для всех BN-слоёв? Есть ли практические кейсы, где это делают намеренно?
  - Если убрать γ и β совсем и оставить только нормализацию, как это повлияет на выразительность (expressiveness) сети?

---

## Q005
type: scenario
difficulty: middle
key_concepts: [running mean/variance, накопление статистик, train/eval режимы, data leakage через статистику]
optional_concepts: [warm-up батчи, экспоненциальное скользящее среднее]
text: Команда обучила модель классификации изображений. На train и val loss всё выглядит идеально. Модель выкатили в продакшн — работает корректно. Но через неделю пришёл репорт: на первых ~50 запросах после перезапуска сервиса предсказания нестабильны и хуже среднего. Дальше всё нормализуется. В чём может быть причина?
follow_up:
  - Как изменится поведение, если уменьшить `momentum` параметр BN (скорость обновления running statistics) в несколько раз? Это поможет или усугубит проблему?

---

## Q006
type: debug
difficulty: middle
key_concepts: [порядок операций train/eval, утечка тестовой статистики, корректность оценки модели]
optional_concepts: [DataLoader shuffle, drop_last]
text: Вот фрагмент тренировочного цикла на PyTorch. Найдите ошибку, которая приводит к завышенной метрике на тестовой выборке:
```python
for epoch in range(epochs):
    for x, y in train_loader:
        optimizer.zero_grad()
        out = model(x)
        loss = criterion(out, y)
        loss.backward()
        optimizer.step()

    # Оценка на тесте
    total, correct = 0, 0
    for x, y in test_loader:
        out = model(x)
        preds = out.argmax(dim=1)
        correct += (preds == y).sum().item()
        total += y.size(0)
    print(f"Accuracy: {correct/total:.4f}")
```
follow_up:
  - После исправления оказалось, что `drop_last=False` в `test_loader` и последний батч содержит только 2 примера. Как это повлияет на running statistics, если забыть переключить модель в eval перед тестом?

---

## Q007
type: system_design
difficulty: senior
key_concepts: [running statistics vs batch statistics, стабильность инференса, обновление статистик при data drift, версионирование модели]
optional_concepts: [онлайн-обновление BN, экспоненциальное скользящее среднее momentum]
text: Ваша модель с несколькими BN-слоями обучена на исторических данных и задеплоена в продакшн. Спустя 3 месяца распределение входных данных начало смещаться (data drift) — средние значения признаков изменились на 15–20%. Предложите стратегию адаптации модели без полного переобучения с нуля, учитывая, что размечать новые данные дорого.
follow_up:
  - Если решите «перекалибровать» BN-статистики, прогнав новые данные через замороженную модель в режиме train(), какие риски это несёт и как их митигировать?
  - Как выстроить мониторинг, чтобы заранее детектировать момент, когда drift стал критичным для BN-слоёв?

---

## Q008
type: tradeoff
difficulty: senior
key_concepts: [зависимость BN от размера батча, альтернативы нормализации, компромисс между стабильностью и точностью, требования к памяти]
optional_concepts: [Group Normalization, gradient accumulation как обходной путь]
text: Вы обучаете детектор объектов на медицинских снимках высокого разрешения (3D MRI). Из-за объёма данных максимальный `batch_size` на вашем железе — 2. Коллега предлагает два варианта: (A) оставить BN с batch_size=2, (B) заменить BN на Group Normalization. Аргументируйте выбор, явно назвав, чем жертвуете в каждом случае.
follow_up:
  - Предположим, вы выбрали Group Normalization, но у вас есть возможность использовать gradient accumulation (накопление градиентов за 16 шагов перед обновлением весов). Меняет ли это ваш выбор нормализации и почему?
