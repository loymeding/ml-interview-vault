---
tags: [interview-prep, yandex, roadmap, evaluation, agents, vlm]
company: Yandex
type: study-roadmap
status: active
created: 2026-08-15
---

# Роадмап подготовки Yandex GenAI

Ориентир: 10–12 недель по 8–12 часов. Если интервью близко, сначала пройти недели 1–4 и 6–7.

## Фаза 1. Общий фундамент оценки — недели 1–2

Карточка: [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка foundation models — benchmark design и error taxonomy]].

Изучить:

- capability/product/safety evaluation;
- benchmark coverage, leakage, contamination и held-out split;
- task success, pairwise preference, rubric score, constraint satisfaction;
- bootstrap CI, slice analysis и uncertainty;
- почему eval должен быть versioned и воспроизводимым.

Практика: написать eval-spec для сценария выбора товара, сделать 80–120 русскоязычных заданий, сравнить две модельные конфигурации и оформить отчёт с CI и разрезами.

## Фаза 2. Human data — неделя 3

Карточка: [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Human evaluation и крауд-разметка для LLM и VLM]].

Изучить rubric design, qualification, gold tasks, overlap, agreement, adjudication и аудит ошибок разметчиков.

Практика: разметить 100–150 пар ответов минимум двумя независимыми людьми; исправить rubric после пилота; сохранить disagreement examples.

## Фаза 3. Judge и данные — недели 4–5

Карточки: [[NLP/LLM и Промпт-инжиниринг/LLM-as-a-Judge — rubric, bias и дообучение]], [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Data curation для fine-tuning и synthetic pipelines]].

Изучить position/verbosity/self-preference bias, human calibration, judge cascade, deduplication, diversity, difficulty и failure-driven generation.

Практика: сравнить judge с human-held-out набором, сделать swap test, проверить judge на adversarial examples и отобрать небольшой датасет по taxonomy ошибок.

## Фаза 4. Агентский контур — недели 6–7

Карточки: [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Агентные среды — API, state, verifier и golden trajectories]], [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/RL-данные для агентов — rollout, reward и trajectory curation]].

Изучить stateful environments, API contract, deterministic verifier, допустимые траектории, rollout reliability, outcome/process reward и reward hacking.

Практика: сделать mock retail environment на Python/SQLite с 50–100 заданиями, 4–6 tool calls, final-state verifier и пятью повторными запусками каждой задачи.

## Фаза 5. Мультимодальность — неделя 8

Карточка: [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка VLM — multi-image, reasoning и tool use]].

Изучить visual perception, OCR, grounding, multi-image comparison, visual reasoning, tool selection и tool-result integration.

Практика: benchmark из 60–80 задач с одним и несколькими изображениями; классифицировать ошибки по этапам pipeline.

## Фаза 6. Генерация изображений — неделя 9

Карточка: [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка text-to-image — human preference и failure modes]].

Изучить compositional alignment, object counting, attributes, spatial relations, aesthetic quality, human preference и расхождение автоматических метрик с людьми.

Практика: 100–150 русских промптов, две-три модели, автоматическая и pairwise human оценка, отчёт по disagreement slices.

## Фаза 7. Product signal — неделя 10

Карточка: [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Product signal — offline eval, логи и пользовательская ценность]].

Изучить weak labels, selection bias, position/latency confounders, offline-to-online gap и осторожную интерпретацию regenerate/save/share.

Практика: синтетический лог событий и записка о том, какие события пригодны для обучения, а какие — только для error mining.

## Фаза 8. Capstone и интервью — недели 11–12

Выбрать либо agent benchmark, либо T2I/VLM benchmark. Подготовить:

- README и архитектуру;
- данные и версии split;
- evaluator/verifier;
- error taxonomy;
- отчёт с CI и ограничениями;
- 10-минутный рассказ «проблема → данные → eval → failure modes → изменение → результат».

## Еженедельный ритм

- 2 часа: primary paper или документация;
- 5–7 часов: код и данные;
- 1–2 часа: аудит ошибок и альтернативных объяснений;
- 1 час: короткий memo в базу знаний.
