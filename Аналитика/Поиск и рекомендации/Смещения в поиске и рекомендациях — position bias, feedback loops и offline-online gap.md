---
tags: [аналитика, поиск, рекомендации, position-bias, selection-bias, feedback-loop, offline-online]
тип: теория
уровень: senior
сложность: высокая
статус: готово
готовность: 95
создано: 2026-08-25
источники:
  - "Recommender Systems Handbook"
  - "Designing Machine Learning Systems — Chip Huyen"
предпосылки:
  - "Метрики поиска и рекомендаций — CTR, Success Rate, Precision@K, Recall@K, MRR и NDCG"
  - "Корреляция и причинность — confounders, selection bias и парадокс Симпсона"
связано:
  - "A-B тест от гипотезы до решения — единица рандомизации, метрики и вердикт"
  - "Мониторинг продукта и ML-системы — алерты, срезы, drift и delayed labels"
сравнить-с: []
---

# Смещения в поиске и рекомендациях — position bias, feedback loops и offline-online gap

> [!abstract] Суть
> Логи ranking-системы отражают не только предпочтения пользователя, но и предыдущую policy: что ему решили показать и на какой позиции. Если обучаться на этих логах буквально, модель начинает воспроизводить собственные старые решения.

## Главная проблема implicit feedback

Клик удобен: его много, он появляется быстро и почти бесплатно. Но клик не равен релевантности.

Наблюдаемое действие можно представить так:

$$\Large P(click)=P(examine)\times P(click\mid examine,relevance,attractiveness)$$

Пользователь сначала должен увидеть объект. Поэтому непоказанный или расположенный внизу товар не получает клик, даже если был бы идеален.

## Position bias

Верхние позиции кликают чаще независимо от качества. Если взять clicks как labels, модель «узнает», что объекты старой модели сверху релевантны, потому что они сверху.

Пример:

- товар A на позиции 1 получил CTR 20%;
- товар B на позиции 10 — CTR 3%.

Нельзя заключить, что A в 6.7 раза релевантнее. Нужна оценка examination probability или рандомизация позиций.

### Как бороться

- небольшое безопасное randomization/exploration;
- interleaving для сравнения rankers;
- inverse propensity scoring;
- click models, разделяющие examination и relevance;
- human judgments;
- counterfactual learning-to-rank;
- A/B-тест конечного продукта.

Inverse propensity weighting грубо перевзвешивает редкие наблюдения:

$$\Large \hat R=\frac{1}{N}\sum_i\frac{Y_i\mathbb{1}(A_i=\pi(X_i))}{p(A_i\mid X_i)}$$

Если action имел маленькую вероятность показа, его вес велик. Это снижает bias, но увеличивает variance; нужны clipping и overlap.

## Exposure bias и missing-not-at-random

Для непоказанных объектов нет feedback. Это не случайный пропуск: старая модель специально выбрала, что показать. Dataset отражает ограниченную область каталога.

Если новый ranker предлагает товары вне этой области, offline replay не знает, как пользователи на них отреагировали. Это **off-policy evaluation** problem.

## Popularity bias

Популярные объекты чаще показывают → они получают больше кликов → модель считает их ещё более привлекательными → показывает ещё чаще.

Последствия:

- long tail не получает данных;
- новые товары не могут стартовать;
- выдача становится однообразной;
- продавцы и авторы получают неравный exposure;
- краткосрочный CTR может быть высоким, долгосрочное discovery — слабым.

Guardrails: catalog coverage, exposure Gini, diversity, new-item exposure, creator/seller fairness.

## Feedback loop

```text
модель выбирает показ
→ пользователь реагирует только на показанное
→ реакция становится label
→ новая модель усиливает прежний выбор
```

Feedback loop не всегда плох: персонализация должна учиться. Опасность — отсутствие exploration и незаметное закрепление ошибки.

## Cold start

### Новый пользователь

Нет истории. Baselines:

- популярное по региону/контексту;
- onboarding preferences;
- session-based signals;
- contextual bandits;
- content-based similarity.

### Новый объект

Нет взаимодействий. Используют контентные признаки, embeddings, продавца/автора, controlled exploration и freshness boost.

Нельзя оценивать cold-start только на случайном split истории: объекты будущего могли попасть в train. Нужен временной split и отдельные cold slices.

## Offline-online gap

Причины, почему NDCG offline растёт, а online outcome падает:

1. **Bias labels:** обучение и тест используют клики старой policy.
2. **Не та цель:** NDCG по кликам, бизнесу важны покупки или долгосрочная ценность.
3. **Distribution shift:** online queries, inventory и пользователи свежее датасета.
4. **Latency:** тяжёлая модель отвечает медленнее, люди уходят.
5. **Serving mismatch:** признаки online считаются иначе или запаздывают.
6. **UI interaction:** новые результаты выглядят иначе, метрика этого не знает.
7. **Diversity/coverage:** персональная точность растёт, каталог схлопывается.
8. **Market equilibrium:** рекомендации перераспределяют спрос и исчерпывают supply.
9. **Leakage:** offline features содержали будущее.
10. **Metric aggregation:** улучшился head, ухудшился важный tail.

## Selection bias в human labels

Даже экспертная разметка не автоматически нейтральна:

- оценивают только pooled top-results нескольких систем;
- запрос без контекста пользователя неоднозначен;
- annotator не знает локальные предпочтения;
- label guideline отличается от реального успеха;
- disagreements скрываются majority vote.

Нужны representative sampling, blind randomization, контроль качества, adjudication и отдельный анализ disagreement.

## Exploration vs exploitation

**Exploitation:** показывать то, что сейчас кажется лучшим — максимизирует краткосрочную награду.

**Exploration:** иногда показывать менее уверенные объекты, чтобы узнать их качество и не застрять в старой policy.

Подходы:

- epsilon-greedy;
- Thompson sampling;
- UCB;
- randomized buckets;
- explore slots;
- interleaving.

Exploration имеет пользовательскую цену, поэтому ограничивается безопасными кандидатами и budget.

## Interleaving

Результаты двух rankers смешиваются в одной выдаче, затем по кликам определяется предпочтение. Это чувствительнее A/B на CTR, потому что пользователь сравнивает системы в одной сессии.

Но interleaving лучше измеряет относительное предпочтение ranking, а не долгосрочные бизнес-эффекты; он требует корректного credit assignment и всё равно подвержен examination patterns.

## Пример: модель подняла CTR и снизила GMV

Новый ranker сильнее использует яркость изображения. Кликабельные дешёвые товары поднялись вверх. CTR +5%, add-to-cart без изменений, purchase per click −8%, AOV −12%, GMV −10%.

Модель честно оптимизировала label, но label был неполной proxy. Исправления:

- обучаться на multi-task/downstream labels;
- учитывать expected value и purchase propensity;
- добавить calibration и business constraints;
- контролировать AOV/GMV и returns online;
- анализировать position-normalized clicks;
- не забывать diversity и seller constraints.

## Как ответить на интервью

> [!question] Почему клик не является ground truth релевантности?
> Для клика объект должен быть показан и замечен; на него влияют позиция, сниппет, цена и UI. Непоказанные объекты имеют неизвестный, а не отрицательный label. Поэтому clicks — biased implicit feedback. Я использую exploration/propensity correction, human labels и обязательно online product metrics.

> [!question] Как проверить новый ranker offline?
> Временной held-out split, representative query set, top-K metrics и slices по intent/head-tail/cold-start. Проверю, как получены labels и покрывает ли pool новые candidates, latency и feature parity. Offline результат служит gate, затем нужен interleaving или A/B с downstream outcome и guardrails.

> [!question] Что такое feedback loop?
> Модель определяет exposure, exposure определяет собираемые labels, а labels обучают следующую модель. Без exploration система усиливает популярность и собственные ошибки. Мониторю coverage и exposure, создаю exploration budget и храню propensity/logging policy.

> [!question] Как оценить эффект рекомендаций, если товары ограничены по запасу?
> User-level independence нарушается: treatment может исчерпать inventory для control. Нужен дизайн по рынкам/времени, supply-aware metrics и анализ equilibrium; простой user A/B может недооценить или исказить общий эффект.

## Типичные ошибки

- Считать непоказанный объект отрицательным.
- Учиться и оцениваться на clicks старой policy без коррекции.
- Рандомно делить строки по времени и допускать leakage.
- Оптимизировать только CTR.
- Не логировать propensity, позицию и candidate set.
- Игнорировать supply-side и network effects.

## Проверка себя

- Как position bias попадает в target?
- Почему IPS уменьшает bias, но может увеличить variance?
- Назовите пять причин offline-online gap.
- Какие guardrails защищают от popularity loop?

## Связано

- [[Метрики поиска и рекомендаций — CTR, Success Rate, Precision@K, Recall@K, MRR и NDCG]]
- [[Корреляция и причинность — confounders, selection bias и парадокс Симпсона]]
- [[A-B тест от гипотезы до решения — единица рандомизации, метрики и вердикт]]
- [[Мониторинг продукта и ML-системы — алерты, срезы, drift и delayed labels]]

---
[[Аналитика/🗺️ Индекс|Назад к индексу аналитики]]
