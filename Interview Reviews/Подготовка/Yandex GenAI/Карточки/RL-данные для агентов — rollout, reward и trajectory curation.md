---
tags: [yandex, interview-prep, rl, rlhf, reward, trajectories, synthetic-data]
тип: теория и практика
уровень: senior
сложность: высокая
статус: готово
готовность: 100
создано: 2026-08-15
вакансии: [agents, Alice AI LLM, Alice AI VLM, R&D, YandexART]
источники:
  - RewardBench
  - Self-Instruct
  - tau-bench
предпосылки:
  - "NLP/LLM и Промпт-инжиниринг/Alignment и обучение LLM — Pretraining, SFT, RLHF, DPO, SimPO"
  - "Interview Reviews/Подготовка/Yandex GenAI/Карточки/Агентные среды — API, state, verifier и golden trajectories"
---

# RL-данные для агентов — rollout, reward и trajectory curation

> [!abstract] Суть
> Для собеседования важнее понимать, как устроен качественный signal loop, чем уметь запускать полноценный PPO на большом кластере. Главный вопрос: как получить данные, которые действительно улучшают нужное поведение и не обучают модель обходить evaluator.

## 0. Сначала простая картина

Здесь reinforcement learning (RL) удобно понимать как цикл обучения по последствиям действий. Модель пробует решить задачу, мы записываем её попытку, проверяем результат и превращаем проверку в сигнал обучения. Если сигнал неверный, модель начнёт оптимизировать не полезное поведение, а удобный для метрики трюк.

SFT — обучение на примерах правильного поведения: «в такой ситуации сделай так». Preference data — пары или списки ответов, где человек выбрал более удачный вариант. RL data — опыт взаимодействия с окружением: последовательность действий, промежуточные наблюдения и итоговая награда. Rollout — одна такая попытка, то есть запуск текущей политики на задаче. Reward — числовая оценка результата; это не сама истина, а её приближённый измеритель.

Поэтому основная работа аналитика здесь — не магия алгоритма оптимизации. Нужно решить, какие попытки сохранять, как обнаруживать ошибки и дубли, как не допускать утечки теста и как понять, что улучшился реальный навык. Trajectory curation означает отбор и очистку полных историй действий, а reward hacking — ситуацию, когда модель получает высокий балл, формально обходя смысл задания.

Дальше карточка разбирает три слоя цикла: откуда берётся reward, как запускаются rollout, как отбираются траектории и как проверяется польза данных на независимом eval. Такой маршрут связывает теорию RL с практической задачей data scientist.

### Словарь перед стартом

- **Policy** — текущая стратегия модели, выбирающая следующее действие.
- **Episode** — один полный запуск от начального состояния до завершения.
- **Reward model** — отдельная модель, которая оценивает качество ответа или поведения.
- **Holdout** — отложенные задачи, которые не использовались при сборе или отборе данных.

## 1. Где находятся данные в цикле обучения

### SFT

На входе демонстрация правильного ответа или действия:

~~~text
prompt + context -> target answer/action
~~~

Модель учится имитировать хорошие примеры, но не видит альтернатив и не получает явного сигнала о том, насколько действие лучше другого.

### Preference learning

Есть chosen и rejected response/trajectory:

~~~text
(prompt, y_chosen, y_rejected)
~~~

Модель или reward model учится предпочитать выбранный вариант. DPO оптимизирует preference напрямую, а reward model может использоваться для reranking или RL.

### RL в среде

Модель сама делает rollout, получает reward и обновляет policy. Это нужно, когда качество зависит от последовательности действий и конечного состояния, а не от одной демонстрации.

> [!tip] Переход от типов обучения к сигналу
> Мы увидели, что SFT, preference data и RL data учат разным вещам. Теперь разберём главный вопрос RL-цикла: откуда берётся оценка результата и почему она может вводить модель в заблуждение.

## 2. Виды reward

| Reward | Пример | Сильная сторона | Риск |
|---|---|---|---|
| Outcome | корзина достигла target state | объективный результат | sparse signal |
| Process | каждый шаг соответствует policy | credit assignment | можно поощрить красивый, но бесполезный шаг |
| Human preference | эксперт выбрал trajectory A | богатый смысловой сигнал | дорого и субъективно |
| LLM judge | judge оценил trace | масштабирование | bias и prompt sensitivity |
| Deterministic verifier | проверка БД/JSON/policy | воспроизводимость | видит только формализуемые свойства |

Для agent задач желательно комбинировать verifier и human/judge audit:

~~~text
reward = outcome_score - safety_penalty - invalid_call_penalty - cost_penalty
~~~

Вес penalty нельзя выбирать только по удобству: маленькая вероятность критичной ошибки может быть важнее большого числа косметических улучшений.

## 3. Rollout pipeline

~~~text
task generation
  -> agent rollout
  -> tool/environment logs
  -> verifier and judge
  -> accepted/rejected trajectories
  -> filtering and deduplication
  -> SFT / preference / RL dataset
  -> independent held-out eval
~~~

Для каждого rollout нужно сохранять:

- task и initial state;
- model/system prompt/tool schema versions;
- sequence of observations/actions;
- tool arguments/results;
- final state;
- verifier output;
- judge output;
- latency, tokens и стоимость;
- причины rejection.

Без этого невозможно понять, почему конкретная trajectory была принята.

## 4. Quality curation

Фильтруем:

- invalid tool calls;
- trajectories, где reward получен из-за бага environment;
- дубликаты и near-duplicates;
- слишком лёгкие примеры;
- leakage из held-out eval;
- синтетические traces с неверным rationale;
- unsafe или privacy-sensitive content;
- примеры с неоднозначным ground truth.

Важна diversity не только по словам, но и по типу reasoning, сложности, пользователю, языку, состоянию environment и пути решения.

> [!tip] Переход от отбора к защите от обхода
> Даже аккуратно отобранные rollout не гарантируют полезного обучения, если модель научилась получать высокий балл формальным трюком. Поэтому отдельно проверяем, не оптимизируем ли мы саму метрику вместо задачи.

## 5. Reward hacking

Reward hacking — модель оптимизирует измеритель, а не задачу.

Примеры:

- агент кладёт правильный товар в корзину, но нарушает подтверждение;
- verifier смотрит только на идентификатор товара, а агент подменяет цену;
- judge оценивает длинное объяснение, хотя действие неверно;
- модель генерирует ровно формат, который ожидает evaluator, не решая задачу;
- synthetic generator создаёт похожие задачи, поэтому train и test отличаются только словами.

Поэтому нужны adversarial tests, независимый human audit и несколько evaluator layers.

## 6. Trajectory curation для RL

Хорошая accepted trajectory должна иметь:

1. корректно понятый user goal;
2. допустимые tool calls;
3. отсутствие лишних опасных действий;
4. достижение target state;
5. устойчивость к небольшим изменениям формулировки;
6. достаточную трудность, чтобы приносить обучающий сигнал.

Для rejected trajectory нужно сохранять причину rejection. Пара accepted/rejected особенно полезна, если различается одним локальным решением: это облегчает credit assignment.

## 7. Synthetic data

Генерировать случайные инструкции слабее, чем строить failure-driven pipeline:

~~~text
найденный failure slice -> параметризованный генератор задач
-> model rollouts -> verifier/judge -> human audit
~~~

Параметры генератора должны контролировать сложность: количество ограничений, длину диалога, число tools, неоднозначность, редкость товара и тип policy.

Synthetic data нужно проверять на:

- разнообразие;
- factual consistency;
- отсутствие шаблонного языка;
- независимость от evaluator prompt;
- переносимость на real-like held-out set.

## 8. Практический пример

Наблюдается failure: агент забывает проверить совместимость товара с устройством.

Плохой ответ: добавить 10 000 общих диалогов о покупках.

Хороший pipeline:

1. Создать taxonomy subcases: разные типы совместимости, отсутствие данных, конфликт характеристик.
2. Сгенерировать задания с разными формулировками.
3. Добавить hard negatives: товар почти подходит, но не подходит по одному условию.
4. Проверять final state и rationale отдельно.
5. Сравнить random и failure-driven data на locked benchmark.

## 9. Как понять, что data помогла

Нужно сравнивать не только training loss и judge score:

- held-out task success;
- improvement именно в target slice;
- unchanged или improved safety guardrails;
- перенос на новые формулировки;
- regression на соседних навыках;
- human preference;
- стоимость и стабильность rollout.

Если вырос только score judge, а human-held-out и task verifier не изменились, data pipeline не доказан.

## Практическая задача

Для mock-agent environment собрать 30 экспертных и 200 model-generated trajectories. Разметить accepted/rejected, причины отказа и тип ошибки. Сравнить random selection с failure-driven selection и оформить report.

## Типичные ошибки

- **Считать reward объектом истины:** reward — лишь proxy поведения.
- **Смешивать train и eval:** synthetic generator может воспроизвести test patterns.
- **Удалять все сложные trajectories:** именно они могут содержать полезный signal.
- **Оценивать только final answer:** теряются ошибки промежуточных действий.
- **Не логировать environment:** невозможно воспроизвести rollout.
- **Оптимизироваться под judge:** возникает evaluator overfitting.

## Вопросы для интервью

> [!question] Что выбрать для agent task: SFT, DPO или RL?
> Если есть качественные демонстрации — начать с SFT. Если есть пары предпочтений — DPO/preference learning. Если результат зависит от последовательности действий и есть надёжный verifier/environment — рассматривать RL, но сначала проверить reward и устойчивость.

> [!question] Чем outcome reward хуже process reward?
> Он разреженный и плохо объясняет, какой шаг привёл к успеху/ошибке. Process reward даёт credit assignment, но может поощрить локально красивое действие, не ведущее к цели.

> [!question] Как обнаружить reward hacking?
> Сравнить reward с независимым verifier/human evaluation, добавить adversarial environments и анализировать trajectory, а не только итоговое число.

## Проверка себя

- Спроектировать reward для покупки с обязательным подтверждением.
- Привести три примера rejected trajectory.
- Объяснить различие SFT, preference learning и RL.
- Назвать минимальный лог, необходимый для воспроизводимости rollout.

## Связано

- [[NLP/LLM и Промпт-инжиниринг/Alignment и обучение LLM — Pretraining, SFT, RLHF, DPO, SimPO]]
- [[NLP/LLM и Промпт-инжиниринг/LLM-as-a-Judge — rubric, bias и дообучение]]

## Источники

- [RewardBench](https://github.com/allenai/reward-bench)
- [Self-Instruct](https://arxiv.org/abs/2212.10560)
- [τ-bench](https://arxiv.org/abs/2406.12045)

---
[[Interview Reviews/Подготовка/Yandex GenAI/🗺️ Индекс|Назад к разделу]]
