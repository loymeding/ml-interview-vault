---
tags: [interview-prep, yandex, sources, literature, evaluation]
company: Yandex
type: source-index
status: active
created: 2026-08-15
---

# Источники Yandex GenAI

## Читать в первую очередь

| Направление | Источник | Зачем |
|---|---|---|
| Foundation model evaluation | [HELM](https://crfm.stanford.edu/helm/) | Покрытие сценариев, многомерные метрики, воспроизводимость |
| VLM evaluation | [VHELM](https://crfm.stanford.edu/helm/vhelm/v2.0.0/) | Системная оценка visual perception, reasoning, safety и robustness |
| Text-to-image | [HEIM](https://arxiv.org/abs/2311.04287) | Автоматическая и human оценка визуальной генерации |
| LLM judge | [Judging LLM-as-a-Judge](https://arxiv.org/abs/2306.05685) | Agreement с людьми и основные bias |
| Reward models | [RewardBench](https://github.com/allenai/reward-bench) | Оценка reward-моделей и preference signal |
| Agent environments | [WebArena](https://arxiv.org/abs/2307.13854) | Реалистичная воспроизводимая среда для long-horizon задач |
| Tool-agent-user | [τ-bench](https://arxiv.org/abs/2406.12045) | API, policy-following, конечное состояние и pass^k |
| Evaluation framework | [Inspect](https://inspect.aisi.org.uk/) | Практическая реализация eval для agents и VLM |
| Static eval tasks | [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) | Как оформлять и запускать воспроизводимые benchmark tasks |
| Crowdsourcing | [Toloka: quality evaluation](https://toloka.ai/blog/quality-evaluation/) | Разметчики, consensus, quality control |
| Crowdsourcing algorithms | [Crowd-Kit](https://arxiv.org/abs/2109.08584) | Методы агрегации и оценки качества исполнителей |
| Text-to-image diagnosis | [GenEval](https://github.com/djghosh13/geneval) | Object, count, attributes и spatial relations |
| Tool use in VLM | [ToolVQA](https://arxiv.org/abs/2508.03284) | Multi-step visual reasoning с внешними инструментами |
| Synthetic alignment data | [Self-Instruct](https://arxiv.org/abs/2212.10560) | Генерация и фильтрация инструкций |
| Data curation | [DataComp-LM](https://proceedings.neurips.cc/paper_files/paper/2024/file/19e4ea30dded58259665db375885e412-Paper-Datasets_and_Benchmarks_Track.pdf) | Данные как объект оптимизации |
| YQL | [YTsaurus YQL docs](https://ytsaurus.tech/docs/en/yql/) | Понять распределённый SQL-подобный контур |

## Как читать paper

Для каждого источника создать короткий memo:

1. Какое поведение измеряется?
2. Какова единица оценки: ответ, диалог, trajectory, final state или image?
3. Откуда берётся ground truth?
4. Какая метрика используется и чего она не видит?
5. Какие leakage, bias и confounders возможны?
6. Что можно перенести в продукт Алисы или YandexART?
7. Какой небольшой эксперимент воспроизвести?

## Русскоязычные и продуктовые источники

Публичные benchmarks часто англоязычные и не покрывают русскую разговорную речь, культурные реалии и продуктовые сценарии Яндекса. Поэтому каждый capstone должен иметь собственный русский held-out слой: ambiguity, colloquial language, code-switching, неполные требования и длинные пользовательские формулировки.

