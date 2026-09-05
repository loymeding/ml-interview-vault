---
tags: [yandex, interview-prep, product-analytics, user-signal, offline-online, rlhf]
тип: теория и практика
уровень: senior
сложность: высокая
статус: готово
готовность: 100
создано: 2026-08-15
вакансии: [YandexART, Alice AI LLM, agents, R&D]
источники: [HELM, Yandex vacancy descriptions]
предпосылки:
  - "А-Б тесты/Методология/Иерархия метрик — Целевая, Guardrail и Proxy"
  - "Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка text-to-image — human preference и failure modes"
---

# Product signal — offline eval, логи и пользовательская ценность

> [!abstract] Суть
> Пользовательское действие — weak signal, а не готовая label. Его нужно очищать от selection, position, latency и intent confounders и проверять на независимой human/product оценке.

## 0. Сначала простая картина

Логи продукта показывают, что пользователь сделал, но почти никогда не объясняют почему. Нажатие «перегенерировать» может означать плохое качество, скучный запрос, долгий ответ или случайный клик. Поэтому product signal — это слабый косвенный признак качества, а не готовая правильная метка.

Offline eval — проверка моделей на заранее сохранённом наборе задач. Она удобна для быстрых сравнений: все версии видят одни и те же запросы. Online evaluation — наблюдение за поведением пользователей в живом продукте, например в A/B-тесте. A/B-тест случайно разделяет пользователей между вариантами и помогает оценить причинный эффект изменения.

Confounder — фактор, который влияет и на поведение пользователя, и на измеряемую метрику, из-за чего легко сделать неверный вывод. Selection bias возникает, если в логах представлены не все пользователи, а только те, кто дошёл до определённого шага. Latency — задержка ответа; она может ухудшить метрику даже при неизменном содержании.

Задача аналитика — связать три уровня: технический результат модели, человеческую оценку и пользу в продукте. В карточке этот путь разбирается последовательно: сначала типы сигналов и их искажения, затем weak labels, разрыв offline-to-online и иерархия метрик.

### Словарь перед стартом

- **Event** — зафиксированное действие пользователя или системы в логе.
- **Proxy** — косвенная метрика, используемая вместо недоступного прямого результата.
- **Guardrail** — ограничение, которое не должно ухудшиться, даже если основная метрика растёт.
- **Causal effect** — изменение метрики именно из-за версии продукта, а не из-за состава аудитории.

## 1. Offline и online

Offline eval отвечает на вопрос:

~~~text
Какая версия модели лучше на фиксированном наборе задач?
~~~

Online product evaluation отвечает на вопрос:

~~~text
Как изменилась ценность и поведение реальных пользователей после релиза?
~~~

Offline нужен для быстрых, контролируемых итераций. Online видит UI, latency, fallback, ranking, реальные формулировки и неожиданные сценарии. Ни один слой не заменяет другой.

> [!tip] Переход от двух режимов оценки к данным
> Offline и online отвечают на разные вопросы. Теперь перечислим конкретные события в логах и разберём, какую часть пользовательского опыта каждое из них приближённо отражает.

## 2. Типы пользовательских сигналов

| Событие | Возможная интерпретация | Почему нельзя делать прямой вывод |
|---|---|---|
| regenerate | результат не удовлетворил | пользователь может исследовать варианты |
| save/share | результат полезен или красив | зависит от UI и social intent |
| edit prompt | ответ не попал в задачу | пользователь мог уточнять замысел |
| abandon | плохой ответ | причиной может быть latency |
| repeat query | потребность не закрыта | пользователь мог проверять формулировки |

Event становится label только после проверки контекста и альтернативных объяснений.

## 3. Основные confounders

### Position bias

Первый или визуально более заметный вариант выбирают чаще.

### Exposure bias

Мы видим действия только для показанных outputs. Непоказанный вариант не получает шанс на save/share.

### Latency

Долгое ожидание может вызвать abandon независимо от качества.

### Selection bias

Пользователи, которые активнее сохраняют изображения, отличаются от остальных.

### Novelty bias

Новый стиль или эффект может получить реакцию, не связанную с долгосрочной ценностью.

### Intent ambiguity

Один и тот же regenerate означает разные вещи для разных целей пользователя.

## 4. Weak labels

Weak label полезен как вероятностный сигнал:

~~~text
P(preferred = 1 | event, context)
~~~

Не стоит превращать каждое событие в жёсткую label. Лучше:

- использовать событие для error mining;
- строить candidate preference pairs;
- учитывать exposure, position и latency;
- делать human audit sample;
- оставлять uncertain class.

> [!tip] Переход от отдельных искажений к системному разрыву
> Даже если каждый сигнал кажется разумным, итог offline eval может не совпасть с продуктом. Поэтому соберём причины этого разрыва в одну проверяемую гипотезу.

## 5. Offline-to-online gap

Модель может улучшить benchmark, но ухудшить продукт:

- ответы стали длиннее и медленнее;
- offline test не содержит редких product intents;
- judge любит стиль, а пользователю нужен короткий результат;
- модель стала лучше на average, но хуже на high-value segment;
- новая версия вызывает больше expensive tools;
- UI показывает лучший ответ ниже в списке.

Поэтому product dashboard должен содержать quality, guardrails, latency, cost и user outcomes.

## 6. Практический пример YandexART

В логах видно, что версия B имеет больше regenerate.

Гипотезы:

1. B хуже следует prompt.
2. B генерирует дольше.
3. B предлагает более однообразные варианты.
4. UI изменил кнопку/regeneration flow.
5. Пользователи используют B для поиска большего числа вариантов.
6. B действительно эстетически слабее на конкретных slices.

План:

- сравнить latency и exposure;
- провести offline prompt adherence evaluation;
- разобрать regenerate по prompt taxonomy;
- сделать pairwise human audit;
- проверить UI/позицию;
- затем решить, какой data signal собирать.

## 7. Практическая задача

Создать синтетический лог с prompt, model version, position, latency, output, user events и slice labels. Для каждого события решить, пригодно ли оно для:

- error mining;
- weak preference;
- reward training;
- release decision.

Для каждого решения указать confounders и нужную проверку.

## 8. Как связать метрики

Удобна иерархия:

- primary: user task success или validated preference;
- secondary: quality dimensions;
- guardrails: safety, latency, cost, complaint rate;
- diagnostic: конкретные error slices;
- proxy: clicks, regenerate, save, judge score.

Proxy не должен автоматически становиться release metric. Сначала нужна валидация связи с primary outcome.

## Типичные ошибки

- **Regenerate = плохой ответ:** игнорируется intent и latency.
- **Save/share = ground truth:** есть selection и social confounders.
- **Сравнить только до и после:** нет контроля и randomized exposure.
- **Оптимизировать proxy:** модель учится получать event, а не решать задачу.
- **Игнорировать cost/latency:** продуктовая ценность может снизиться при росте качества.
- **Не анализировать slices:** средняя метрика скрывает деградацию сегмента.

## Вопросы для интервью

> [!question] Как понять, что regenerate вызван качеством, а не latency?
> Разделить события по latency buckets, сравнить с offline quality на тех же prompts, учесть UI/exposure и проверить human sample.

> [!question] Можно ли использовать save как reward?
> Только как noisy/weak signal после калибровки и контроля selection bias. Для критичного обучения нужен human или verifier audit.

> [!question] Что делать, если offline и online расходятся?
> Идентифицировать непокрытые product factors, обновить benchmark, проверить logging/experiment design и не делать вывод только по одной стороне.

## Проверка себя

- Назвать пять confounders пользовательского сигнала.
- Разделить primary, guardrail, diagnostic и proxy metrics.
- Составить investigation plan для роста regenerate.
- Объяснить, почему offline eval нельзя выкинуть после A/B-теста.

## Связано

- [[А-Б тесты/Методология/Иерархия метрик — Целевая, Guardrail и Proxy]]
- [[А-Б тесты/Продвинутые методы/Uplift-моделирование]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка text-to-image — human preference и failure modes]]

## Источники

- [HELM](https://crfm.stanford.edu/helm/)
- [YandexART vacancy](https://yandex.ru/jobs/vacancies/analitikrazrabotchik-modeli-dlya-generatsii-izobrazheniy-yandexart-47577)
- [Agent scenarios vacancy](https://yandex.ru/jobs/vacancies/analitik-kachestva-produktovih-agentskih-stsenariev-47049)

---
[[Interview Reviews/Подготовка/Yandex GenAI/🗺️ Индекс|Назад к разделу]]
