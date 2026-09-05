---
tags: [interview-prep, yandex, vacancy-analysis, requirements]
company: Yandex
type: vacancy-matrix
status: active
created: 2026-08-15
---

# Вакансии и матрица требований Yandex GenAI

## Общий профиль

Все вакансии находятся на стыке аналитики, ML research и data engineering. Ожидается не только посчитать метрику, но и самостоятельно построить контур оценки, понять причины ошибки и предложить следующий эксперимент.

Общий цикл:

```text
логи/запросы -> срезы -> taxonomy ошибок -> benchmark/разметка
-> данные или reward -> новая модель -> независимая переоценка
```

## Матрица требований

| Компетенция | YandexART | Alice AI LLM | Агентские сценарии | Alice AI VLM | R&D |
|---|---:|---:|---:|---:|---:|
| Метрики и benchmark design | 🔥 | 🔥 | 🔥 | 🔥 | 🔥 |
| Error analysis и slice mining | 🔥 | 🔥 | 🔥 | 🔥 | 🔥 |
| Human evaluation/крауд | 🔥 | 🔥 | 🟡 | 🔥 | 🔥 |
| LLM-as-a-Judge | 🔥 | 🔥 | 🔥 | 🔥 | 🔥 |
| Synthetic data и filtering | 🟡 | 🔥 | 🔥 | 🔥 | 🔥 |
| Tool use и agent trajectories | — | 🟡 | 🔥 | 🔥 | 🔥 |
| RL-сигнал и reward design | 🔥 | 🟡 | 🔥 | 🟡 | 🔥 |
| Multi-image/VLM | — | — | 🟡 | 🔥 | 🟡 |
| Text-to-image/CV | 🔥 | — | — | 🟡 | 🟡 |
| Python для инфраструктуры | 🟡 | 🟡 | 🔥 | 🟡 | 🟡 |
| SQL/анализ больших логов | 🔥 | 🔥 | 🔥 | 🔥 | 🔥 |
| Статистика и uncertainty | 🔥 | 🔥 | 🔥 | 🟡 | 🔥 |
| Чтение research на английском | 🟡 | 🟡 | 🔥 | 🔥 | 🔥 |

## Что уже покрыто существующей базой

- База LLM и alignment: [[NLP/LLM и Промпт-инжиниринг/Alignment и обучение LLM — Pretraining, SFT, RLHF, DPO, SimPO]].
- Общий LLM judge: [[NLP/LLM и Промпт-инжиниринг/LLM-as-a-Judge — rubric, bias и дообучение]].
- Общие агенты: [[NLP/LLM и Промпт-инжиниринг/Агенты, function calling и structured output]].
- Архитектура VLM: [[NLP/Модели и архитектуры/Визуально-языковые модели (VLM) и Vision Encoder]].
- Метрики и статистика: [[А-Б тесты/Статистика/Непараметрика и Bootstrapping в А-Б-тестах]] и [[А-Б тесты/Методология/Иерархия метрик — Целевая, Guardrail и Proxy]].

Новые карточки этого раздела должны отвечать на вопрос «как применять это для качества foundation model в продукте», а не повторять архитектурную базу.

## Диагностика вакансий

### YandexART

Нужно уметь сравнивать версии датасетов, строить срезы промптов, разрабатывать human/automated image evaluation, запускать асессорскую разметку и осторожно интерпретировать save/share/regenerate как weak preference signal.

### Alice AI LLM

Нужно уметь находить проблемные навыки модели, собирать данные для fine-tuning, формировать инструкции асессорам, проверять качество разметки и оценивать эффект изменений.

### Агентские сценарии

Нужно уметь кодировать среды над API, задавать конечные состояния, строить verifier, собирать golden trajectories и оценивать надёжность многошагового tool use.

### Alice AI VLM

Нужно разделять visual perception, grounding, reasoning, выбор тула, аргументы вызова и использование результата; отдельно оценивать multi-image и production-like сценарии.

### R&D

Нужно быстро прочитать новый benchmark или dataset, проверить его применимость, найти ограничения и превратить идею в воспроизводимый эксперимент.

## Ссылки на исходные вакансии

- [YandexART](https://yandex.ru/jobs/vacancies/analitikrazrabotchik-modeli-dlya-generatsii-izobrazheniy-yandexart-47577)
- [Аналитик-разработчик Alice AI LLM](https://yandex.ru/jobs/vacancies/analitikrazrabotchik-analitiki-yandexgpt-17523)
- [Аналитик качества продуктовых агентских сценариев](https://yandex.ru/jobs/vacancies/analitik-kachestva-produktovih-agentskih-stsenariev-47049)
- [Аналитик-разработчик Alice AI VLM](https://yandex.ru/jobs/vacancies/analitikrazrabotchik-v-alice-ai-vlm-45545)
- [Аналитик-разработчик в R&D](https://yandex.ru/jobs/vacancies/analitikrazrabotchik-v-otdel-nlp-16400)

