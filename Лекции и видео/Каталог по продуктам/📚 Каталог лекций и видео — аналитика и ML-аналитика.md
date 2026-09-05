---
type: resource_catalog
тип: каталог
section: Аналитика
source_type: video-catalog
category: "Аналитика, ML и GenAI"
product_contexts:
  - "Яндекс Практикум"
  - "Яндекс Карты"
  - "Яндекс Такси / Яндекс Про"
  - "Яндекс Маркет"
  - "Яндекс Музыка"
  - "Яндекс Поиск"
  - "Алиса / Alice AI"
  - "YandexART"
  - "Ozon и другие e-commerce-продукты"
status: curated
updated: 2026-08-26
---

# Видеоматериалы и лекции — аналитика и ML-аналитика

Это не просто список ссылок. Здесь материалы разложены по тому, **какой навык они тренируют**, насколько близки к предполагаемому интервью и в каком порядке их лучше смотреть.

Главная идея подготовки такая: на ML System Design недостаточно знать название метрики или формулу. Нужно уметь связать цепочку:

> бизнес-цель → пользовательское поведение → данные → метрика → эксперимент/метод оценки → ограничение → решение.

## Как читать приоритеты

- **Must** — сначала, если времени мало. Материал прямо закрывает пробелы для интервью.
- **High** — сильное углубление и реальные продуктовые примеры.
- **Optional** — полезно после ядра или для конкретного кейса.

По происхождению источника:

- **P1** — первичный материал: Яндекс, Ozon, Airbnb, Booking, Uber, Netflix, официальный образовательный канал.
- **P2** — сильный экспертный или конференционный материал.
- **P3** — вторичное объяснение или тренировочное собеседование.

## Быстрый маршрут, если до интервью 7–10 дней

1. **База продуктовой аналитики:** [Яндекс Практикум — метрики и A/B-тесты](https://www.youtube.com/watch?v=6vX7divNBRA) и [углубление](https://www.youtube.com/watch?v=2gvf__7_X2c).
2. **Метрики сложного продукта:** [интегральные метрики качества в Яндекс Картах](https://www.youtube.com/watch?v=ugyk4MxlCWU) и [чувствительные метрики для глобальных KPI](https://www.youtube.com/watch?v=8kPzwqDUk5o).
3. **Эксперименты:** [A/B-тесты как способ развития продукта](https://www.youtube.com/watch?v=gMx-juYkNCw), [свитчбэки в Такси](https://www.youtube.com/watch?v=0c8hxTMOZYg), [эффективность против Retention в Яндекс Про](https://www.youtube.com/watch?v=CGCSi-z_rnk).
4. **Поиск и рекомендации:** [рекомендации и ML](https://www.youtube.com/watch?v=-JepzjMJ_cU), [обучение ранжированию](https://www.youtube.com/watch?v=JTW9STt4_TE), [offline–online gap в RecSys](https://www.youtube.com/watch?v=rjGGSHhKDMM).
5. **ML-аналитика:** [что такое ML-аналитика и как оценивать LLM-продукты](https://www.youtube.com/watch?v=l7rNXP1WJdQ), [качество моделей и метрики для LLM](https://www.youtube.com/watch?v=peBvWuCpH_Y), [RAG: архитектуры и качество](https://www.youtube.com/watch?v=-efHhXHhNys).
6. **Репетиция ответа:** [ETA ML System Design](https://www.youtube.com/watch?v=l-2m6YXlEjM) или [ranking model mock interview](https://www.youtube.com/watch?v=7_E4wnZGJKo).

После каждого ролика не пересказывай содержание. Составь короткий ответ в формате: «какую задачу решали, почему простая метрика не подходила, как проверяли причинный эффект, какие были guardrails и какое решение приняли».

---

## 1. Продуктовые метрики и аналитическое мышление

| Приоритет | Источник | Что забрать для интервью |
|---|---|---|
| **Must · P1** | [Погружаемся в продуктовую аналитику: метрики и A/B-тесты](https://www.youtube.com/watch?v=6vX7divNBRA) — Яндекс Практикум, 1:34:34 | Engagement, retention, LTV, proxy-метрики, смысл A/B-теста, p-value, peeking и data-driven подход. Смотреть не подряд: 12:01–40:39 и 40:39–1:07:36. |
| **Must · P1** | [Продуктовая аналитика: углубляемся в метрики и A/B-тесты](https://www.youtube.com/watch?v=2gvf__7_X2c) — Яндекс Практикум, 1:25:45 | Профессия аналитика, взаимосвязь метрик, воронка, аномалии, Data Driven и полный жизненный цикл эксперимента. Особенно полезны 09:48–20:09 и 33:55–59:39. |
| **Must · P1** | [Дайте мне число: интегральные метрики качества и эффективности в Яндекс Картах](https://www.youtube.com/watch?v=ugyk4MxlCWU) — Тимофей Струнков, Yandex for Analytics | Как собрать сложное качество продукта в устойчивую интегральную метрику, не потеряв связь с бизнесом и процессом. Это хороший материал для вопросов «какую одну метрику выберете?» и «что делать, если метрик слишком много?». Описание и дата подтверждены страницей [Яндекс Образования](https://education.yandex.ru/knowledge/daite-mne-chislo-kak-mi-postroili-integralnie-metriki-kachestva-i-effektivnosti-v-yandeks-kartakh). |
| **Must · P1** | [Как подобрать чувствительные метрики для глобальных KPI](https://www.youtube.com/watch?v=8kPzwqDUk5o) — Олег Хомюк, Yandex for Analytics | Sensitivity, связь локальных и глобальных KPI, выбор метрики, на которой действительно виден эффект. |
| **High · P1** | [Продуктовая аналитика по шагам: инструкция от Яндекс.Метрики и Yandex.Cloud](https://www.youtube.com/watch?v=fDeqtFRawdo) | Приземлённое прохождение от вопроса бизнеса до отчёта и решения. Хорошо для повторения базового рабочего процесса аналитика. |
| **High · P1** | [Продуктовая аналитика и выбор метрик](https://www.youtube.com/watch?v=qO63yet5wTo) — Александр Сергеев, Analytics Day | Короткая лекция именно про выбор метрик, а не про набор формул. |
| **High · P2** | [Продуктовые процессы для роста Retention](https://www.youtube.com/watch?v=rWSS8yfli5U) — Андрей Законов | Как думать о retention как о процессе и продуктовой системе, а не как об одной красивой кривой. |
| **Optional · P3** | [ПРОДУКТ в IT: как рассчитать основные метрики](https://www.youtube.com/watch?v=JGVd_a3nVbE) | Быстрый повтор терминов перед практикой; не заменяет первичные материалы. |

Связанные карточки: [[Система продуктовых метрик — North Star, target, guardrail, proxy и дерево метрик]], [[Воронка и конверсия — от события до диагностики потерь]], [[Retention, churn и когортный анализ]], [[Декомпозиция метрик и поиск причин изменений]], [[Монетизация и юнит-экономика — GMV, revenue, AOV, ARPU, ARPPU, CAC и LTV]].

### Как работать с блоком

После первых двух роликов ответь вслух на три вопроса:

1. Почему «рост вовлечённости» не обязательно означает пользу для пользователя?
2. Как отличить North Star metric от удобной, но бесполезной proxy-метрики?
3. Что делать, если локальная метрика растёт, а глобальный KPI не меняется?

---

## 2. A/B-тесты, причинность и эксперименты

| Приоритет | Источник | Что забрать для интервью |
|---|---|---|
| **Must · P1** | [A/B тесты как способ развития продукта](https://www.youtube.com/watch?v=gMx-juYkNCw) — Yandex for Products, 2:32:28 | Большой разбор полного цикла: гипотеза, дизайн, рандомизация, метрики, статистика и решение. Можно смотреть фрагментами как справочник. |
| **Must · P1** | [A/B тесты и как мы их готовим](https://www.youtube.com/watch?v=TpUflOQo1kI) — Станислав Гафаров, Yandex Education, 31:17 | Короткий практический шаблон подготовки эксперимента. |
| **Must · P1** | [Свитчбэк-эксперименты и гипотезы в Яндекс Такси](https://www.youtube.com/watch?v=0c8hxTMOZYg) — Алексей Чубуков, 31:18 | Почему обычный user-level A/B ломается в marketplace/такси, что рандомизировать по времени или географии, как думать о interference. Описание также есть в [каталоге Яндекс Образования](https://education.yandex.ru/knowledge?_rsc=23hwu&page=29). |
| **Must · P1** | [Эксперименты в Яндекс Про: баланс эффективности и Retention](https://www.youtube.com/watch?v=CGCSi-z_rnk) — Игорь Рубанов, 15:19 | Почему краткосрочный рост acceptance/completion не закрывает вопрос решения; как DSAT может быть ведущим сигналом долгосрочного retention. Подтверждение содержания — [страница доклада](https://education.yandex.ru/knowledge/eksperimenti-v-yandeks-pro-balans-effektivnosti-i-retention). |
| **Must · P1** | [Формула доверия: CI для Ratio- и Uplift-метрик](https://www.youtube.com/watch?v=0yhbw9QV9Z4) — Диля Хакимова | Что делать с ratio-метриками, серыми результатами и интервалом неопределённости. Официальное описание: [Яндекс Образование](https://education.yandex.ru/knowledge/formula-doveriia-doveritelnie-intervali-dlia-ratio-i-uplift). |
| **Must · P1** | [Инкрементальность перформанса и как её качать](https://www.youtube.com/watch?v=J7nldlI_x0g) — Яндекс Маркет | Как отделять эффект рекламы от покупок, которые произошли бы и без рекламы. Описание — [Яндекс Образование](https://education.yandex.ru/knowledge/inkrementalnost-performansa-i-kak-yee-kachat). |
| **High · P1** | [Эксперименты в Директе, или A/B-тестирование нового поколения](https://www.youtube.com/watch?v=ZAUUfkON81o) | Платформа экспериментов, несколько сегментов, неравные группы, неоднозначные результаты и принятие решения. |
| **High · P1** | [Tinkoff Product Analytics Meetup: A/B-тесты](https://www.youtube.com/watch?v=97uIWYft0xU) — Код Жёлтый, 2024 | Сравнивает последовательный анализ, автоматизацию решений и большое число метрик; хорошая связка с реальными ограничениями крупных компаний. |
| **High · P2** | [Online Controlled Experiments](https://www.youtube.com/watch?v=ZfhQ-fIg4EU) — Ron Kohavi / ACM | Каноническая база trustworthy experimentation. |
| **High · P2** | [A/B Testing Pitfalls: getting numbers you can trust is hard](https://www.youtube.com/watch?v=HEGI5QN3fXE) — Ron Kohavi | Ошибки, подглядывание, false positives, практическая культура экспериментов. |
| **High · P2** | [Causal inference for ecosystem effects](https://www.youtube.com/watch?v=wTgod2--WrQ) | Когда пользовательский A/B-тест не отвечает на вопрос об эффекте экосистемы. |
| **Must · P1** | [Propensity score matching: аналоги A/B-тестов](https://www.youtube.com/watch?v=eCTqp227xvo) | Matching, observational data и границы причинных выводов, когда рандомизация невозможна. |
| **High · P2** | [Uplift modelling for more efficient marketing](https://www.youtube.com/watch?v=2J9j7peWQgI) | Почему модель отклика не равна модели инкрементального эффекта и кому действительно стоит показывать воздействие. |
| **High · P2** | [Uplift Modelling — Ivan Klimuk](https://www.youtube.com/watch?v=A6a1MbH4fFk) | Второй прикладной пример uplift в банковском/маркетинговом контексте. |

Связанные карточки: [[A-B тест от гипотезы до решения — единица рандомизации, метрики и вердикт]], [[Ловушки экспериментов — SRM, peeking, multiple testing, novelty и interference]], [[Корреляция и причинность — confounders, selection bias и парадокс Симпсона]], [[Наблюдательные исследования — matching, difference-in-differences и regression discontinuity]], [[Статистический вывод — доверительный интервал, p-value, мощность и MDE]].

### Что законспектировать

Для каждого эксперимента выпиши:

- единицу рандомизации и почему она выбрана;
- primary metric, guardrails и delayed metrics;
- источник возможного interference;
- что будет означать «статистически значимо, но бизнес-неважно»;
- какое решение примут при конфликте краткосрочного и долгосрочного эффекта.

---

## 3. Поиск, ранжирование и рекомендации

| Приоритет | Источник | Что забрать для интервью |
|---|---|---|
| **Must · P1** | [Рекомендации и машинное обучение](https://www.youtube.com/watch?v=-JepzjMJ_cU) — Yandex for Developers, 2:57:58 | Полная картина рекомендательной системы и связка алгоритма с продуктом. |
| **Must · P1** | [Рекомендательные системы — К. В. Воронцов](https://www.youtube.com/watch?v=J-QueLndVI8), 1:21:24 | База: постановка задачи, сигналы, коллаборативные/контентные методы и оценка. |
| **Must · P1** | [Обучение ранжированию — К. В. Воронцов](https://www.youtube.com/watch?v=JTW9STt4_TE) | Learning-to-rank, pairwise/listwise постановки и связь с NDCG/MRR. |
| **Must · P1** | [Как Яндекс решает задачу ранжирования большими нейросетями](https://www.youtube.com/watch?v=M0-UGNFa0PA) | Как offline ranker становится частью search-продукта и какие ограничения появляются в production. |
| **Must · P1** | [Recommender Systems — Nikolay Savushkin and Dmitry Logvin](https://www.youtube.com/watch?v=_xdEEd6Zgwg) | Современный практический обзор рекомендательных систем Яндекса. |
| **High · P1** | [Generative recommendation technologies at Yandex](https://www.youtube.com/watch?v=0KPQG70Uhvs) | Новые генеративные подходы и вопросы, которые нужно задавать при оценке их реальной пользы. |
| **High · P1** | [Гибридная генеративно-ранжирующая модель в рекомендациях Яндекс Музыки](https://www.youtube.com/watch?v=WIRV4LFiVT8) | Архитектура production-рекомендаций и компромиссы между генерацией, ранжированием и latency. |
| **High · P1** | [Как улучшить рекомендации незнакомого в Яндекс Музыке](https://www.youtube.com/watch?v=_xfzdaEDnOE) | Cold start, exploration и измерение полезности незнакомого контента. |
| **High · P1** | [Ozon Tech: поиск, рекомендации и реклама](https://www.youtube.com/watch?v=yEfljf1rn1c) — ≈3 ч | Сильный e-commerce-кейс: рекламная платформа, search runtime, рекомендации и товарное ML. В описании ролика есть таймкоды 31:25, 1:12:03, 1:44:50 и 2:24:15 — можно смотреть только нужный доклад. |
| **High · P1** | [Как устроен поиск Яндекса](https://www.youtube.com/watch?v=vB27LVWK-KU) — 3:06:05 | Архитектурный контекст поиска: индексация, кандидаты, ranking, качество и ограничения. |
| **High · P2** | [Evaluation Measures for Search and Recommender Systems](https://www.youtube.com/watch?v=BD9TkvEsKwM) | Напоминание, что Precision/Recall/MRR/NDCG отвечают на разные вопросы. |
| **Must · P2** | [Offline metrics that predict online impact in RecSys](https://www.youtube.com/watch?v=rjGGSHhKDMM) | Один из самых важных материалов для собеседования: почему хороший offline score не гарантирует онлайн-эффект. |
| **High · P2** | [Position-biased click feedback and unbiased LTR](https://www.youtube.com/watch?v=dN_t0VnqtIs) | Position bias, click feedback и почему логи выдачи нельзя считать нейтральной разметкой. |
| **High · P2** | [BIAS: counterfactual learning to rank](https://www.youtube.com/watch?v=69E4YwX3M-8) | Более продвинутая версия темы position bias и counterfactual evaluation. |
| **Optional · P1** | [Как устроена система рекомендаций Ozon — внутри Intro Meetup](https://www.youtube.com/watch?v=yEfljf1rn1c) | Повторное указание конкретного фрагмента большого Ozon-ролика: смотреть 1:44:50–2:55:55. |

Связанные карточки: [[Метрики поиска и рекомендаций — CTR, Success Rate, Precision@K, Recall@K, MRR и NDCG]], [[Смещения в поиске и рекомендациях — position bias, feedback loops и offline-online gap]], [[ML-метрики в продукте — от offline score к бизнес-решению]].

### Мини-задание после просмотра

Возьми гипотезу «Recall@10 вырос, но покупки снизились» и ответь:

1. Какие причины проверишь в retrieval, reranking, контексте и трекинге?
2. Какие offline- и online-метрики поставишь рядом?
3. Какой guardrail остановит rollout?
4. Как отделишь позиционный bias от настоящего падения релевантности?

---

## 4. ML-аналитика, качество моделей и LLM-продукты

Этот блок особенно важен для вакансии аналитика-разработчика: здесь обсуждается не «насколько модель хороша сама по себе», а **как доказать её применимость, качество и эффект для продукта**.

| Приоритет | Источник | Что забрать для интервью |
|---|---|---|
| **Must · P1** | [Что такое ML-аналитика, или как измерить качество LLM-продуктов](https://www.youtube.com/watch?v=l7rNXP1WJdQ) — Таймураз Тибилов, 27:04 | Конфигурация оценки сложного LLM-пайплайна, разрыв между компонентными и end-to-end метриками, human/LLM evaluation и проблемы production-оценки. Официальное описание — [Яндекс Образование](https://education.yandex.ru/knowledge/chto-takoe-ml-аналитика-ili-kak-izmerit-kachestvo-llm-produktov-taimuraz-tibilov). |
| **Must · P1** | [Человек и LLM: как оценивать качество моделей и строить метрики](https://www.youtube.com/watch?v=peBvWuCpH_Y) — Ирина Барская, Яндекс Поиск | Как проектировать метрики для генеративного продукта и не сводить качество к одному автоматическому score. |
| **Must · P1** | [Продуктовый ML: ожидание и реальность — «Идеи» в Яндекс Картах](https://www.youtube.com/watch?v=NTf9-Gpeg0I) | Как проверить, что ML-функция решает пользовательскую задачу, а не только показывает красивую offline-метрику. Описание — [Яндекс Образование](https://education.yandex.ru/knowledge/produktovii-ml-ozhidanie-i-realnost-kak-rabotaiut-idei-v-yandeks-kartakh). |
| **Must · P1** | [RAG-системы сегодня: архитектуры, качество и наши кейсы](https://www.youtube.com/watch?v=-efHhXHhNys) | Retrieval quality, answer quality, latency, cost and failure analysis for a RAG product. |
| **High · P1** | [Опыт применения YandexGPT в разметке](https://www.youtube.com/watch?v=I98Vj_3BPcw) | Как оценивать LLM, когда она используется как инструмент разметки: agreement, sampling, human audit и стоимость ошибки. |
| **High · P2** | [ML System Design — оценка качества модели](https://www.youtube.com/watch?v=u4FtjGOJ9M0) — ODS AI Ru, 28:47 | Интервью-ориентированная рамка: цель модели, offline metric, threshold, calibration, business metric and monitoring. |
| **High · P2** | [Оценка работы ML-моделей от обучения до продакшена](https://www.youtube.com/watch?v=oXSXHlHacAo) | Lifecycle, acceptance criteria, validation and release quality. |
| **Must · P2** | [Data drift & Concept drift: как мониторить ML-модели в production](https://www.youtube.com/watch?v=LAQ2dj1ZEO4) — ODS AI Ru, 26:12 | Что мониторить при отсутствии быстрых labels: data quality, drift, prediction distribution, delayed business labels and slices. |
| **High · P2** | [Production ML Monitoring: outliers, drift, explainers & statistical performance](https://www.youtube.com/watch?v=n0bR0IArJDo) | Дополняет русский материал production-мониторингом и статистическими проверками. |
| **Must · P2** | [How to evaluate a RAG system](https://www.youtube.com/watch?v=qI2qQfOG0Js) | Удобная практическая раскладка retrieval metrics, faithfulness, answer relevance and test-set design. |
| **Must · P2** | [RAG and Agents Evaluation](https://www.youtube.com/watch?v=WUGtDveIe7A) — Alexey Grigorev | Как оценивать агентный пайплайн, где один итоговый ответ зависит от нескольких действий и вызовов инструментов. |
| **High · P2** | [How to systematically set up LLM evals](https://www.youtube.com/watch?v=a3SMraZWNNs) | Dataset, unit tests, LLM-as-a-judge, rubric, regression tests and production feedback loop. |
| **Optional · P2** | [AI Tutor and methods for its evaluation](https://www.youtube.com/watch?v=obeNjEOnG84) | Пример оценки сложного AI-продукта с несколькими типами качества. |

Связанные карточки: [[ML-метрики в продукте — от offline score к бизнес-решению]], [[Мониторинг продукта и ML-системы — алерты, срезы, drift и delayed labels]], [[Аналитический каркас ML System Design — цель, данные, эксперимент и решение]], [[Качество данных — дубликаты, пропуски, late events, часовые пояса и join explosion]], [[Метрики поиска и рекомендаций — CTR, Success Rate, Precision@K, Recall@K, MRR и NDCG]].

### Формула конспекта для ML-аналитики

По каждому продукту заполни пять строк:

1. **Что полезного должен получить пользователь?**
2. **Какая модельная метрика лишь приближает эту пользу?**
3. **Какие ошибки особенно дороги?**
4. **Как проверим online/incremental effect?**
5. **Что делаем при конфликте качества, latency, стоимости и fairness?**

---

## 5. ML System Design и репетиция кейсов

| Приоритет | Источник | Что тренировать |
|---|---|---|
| **Must · P3** | [ETA system — full ML System Design mock interview](https://www.youtube.com/watch?v=l-2m6YXlEjM) | Сначала уточнить objective и SLA, затем данные/labels, baseline, offline/online metrics, rollout и мониторинг. |
| **Must · P3** | [Design a ranking model — full mock interview](https://www.youtube.com/watch?v=7_E4wnZGJKo) | Ранжирование: candidates, features, labels, leakage, NDCG/CTR, exploration, latency и A/B-тест. |
| **High · P3** | [Design an ML Recommendation Engine](https://www.youtube.com/watch?v=FoSCaue3lcg) | Retrieval → ranking → re-ranking, cold start, diversity, feedback loops and business constraints. |
| **High · P1** | [ML-модели в продакшне Яндекс Такси](https://www.youtube.com/watch?v=IyDdfghMcbg) | Production lifecycle в marketplace-сервисе: data, serving, monitoring and business effect. |
| **High · P1** | [Всё, что вы делали не так в проектах с ML](https://www.youtube.com/watch?v=ST2uIcBmebg) | Типовые ошибки: неправильная постановка, leakage, не тот baseline, отсутствие мониторинга и разрыв с бизнесом. |
| **High · P1** | [Aouj: три истории про ML в offline retail](https://www.youtube.com/watch?v=Aouj_OIDZPY) | Кейсы, где важны не только модели, но и внедрение, операционные ограничения и измерение эффекта. |
| **High · P2** | [Bot Detection — ML System Design problem breakdown](https://www.youtube.com/watch?v=wI4L11EoW3I) | Хорошая тренировка для fraud/abuse-кейсов: imbalance, threshold, cost of errors and delayed labels. |

Связанные карточки: [[Аналитический каркас ML System Design — цель, данные, эксперимент и решение]], [[Банк аналитических и ML System Design кейсов]], [[Как решать аналитический кейс на интервью]], [[Разобранный кейс — в поиске вырос CTR, но снизились покупки]].

### Как смотреть mock interview

Поставь видео на паузу после постановки задачи и сначала проговори свой ответ. Структура ответа:

1. уточняю пользователя, действие и бизнес-решение;
2. называю ограничения и цену ошибки;
3. предлагаю простой baseline;
4. описываю данные и разбиение без leakage;
5. выбираю offline metrics;
6. связываю их с online-метриками и экспериментом;
7. добавляю rollout, мониторинг и план отката.

---

## 6. Кейсы компаний и масштабирование аналитики

Это блок для понимания того, как похожие задачи решаются в зрелых продуктах. Его не нужно смотреть полностью перед основным ядром.

| Приоритет | Источник | Что особенно полезно |
|---|---|---|
| **High · P1** | [Netflix: a confluence of metrics, algorithms & experimentation](https://www.youtube.com/watch?v=K4FRu6iIgOA) | Как связываются алгоритм, пользовательские метрики и эксперимент в контентном продукте. |
| **High · P1** | [Scaling experimentation at Airbnb](https://www.youtube.com/watch?v=8F3k9nNVf5Q) | Платформа, процессы, культура и качество экспериментов при масштабировании. |
| **High · P1** | [Online Experiments at Booking.com](https://www.youtube.com/watch?v=4e_rBd96iGA) | Демократизация экспериментов и предотвращение ошибочных решений. |
| **Must · P1** | [Uber Marketplace Experimentation](https://www.youtube.com/watch?v=IR000RqN7pw) | Marketplace interference и дизайн эксперимента на взаимосвязанном рынке. |
| **High · P1** | [Uber Sequential Testing](https://www.youtube.com/watch?v=4rWOx5fOJbg) | Sequential testing для быстрых продуктовых циклов. |
| **High · P1** | [Ozon Tech Intro Meetup: поиск, рекомендации и реклама](https://www.youtube.com/watch?v=yEfljf1rn1c) | Близкий к e-commerce набор систем; особенно полезен для контекста вакансии. |
| **High · P1** | [Tinkoff Product Analytics Meetup: A/B-тесты](https://www.youtube.com/watch?v=97uIWYft0xU) | Много метрик, sequential analysis и автоматизированная интерпретация тестов. |
| **High · P2** | [Avito: как работает фреймворк матчинга](https://rutube.ru/video/b0228f2bfb9c416978058665da588366/) | Реальный кейс, где A/B-тест невозможен: matching, regression, propensity score, оценка bias и ложных прокрасов. Это Rutube, но содержание настолько релевантно, что его стоит оставить в маршруте. |
| **High · P2** | [Как улучшить A/B-тесты в Avito: CUPED, bootstrap, стратификация](https://habr.com/ru/companies/avito/articles/571094/) | Письменное дополнение к кейсу Avito и хороший материал для вопросов «как повысить чувствительность теста?». |
| **High · P1** | [Ozon: высоконагруженная платформа A/B-тестов](https://habr.com/ru/companies/ozontech/articles/689052/) | Техническая сторона сплита трафика и experiment platform. |

---

## 7. Что смотреть для конкретного пробела

| Если не уверен в… | Начни с… | Потом переходи к… |
|---|---|---|
| North Star, guardrails, proxy | Яндекс Практикум `6vX7divNBRA` | Интегральные метрики Карт `ugyk4MxlCWU` |
| A/B-тестах и p-value | Яндекс Практикум `2gvf__7_X2c` | Ron Kohavi `ZfhQ-fIg4EU`, `HEGI5QN3fXE` |
| Switchback/interference | Яндекс Такси `0c8hxTMOZYg` | Uber Marketplace `IR000RqN7pw` |
| Ratio/uplift/инкрементальности | `0yhbw9QV9Z4`, `J7nldlI_x0g` | `eCTqp227xvo`, `2J9j7peWQgI` |
| Search/RecSys metrics | Воронцов `JTW9STt4_TE`, `J-QueLndVI8` | `rjGGSHhKDMM`, `dN_t0VnqtIs` |
| ML-аналитике | `l7rNXP1WJdQ` | `peBvWuCpH_Y`, `NTf9-Gpeg0I` |
| LLM/RAG evaluation | `-efHhXHhNys` | `qI2qQfOG0Js`, `WUGtDveIe7A` |
| Drift/monitoring | `LAQ2dj1ZEO4` | `n0bR0IArJDo`, `u4FtjGOJ9M0` |
| ML System Design | `l-2m6YXlEjM` | `7_E4wnZGJKo`, `FoSCaue3lcg` |

## 8. Готовый промт для прогона через ИИ

```text
Ты — строгий интервьюер на позицию аналитика-разработчика с фокусом на ML и продуктовую аналитику.

Проведи со мной реалистичный 35–45-минутный разбор кейса. Выбери одну тему из списка: продуктовые метрики, A/B-тест, switchback/marketplace, поиск, рекомендации, ML-метрики, LLM/RAG evaluation, drift/monitoring или ML System Design.

Правила:
1. Не подсказывай структуру ответа заранее и не поддакивай.
2. Давай вводные порциями. После каждого моего ответа задавай один уточняющий вопрос.
3. Если я называю метрику, спроси: что именно она измеряет, на каком уровне считается, какие у неё смещения и какое бизнес-решение изменится по её результату.
4. Если я предлагаю A/B-тест, проверь единицу рандомизации, interference, SRM, MDE, длительность, primary/guardrail/delayed metrics и multiple testing.
5. Если это ML-кейс, проверь baseline, labels, leakage, offline–online gap, latency, стоимость ошибки, rollout, мониторинг и план отката.
6. В кейсе поиска/рекомендаций отдельно спроси про candidate generation, ranking, position bias, exploration, cold start, diversity и feedback loops.
7. В кейсе LLM/RAG отдельно спроси про retrieval quality, groundedness/faithfulness, answer relevance, human evaluation, LLM-as-a-judge, latency, cost и regression set.
8. Не исправляй меня сразу. Сначала попроси закончить мысль. Затем укажи 2–3 конкретные ошибки или пробела и попроси улучшить ответ.
9. В конце оцени по шкале 0–3: постановка задачи, метрики, причинность/эксперимент, статистическая корректность, ML-понимание, связь с бизнесом, production/monitoring и ясность речи.
10. Заверши эталонным ответом уровня strong middle/senior и дай три упражнения на следующий прогон.

Начни с короткого кейса, который реалистичен для Яндекса или похожего marketplace/search-продукта, но не сообщай заранее, какие именно концепции хочешь проверить.
```

## 9. Новые карточки из папки «Транскрипции»

Ниже — локальные конспекты, подготовленные по новым файлам из `C:\Транскрипции`. В карточках источник отделён от учебных пояснений, а новые термины раскрыты простыми словами.

| Контекст | Карточки |
|---|---|
| Продуктовая аналитика | [[Лекции и видео/Разборы лекций/Продуктовая аналитика/01 — Метрики продуктовой аналитики и A-B-тесты — по транскрипции Яндекс Практикума.md|Метрики и A/B-тесты]] · [[Лекции и видео/Разборы лекций/Продуктовая аналитика/04 — Основные продуктовые метрики — конверсия, retention, ROI, ARPU и LTV.md|Базовые метрики]] · [[Лекции и видео/Разборы лекций/Продуктовая аналитика/05 — Как растить retention в Алисе и умных устройствах — по лекции Андрея Законов.md|Retention в Алисе]] · [[Лекции и видео/Разборы лекций/Продуктовая аналитика/06 — Продуктовая аналитика в Яндекс.Метрике и Cloud — ClickHouse, воронки и когорты.md|Метрика, ClickHouse и DataLens]] |
| Карты и продуктовый ML | [[Лекции и видео/Разборы лекций/Яндекс Карты/02 — Интегральная метрика качества справочника Яндекс Карт — по лекции Тимофея Стрункова.md|Интегральное качество справочника]] · [[Лекции и видео/Разборы лекций/Яндекс Карты/09 — Продуктовый ML и режим «Идеи» в Яндекс Картах — по лекции Никиты Киселёва.md|Режим «Идеи»]] |
| RAG и генеративные модели | [[Лекции и видео/Разборы лекций/RAG и LLM/03 — RAG-системы — архитектура, качество и кейсы Алисы — по лекции Андрея Соколова.md|RAG и кейсы Алисы]] · [[Лекции и видео/Разборы лекций/RAG и LLM/14 — Как системно строить оценки LLM — unit-тесты, человек и LLM-as-a-judge.md|LLM-evals]] · [[Лекции и видео/Разборы лекций/RAG и LLM/15 — AI-тьютор — диалог, оценка и human-in-the-loop.md|AI-тьютор]] |
| Эксперименты и причинность | [[Лекции и видео/Разборы лекций/Эксперименты и причинность/11 — Switchback-эксперименты в Яндекс Такси — сетевые эффекты и рандомизация по времени.md|Switchback]] · [[Лекции и видео/Разборы лекций/Эксперименты и причинность/12 — Доверительные интервалы для ratio- и uplift-метрик — по лекции Дили Хакимовой.md|Ratio и uplift]] · [[Лекции и видео/Разборы лекций/Эксперименты и причинность/13 — Propensity Score Matching — аналог A-B-теста при невозможности эксперимента.md|PSM]] · [[Лекции и видео/Разборы лекций/Эксперименты и причинность/16 — Фреймворк матчинга в наблюдательных исследованиях — по лекции «Диванная аналитика».md|Matching в Авито]] |
| ML System Design | [[Лекции и видео/Разборы лекций/ML System Design/10 — Оценка качества ML-системы — интерфейс, данные, модель и мониторинг.md|Оценка качества]] · [[Лекции и видео/Разборы лекций/ML System Design/17 — Mock ML System Design в Яндексе — прогноз спроса и запасов валюты.md|Прогноз спроса]] · [[Лекции и видео/Разборы лекций/ML System Design/18 — ETA в Яндекс Картах — baseline, backtesting и расчёт маршрута.md|ETA]] · [[Лекции и видео/Разборы лекций/ML System Design/19 — Ranking-модель для ленты Instagram — кандидаты, признаки, negative sampling и A-B-тест.md|Ranking]] · [[Лекции и видео/Разборы лекций/ML System Design/22 — Как проходить ML-секцию в Яндексе — геоподсказки, метрики и эксперимент.md|ML-секция в Яндексе]] |
| Продакшен ML | [[Лекции и видео/Разборы лекций/Продакшен ML/20 — ML-модели в продакшне Яндекс Такси — полный конвейер.md|Модели в продакшне Такси]] · [[Лекции и видео/Разборы лекций/Продакшен ML/21 — Ошибки ML-проектов — постановка, метрика, эксперимент и бизнес-контекст.md|Ошибки ML-проектов]] |

## 10. Как понять, что материал действительно усвоен

Видео просмотрено не тогда, когда ты дошёл до конца, а когда можешь без конспекта:

- объяснить метрику простыми словами и привести пример, где она обманывает;
- предложить baseline и объяснить, почему он нужен;
- назвать единицу анализа и единицу рандомизации;
- связать offline score с online/business effect;
- разобрать конфликт primary metric и guardrail;
- назвать источник bias или leakage;
- предложить мониторинг, delayed labels и план отката;
- защитить решение перед продуктом, который не знает статистических терминов.

---

## Источники и ограничения поиска

Полный внутренний журнал источников, включая источники, которые не вошли в основной маршрут, находится в [`.research/analytics-video-research/report-source.md`](<../.research/analytics-video-research/report-source.md>). Метаданные собирались 26 августа 2026 года. YouTube иногда ограничивал прямой просмотр страниц, поэтому для части роликов дата, описание и таймкоды проверялись через поиск YouTube и официальные страницы Яндекс Образования. Ни один источник не является гарантией того, что именно его покажут на интервью: приоритет отражает соответствие темам подготовки.
