---
tags: [аналитика, ml-metrics, classification, regression, calibration, threshold, offline-online]
тип: теория
уровень: middle
сложность: высокая
статус: готово
готовность: 95
создано: 2026-08-25
источники:
  - "Data Science for Business — Provost, Fawcett"
  - "Designing Machine Learning Systems — Chip Huyen"
предпосылки:
  - "Система продуктовых метрик — North Star, target, guardrail, proxy и дерево метрик"
связано:
  - "Метрики поиска и рекомендаций — CTR, Success Rate, Precision@K, Recall@K, MRR и NDCG"
  - "Мониторинг продукта и ML-системы — алерты, срезы, drift и delayed labels"
сравнить-с: []
---

# ML-метрики в продукте — от offline score к бизнес-решению

> [!abstract] Суть
> Offline-метрика описывает определённое свойство предсказаний на конкретном датасете. Бизнес-решение появляется только после выбора operating point, цены ошибок, проверки срезов, калибровки и online-эффекта.

## Почему «у модели F1 = 0.87» недостаточно

Нужно узнать:

- какой класс положительный;
- каков baseline и prevalence;
- на каком пороге посчитана метрика;
- как разделены train/test;
- какие ошибки дороже;
- как метрика ведёт себя по сегментам;
- что делает продукт с предсказанием;
- изменилась ли пользовательская или бизнес-метрика online.

Модель — компонент решения. Одинаковый score может быть полезен в triage для оператора и опасен в полностью автоматическом отказе пользователю.

## Матрица ошибок

| | Предсказали + | Предсказали − |
|---|---:|---:|
| Истина + | TP | FN |
| Истина − | FP | TN |

Метрики:

$$\Large Precision=\frac{TP}{TP+FP}$$

$$\Large Recall=\frac{TP}{TP+FN}$$

$$\Large Specificity=\frac{TN}{TN+FP}$$

$$\Large F1=2\cdot\frac{Precision\cdot Recall}{Precision+Recall}$$

### Как выбирать

- Высокий **recall**, если пропустить positive дорого: опасный контент, fraud candidate generation, медицинский screening.
- Высокий **precision**, если каждое срабатывание дорого: ручная проверка, блокировка честного пользователя, дорогой вызов внешнего API.
- **F1** полезен как компактный баланс, но предполагает особую симметрию precision и recall и игнорирует TN. Бизнес-стоимости лучше задавать явно.

## Threshold и expected cost

Модель выдаёт score, а не готовое бизнес-решение. Порог определяет confusion matrix.

$$\Large ExpectedCost(t)=C_{FP}\cdot FP(t)+C_{FN}\cdot FN(t)+C_{review}\cdot N_{review}(t)$$

Например, fraud:

- пропущенный fraud стоит в среднем 10 000 ₽;
- ручная проверка — 100 ₽;
- ложная блокировка имеет оценённую стоимость 1 500 ₽ через поддержку и churn.

Выбираем threshold не по максимальному F1 автоматически, а по минимальному ожидаемому cost при capacity и policy constraints. Часто есть три зоны: approve, manual review, decline.

## Accuracy

$$\Large Accuracy=\frac{TP+TN}{N}$$

При fraud rate 1% модель «всегда честный» имеет accuracy 99%, но не ловит ничего. Accuracy полезна при сбалансированных классах и близкой цене ошибок; в иных случаях её нужно дополнять.

## ROC-AUC и PR-AUC

ROC-AUC — вероятность, что случайный positive получит score выше случайного negative. Она оценивает ranking без фиксированного порога.

PR-AUC фокусируется на positive class и при сильном дисбалансе лучше отражает purity найденных positives. Baseline PR-AUC связан с prevalence, поэтому сравнение между датасетами требует осторожности.

Но высокий AUC не выбирает operating point и не гарантирует хорошую калибровку. Две модели с одинаковым ROC-AUC могут давать разную экономику на нужном FPR.

## Калибровка

Если модель выдаёт 0.8, хотим, чтобы среди объектов с таким score примерно 80% были positive.

Ranking и calibration — разные свойства. Монотонное преобразование score не меняет ROC-AUC, но ломает вероятностный смысл.

Калибровка важна для:

- expected value и risk-based decisions;
- объединения вероятности с ценой действия;
- capacity planning;
- понятных порогов между сегментами;
- мониторинга drift.

Метрики: Brier score, log loss, calibration curve, ECE. Калибровку проверяют по сегментам и времени.

## Метрики регрессии

$$\Large MAE=\frac{1}{n}\sum|y_i-\hat y_i|$$

MAE легко интерпретировать в единицах target и устойчивее к хвосту.

$$\Large RMSE=\sqrt{\frac{1}{n}\sum(y_i-\hat y_i)^2}$$

RMSE сильнее штрафует крупные ошибки. Подходит, когда катастрофическая ошибка непропорционально дороже.

$$\Large MAPE=\frac{100\%}{n}\sum\left|\frac{y_i-\hat y_i}{y_i}\right|$$

MAPE понятна в процентах, но ломается при $y=0$ и чрезмерно штрафует ошибки на малых значениях. Для спроса с нулями часто используют WAPE, MAE/quantile loss и отдельный анализ zeros.

### Mean error / bias

Средняя подписанная ошибка показывает систематическое завышение или занижение:

$$\Large Bias=\frac{1}{n}\sum(\hat y_i-y_i)$$

MAE может быть стабильной, но forecast систематически недооценивает спрос — бизнес теряет наличие.

## Quantile loss

Если цена недопрогноза и перепрогноза разная, предсказывают квантиль. Для $\tau$:

$$\Large L_{\tau}(y,\hat y)=\max\big(\tau(y-\hat y),(\tau-1)(y-\hat y)\big)$$

$\tau=0.9$ даёт прогноз, выше которого target должен оказаться примерно в 10% случаев. Это полезно для capacity и запасов.

## Метрика должна соответствовать единице решения

Если модель предсказывает риск заказа, а решение принимается по пользователю, event-level split и metric могут вводить в заблуждение. Heavy user создаёт много строк и доминирует.

Нужно согласовать:

- prediction unit;
- decision unit;
- randomization unit;
- aggregation unit метрики.

## Validation split

Случайный split опасен, если:

- один пользователь попадает в train и test;
- есть временная зависимость;
- объекты каталога повторяются;
- feature использует будущее;
- target delayed.

Для production чаще нужен temporal out-of-time test, group split по пользователю/объекту и отдельные cold-start slices.

## Slices и worst-group quality

Средняя метрика может скрыть провал:

- новый пользователь;
- редкий intent;
- конкретный язык/регион;
- новая категория;
- low-end device;
- высокая нагрузка;
- protected/safety group.

Срезы выбирают из product risk и механизма модели, а не только после поиска отрицательных цифр. Для ключевых сегментов задают acceptance criteria.

## Offline → online

Offline metric нужна для быстрых итераций и gating. Online проверяет полный causal effect системы:

```text
offline quality
+ serving correctness
+ latency
+ UI
+ user adaptation
+ market effects
= online product outcome
```

Рост offline metric может не дойти до продукта из-за latency, stale features, неправильного threshold, mismatch labels и target, feedback loops или слабого adoption.

## Пример: автоматическая модерация

Задача — блокировать опасный контент.

- Candidate filter: высокий recall, чтобы не пропустить опасные случаи.
- High-confidence auto-block: высокий precision, чтобы не блокировать нормальный контент.
- Gray zone: ручная проверка в пределах capacity.
- Offline: PR-AUC, recall при заданном precision, calibration, slices.
- Product: incidents, appeals, moderator load, decision latency.
- Guardrails: false blocks, creator churn, cost.

Так один model score превращается в policy с несколькими operating points.

## Как ответить на интервью

> [!question] Какая ML-метрика важнее?
> Она определяется ценой ошибок и местом модели в pipeline. Я начну с decision policy: что происходит при positive/negative prediction, есть ли manual review и capacity. Затем выберу metric at operating point и ranking/calibration metrics для разработки, обязательно добавив slices и online product outcome.

> [!question] ROC-AUC выросла, F1 упал. Как возможно?
> ROC-AUC оценивает порядок по всем порогам, F1 — один выбранный threshold. Новый score может ранжировать лучше, но старый threshold больше не подходит или calibration изменилась. Нужно перенастроить operating point на validation set и оценить cost.

> [!question] Почему нельзя выбирать threshold на test?
> Тогда test участвует в оптимизации и оценка становится оптимистичной. Threshold выбираю на validation по business objective, test оставляю для финальной unbiased оценки.

> [!question] Почему модель с меньшим MAE может быть хуже?
> Ошибки могут быть сосредоточены в дорогих сегментах, иметь неправильный знак или нарушать SLA. Например, небольшой средний MAE, но сильный недопрогноз пикового спроса. Нужны weighted/quantile metrics и business simulation.

## Типичные ошибки

- Выбирать F1 без цены FP/FN.
- Считать AUC готовой policy.
- Игнорировать calibration.
- Оптимизировать MAPE при нулевых target.
- Делать случайный row split при повторяющихся users.
- Смотреть только average metric.
- Объявлять offline gain продуктовым эффектом.

## Проверка себя

- Как выбрать threshold fraud-модели при лимите ручной проверки?
- Чем ranking и calibration отличаются?
- Когда MAE лучше RMSE?
- Назовите причины, по которым offline gain исчезает online.

## Связано

- [[Classic Machine Learning/Метрики/Метрики бинарной классификации|Метрики бинарной классификации]]
- [[Classic Machine Learning/Метрики/Выбор оптимального порога классификации|Выбор порога]]
- [[Classic Machine Learning/Модели/Калибровка моделей (Platt Scaling, Isotonic Regression)|Калибровка моделей]]
- [[Метрики поиска и рекомендаций — CTR, Success Rate, Precision@K, Recall@K, MRR и NDCG]]

---
[[Аналитика/🗺️ Индекс|Назад к индексу аналитики]]

