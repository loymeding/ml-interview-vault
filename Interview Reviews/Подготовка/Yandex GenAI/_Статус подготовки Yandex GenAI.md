---
tags: [interview-prep, yandex, generative-ai, evaluation, agents, vlm]
company: Yandex
role: Аналитик-разработчик / аналитик качества генеративных моделей
type: interview-prep-dashboard
status: active
created: 2026-08-15
---

# Подготовка к вакансиям Yandex GenAI

> [!abstract] Фокус подготовки
> Пять вакансий объединяет роль аналитика-разработчика качества foundation models: спроектировать измерение качества, найти failure modes, собрать надёжный human/LLM/tool-based сигнал, превратить его в данные и проверить прирост на независимом eval.

## Целевые вакансии

| Вакансия | Основной фокус | Ключевые артефакты подготовки |
|---|---|---|
| YandexART | Text-to-image, крауд, human preference, пользовательский сигнал | T2I benchmark, prompt taxonomy, pairwise human eval |
| Alice AI LLM | Срезы ошибок, fine-tune data, асессоры | rubric, annotation audit, data curation pipeline |
| Агентские сценарии | Product benchmarks, API-среды, golden trajectories, RL | mock environment, verifier, trajectory dataset |
| Alice AI VLM | Multi-image, reasoning, tool use, judge | multimodal benchmark, error taxonomy, judge audit |
| R&D | Широкий контур моделей, новые datasets/benchmarks, эксперименты | research memos, reproducible eval, ablation reports |

## Статус по блокам

| Блок | Приоритет | Статус |
|---|---:|---|
| Benchmark design и error taxonomy | 🔥 | 🟢 Готово |
| Human evaluation и крауд-разметка | 🔥 | 🟢 Готово |
| LLM-as-a-Judge и quality audit | 🔥 | 🟢 Есть базовая карточка |
| Data curation и synthetic pipelines | 🔥 | ⚪ Запланировано |
| Агентные среды, verifier и trajectories | 🔥 | 🟢 Готово |
| VLM evaluation: multi-image/reasoning/tool use | 🔥 | 🟢 Готово |
| Text-to-image evaluation | 🔥 | 🟢 Готово |
| Product signal и offline-to-online | 🟡 | 🟢 Готово |
| YQL/Toloka/reproducibility | 🟡 | 🟢 Готово |

## Как пользоваться разделом

1. Читать [[Роадмап подготовки Yandex GenAI]] по порядку, а не отдельные карточки случайно.
2. Для каждой темы сначала изучать [[Вакансии и матрица требований Yandex GenAI]], затем карточку, затем выполнять практику.
3. После изучения отвечать на вопросы из [[Разборы кейсов/Банк и тренировки/Вопросы и практические кейсы Yandex GenAI.md|Вопросы и практические кейсы Yandex GenAI]] вслух и письменно.
4. Все проекты держать на русском языке и проверять на независимом held-out наборе: это ближе к продуктовым сценариям Алисы, чем только англоязычные публичные benchmark'и.

## Основной результат подготовки

К концу трека должны быть готовы:

- один воспроизводимый benchmark;
- одна инструкция для human annotation и её audit;
- один LLM-judge, откалиброванный на людях;
- одна mock-agent environment с детерминированным verifier;
- один multimodal или text-to-image mini-benchmark;
- 5–8 коротких research/product memos с ограничениями методов.

## Навигация

- [[Вакансии и матрица требований Yandex GenAI]]
- [[Роадмап подготовки Yandex GenAI]]
- [[Разборы кейсов/Банк и тренировки/Вопросы и практические кейсы Yandex GenAI.md|Вопросы и практические кейсы Yandex GenAI]]
- [[Источники Yandex GenAI]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка foundation models — benchmark design и error taxonomy]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Human evaluation и крауд-разметка для LLM и VLM]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Агентные среды — API, state, verifier и golden trajectories]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/RL-данные для агентов — rollout, reward и trajectory curation]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка VLM — multi-image, reasoning и tool use]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Оценка text-to-image — human preference и failure modes]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Data curation для fine-tuning и synthetic pipelines]]
- [[Interview Reviews/Подготовка/Yandex GenAI/Карточки/Product signal — offline eval, логи и пользовательская ценность]]
