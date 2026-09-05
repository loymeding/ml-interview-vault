---
tags: [аналитика, интервью, кейсы, практика, product-sense, ml-system-design]
тип: практика
уровень: middle
сложность: высокая
статус: готово
готовность: 95
создано: 2026-08-26
источники:
  - "Yandex Jobs — Секции собеседований для аналитиков"
  - "Machine Learning System Design — Babushkin, Kravchenko"
предпосылки:
  - "Как решать аналитический кейс на интервью"
связано:
  - "Разобранный кейс — в поиске вырос CTR, но снизились покупки"
  - "Аналитический каркас ML System Design — цель, данные, эксперимент и решение"
сравнить-с: []
---

# Банк аналитических и ML System Design кейсов

> [!abstract] Суть
> Эти кейсы нужно решать вслух, не читая подсказку. На первый проход отводите 15–20 минут, на второй — 8–10 минут. Цель — не угадать ответ, а провести интервьюера по устойчивой логике.

## Правила тренировки

1. Прочитайте только условие.
2. За минуту назовите структуру ответа.
3. Задайте не больше трёх первых уточнений.
4. Зафиксируйте assumptions и продолжайте.
5. Выберите primary, guardrails, diagnostics и slices.
6. Предложите данные, проверку и решение.
7. Только после ответа откройте блок «Что должно прозвучать».

Оценивайте себя по шести критериям от 0 до 2:

| Критерий | 0 | 1 | 2 |
|---|---|---|---|
| Структура | поток идей | частичный план | ясный каркас и резюме |
| Продукт | только локальная метрика | названа цель | пользователь, бизнес и trade-offs |
| Метрики | список | primary есть | иерархия, формулы, unit и maturity |
| Данные | «возьмём логи» | базовые таблицы | grain, bias, quality, slices |
| Причинность | before/after | A/B без деталей | правильный design и assumptions |
| Решение | анализа ради анализа | общий вывод | decision rule, rollout и monitoring |

Максимум — 12. Для уверенного интервью ориентир — 9–10 без помощи.

> [!tip] Подробный тренировочный разбор
> Кейс о программе лояльности разобран отдельной карточкой в формате полноценного диалога с интервьюером: [[Разборы кейсов/Аналитика и эксперименты/Разбор кейса 409 — программа лояльности.md|кейс 409 — корреляция или причинность]].

> [!note] Расширенные разборы всех 18 кейсов
> Для каждого задания ниже есть подробный разбор с уточняющими вопросами, допущениями, несколькими гипотезами, метриками, вариантами эксперимента и логикой ответа: [[Полные разборы 18 аналитических и ML System Design кейсов]].

---

## Кейс 1. Ресторан: скорость или рейтинг

Сейчас сервис доставки ранжирует рестораны в основном по ETA. Продакт предлагает сортировать по рейтингу, считая, что это повысит удовлетворённость. Как проверить и принять решение?

**Дополнительные условия от интервьюера:** часть ресторанов быстро перегружается; у новых ресторанов мало отзывов; рейтинг и ETA зависят от района.

> [!check]- Что должно прозвучать
> Пользовательская цель; completed orders/long-term value как primary; ETA, cancellations и complaints как guardrails; rating reliability/cold start; supply interference; user A/B против geo-time switchback; multi-objective ranking вместо бинарного выбора.

## Кейс 2. CTR поиска вырос, покупки снизились

После релиза поискового ранжировщика CTR вырос на 7%, а покупки на поискового пользователя упали на 4%. Ваши действия?

**Усложнение:** NDCG@10 offline тоже выросла; изменение сильнее на популярных запросах.

> [!check]- Что должно прозвучать
> Проверка instrumentation и experiment validity; декомпозиция `sessions × CTR × cart/click × purchase/cart`; position bias; label mismatch; AOV/availability/price; head-tail mix; latency; не считать CTR целью; решение через rollback/segment rollout и новый objective.

## Кейс 3. Рекомендации подняли GMV, но ухудшили retention

В A/B-рекомендательный блок дал +3% GMV за неделю, D30 retention treatment ниже на 1.2 п.п. Что делать?

**Усложнение:** D30 ещё созрел только у половины пользователей; блок часто показывает дорогие популярные товары.

> [!check]- Что должно прозвучать
> Не смешивать зрелые и незрелые когорты; uncertainty и pre-specified guardrail; краткосрочный revenue vs долгосрочная ценность; returns/complaints/diversity; сегменты; LTV simulation; holdout; возможно, ограничить frequency/popularity.

## Кейс 4. Бот закрывает больше обращений, но жалоб больше

Новая LLM-версия увеличила automation rate с 45% до 60%, а жалобы — с 1% до 1.6%. Команда рада снижению нагрузки операторов. Как оценивать релиз?

**Усложнение:** корректность ответа размечается только на 500 диалогах в неделю, repeat contact созревает 7 дней.

> [!check]- Что должно прозвучать
> Automation rate как proxy, а не конечная цель; resolved without repeat contact; safety/unsupported claims; stratified human sample; answerable/unanswerable slices; delayed labels; cost операторов vs cost ошибки; abstention/escalation threshold; staged rollout/fallback.

## Кейс 5. Определить лучшие категории бизнеса

Вам дали продажи маркетплейса. Нужно выбрать три категории, в которые стоит инвестировать в следующем квартале. Как подойдёте?

**Усложнение:** одна категория быстро растёт из-за краткой акции, другая имеет высокий GMV и отрицательную маржу, третья — низкий объём и высокий retention.

> [!check]- Что должно прозвучать
> Не ранжировать только по GMV; revenue/margin, growth decomposition, repeat rate/LTV, CAC, supply, seasonality, concentration, returns; confidence для малых категорий; сценарии и стратегические constraints.

## Кейс 6. Сколько пушей отправлять

Пользователи с пятью пушами в неделю покупают вдвое чаще пользователей без пушей. Продакт хочет всем отправлять пять. Что скажете?

**Усложнение:** пуши назначаются моделью intent, unsubscribe приходит с задержкой.

> [!check]- Что должно прозвучать
> Confounding/reverse causality; treatment policy; frequency experiment; user-level randomization; incremental purchases/margin; unsubscribe, disable notifications, churn guardrails; heterogeneous/uplift effect; delayed outcome.

## Кейс 7. Повышение цены подписки

Нужно оценить повышение цены premium на 15%. Какие метрики и эксперимент?

**Усложнение:** пользователи обсуждают цену, есть годовые и месячные тарифы, billing cycles различаются.

> [!check]- Что должно прозвучать
> Revenue/margin/LTV, conversion и churn; maturity по billing cycle; grandfathering; interference/communication; сегменты willingness-to-pay; price elasticity; ethical/legal constraints; cohort-based analysis, не краткий CTR.

## Кейс 8. Fraud precision вырос, бизнес недоволен

У fraud-модели precision вырос с 40% до 55%, recall снизился с 70% до 45%. Команда ML считает модель лучше, финансовая команда — хуже. Кто прав?

**Усложнение:** ручная проверка ограничена 10 000 кейсов в день; fraud amounts имеют тяжёлый хвост.

> [!check]- Что должно прозвучать
> Operating point и capacity; weighted loss по amount; expected cost FP/FN/review; PR curve и recall at review budget; calibration; сегменты; delayed chargebacks; triage zones; обе команды могут быть правы относительно разных objectives.

## Кейс 9. Offline NDCG выросла, A/B отрицательный

Новая рекомендационная модель стабильно лучше на offline NDCG@20, но два A/B-теста показывают падение purchase rate. Назовите план расследования.

**Усложнение:** offline test сформирован из кликов старой модели; новая модель в два раза медленнее.

> [!check]- Что должно прозвучать
> Policy/position bias, exposure coverage, target mismatch clicks vs purchases, latency, feature parity, temporal split, candidate distribution, cold start, A/B SRM/exposure, UI/adoption, diversity/supply, error analysis. Повторять третий A/B без изменения гипотезы не надо.

## Кейс 10. Прогноз спроса улучшился по MAE, дефицит вырос

MAE demand forecast снизилась на 8%, но out-of-stock rate вырос. Почему и что изменить?

**Усложнение:** ошибки на пиковых днях редки, но особенно дороги.

> [!check]- Что должно прозвучать
> MAE усредняет; bias/underforecast, tail и horizon slices; quantile loss/service level; lead time; inventory policy после прогноза; promo/holiday features; business simulation cost overstock vs stockout; temporal backtest.

## Кейс 11. Упала конверсия оплаты

За ночь checkout → paid conversion упала с 88% до 63%. Как расследовать?

**Усложнение:** падение видно только на iOS, новая версия вышла вчера, server payment count снизился лишь на 5%.

> [!check]- Что должно прозвучать
> Определение и числитель/знаменатель; client vs server source; duplicate checkout/missing success event; app version, payment method, error code; rollout timeline; raw journeys; не делать продуктовый вывод до проверки instrumentation; rollback SDK/релиза.

## Кейс 12. Комиссия продавцов

Маркетплейс повысил комиссию в одном регионе. GMV не изменился за две недели. Было ли решение нейтральным?

**Усложнение:** часть продавцов повысила цены, часть сократила ассортимент; соседний регион без изменения похож по тренду.

> [!check]- Что должно прозвучать
> Revenue и margin vs GMV; seller churn, price, assortment, buyer CR; delayed effects; DiD с pre-trends; spillover между регионами; heterogeneous sellers; market equilibrium; два недельных среза могут быть недостаточны.

## Кейс 13. Cold start рекомендаций

Спроектируйте рекомендации для новых пользователей музыкального сервиса без истории.

**Усложнение:** onboarding нельзя делать длиннее 20 секунд, контент обновляется ежедневно.

> [!check]- Что должно прозвучать
> Context/popularity/editorial baseline; короткий preference elicitation; session signals; exploration; content embeddings; primary meaningful listening/return, не только click; skip guardrail; catalog coverage, novelty; cold-start offline split и online experiment.

## Кейс 14. Оценка RAG-помощника операторов

Нужно решить, можно ли выпустить RAG-помощника, предлагающего оператору черновик ответа. Как построите evaluation?

**Усложнение:** база знаний меняется ежедневно, оператор может отредактировать ответ, часть запросов не имеет ответа в документах.

> [!check]- Что должно прозвучать
> Retrieval и generation eval отдельно; answerability; Recall@K/MRR, faithfulness/correctness/completeness; operator acceptance/edit distance/time saved; unsupported claim/safety; freshness index; human rubric и inter-rater; shadow/pilot; leakage; feedback labels biased acceptance.

## Кейс 15. Новый onboarding и retention

После упрощения регистрации новых пользователей стало на 25% больше, D7 retention зарегистрированных снизился с 24% до 20%. Успех или провал?

**Усложнение:** число D7 active в абсолюте выросло, канал трафика тоже изменился.

> [!check]- Что должно прозвучать
> Считать абсолюты и rates; activation/meaningful action; cohort maturity; composition/channel mix; experimental assignment; downstream D30/LTV; качество привлечённой аудитории; цель onboarding — не максимальная регистрация.

## Кейс 16. Метрика поддержки по операторам

У операторов команды A средний handle time ниже команды B, но CSAT тоже ниже. Руководитель хочет премировать A за скорость. Как оценить справедливо?

**Усложнение:** сложные обращения маршрутизируются опытным операторам B.

> [!check]- Что должно прозвучать
> Case-mix/confounding; resolution и repeat contact; severity/intent; median/tail; quality-adjusted throughput; routing policy; hierarchical adjustment; gaming; не превращать handle time в единственную target.

## Кейс 17. Рекламная модель и аукцион

Новая CTR-модель лучше калибрована и повышает ожидаемый revenue аукциона offline. Какие риски перед запуском?

**Усложнение:** изменение score влияет на позиции, цены и будущие обучающие данные.

> [!check]- Что должно прозвучать
> Calibration by segment, auction simulation, advertiser/user guardrails, position bias, feedback loop, exploration, budget pacing, latency, marketplace equilibrium, A/B randomization and long-term advertiser retention.

## Кейс 18. Аномалия DAU

DAU упал на 10% в понедельник утром. Опишите первые 30 минут расследования.

**Усложнение:** падение глобальное, но server requests стабильны; Android SDK обновился ночью.

> [!check]- Что должно прозвучать
> Definition active event; freshness/ingestion/schema; platform/version; server-side source; numerator only; same-weekday baseline; raw event comparison; likely logging issue; communication and temporary dashboard annotation before product alarm.

---

## Промпт для тренировки с ИИ

```text
Ты — строгий интервьюер на позицию аналитика-разработчика / Data Scientist,
проводящий секцию «Аналитический кейс» или «ML System Design».

Выбери один реалистичный кейс из сфер поиска, рекомендаций, маркетплейса,
доставки, поддержки, рекламы, antifraud или ML-продукта. Не раскрывай решение
и не перечисляй нужные метрики заранее.

Правила интервью:
1. Дай только исходное условие и дождись моих уточняющих вопросов.
2. Отвечай только на то, что я спросил. Если вопрос неоднозначен,
   заставь меня зафиксировать assumption.
3. Не поддакивай. Если я называю метрику без unit, denominator, окна или
   связи с целью, попроси уточнить.
4. Проверяй: продуктовую цель, primary/guardrail/diagnostic metrics,
   данные и grain, bias и качество данных, причинный дизайн, статистическую
   неопределённость, срезы, риски и финальное решение.
5. По ходу кейса добавь 2–3 реалистичных ограничения: delayed labels,
   interference, низкий трафик, конфликт метрик, поломка логирования,
   offline-online gap или ограничение latency/cost.
6. Если я перескакиваю к модели, верни меня к decision, target и baseline.
7. Если я перечисляю гипотезы, попроси приоритизировать и назвать проверяемое
   наблюдение для первой гипотезы.
8. Не давай подсказку, пока я явно не скажу «нужна подсказка».

После слов «я закончил»:
- оцени от 0 до 2: структуру, продуктовую логику, метрики, данные,
  причинность/эксперимент, решение и коммуникацию;
- укажи три сильных момента и три главных пробела;
- задай два добивающих вопроса уровня middle+/senior;
- покажи эталонный ответ, который можно произнести за 5–7 минут;
- отдельно перечисли assumptions, которые в эталоне нельзя выдавать за факты.

Начни с кейса средней сложности. Не используй кейс, который уже был в этой
сессии. Пиши по-русски и веди диалог по одному вопросу за раз.
```

## Режим повторения

- День 1: кейсы 1, 4, 11 — продукт, ML и диагностика.
- День 2: кейсы 6, 8, 12 — причинность, threshold, квазиэксперимент.
- День 3: кейсы 2, 9, 14 — ranking/RAG и offline-online gap.
- День 4: кейсы 3, 7, 15 — long-term metrics и maturity.
- День 5: кейсы 5, 10, 16 — экономика, forecasting, case-mix.
- День 6: кейсы 13, 17, 18 — cold start, marketplace effects, incident response.

## Проверка себя

- Можете ли вы за 60 секунд дать структуру любого кейса?
- В каждом ли ответе есть решение, а не только анализ?
- Умеете ли вы назвать допущение, на котором держится causal conclusion?
- Можете ли вы объяснить, как метрику можно «улучшить», ухудшив продукт?

## Связано

- [[Как решать аналитический кейс на интервью]]
- [[Разобранный кейс — в поиске вырос CTR, но снизились покупки]]
- [[Аналитический каркас ML System Design — цель, данные, эксперимент и решение]]

## Источники

- [Секции собеседований для аналитиков — Яндекс](https://yandex.ru/jobs/interview/analytics).
- Machine Learning System Design — Valerii Babushkin, Arseny Kravchenko.

---
[[Разборы кейсов/🗺️ Индекс|Назад к индексу кейсов]]
