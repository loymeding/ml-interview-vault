---
tags: [yandex, interview-prep, data-curation, synthetic-data, fine-tuning, datasets]
тип: теория и практика
уровень: senior
сложность: высокая
статус: готово
готовность: 100
создано: 2026-08-15
вакансии: [YandexART, Alice AI LLM, agents, Alice AI VLM, R&D]
источники: [Self-Instruct, DataComp-LM, CLEAR]
предпосылки:
  - "NLP/LLM и Промпт-инжиниринг/Alignment и обучение LLM — Pretraining, SFT, RLHF, DPO, SimPO"
  - "NLP/LLM и Промпт-инжиниринг/Fine-tuning и PEFT в LLM — BERT, LoRA, QLoRA"
---

# Data curation для fine-tuning и synthetic pipelines

> [!abstract] Суть
> Полезность data определяется не размером, а тем, закрывает ли она реальные failure modes, разнообразна ли, проверяема ли и улучшает ли независимый eval.

## 0. Сначала простая картина

Fine-tuning меняет поведение модели через примеры, которые мы ей показываем. Значит, датасет — не склад текстов, а набор сигналов: он подсказывает, какие ответы считать нормой, какие действия выбирать и какие ошибки не допускать. Data curation — работа по отбору, очистке и описанию этих примеров.

Quality отвечает на вопрос «пример сам по себе хороший?»: он корректен, понятен, безопасен и действительно относится к задаче. Utility отвечает на другой вопрос: «поможет ли этот пример именно нашей модели на нужном навыке?». Большой набор может быть качественным, но бесполезным, если в нём нет редких случаев, на которых модель ошибается.

Synthetic data — примеры, созданные другой моделью или программой. Они помогают быстро закрывать пробелы, но могут повторять одну манеру письма, переносить ошибки учителя или незаметно дублировать тест. Provenance — происхождение примера: откуда он взялся, какой моделью создан, как проверен и в какую версию набора попал.

Поэтому разумный pipeline начинается с реальных failure modes, затем добавляет целевые примеры, проверяет их качество и измеряет прирост на отложенном наборе. Ниже мы идём от источников данных к схеме provenance, очистке, генерации и проверке полезности — так, как это обсуждается на R&D-собеседовании.

### Словарь перед стартом

- **Fine-tuning** — дополнительное обучение уже готовой модели на целевых примерах.
- **Synthetic** — созданный автоматически, а не собранный напрямую из пользовательского или экспертного источника.
- **Deduplication** — удаление повторов и почти повторов.
- **Data contamination** — попадание тестовых примеров или их близких копий в обучение.

## 1. Quality и utility

Data quality — корректность, полнота и соответствие инструкции. Data utility — то, насколько пример помогает улучшить нужный навык конкретной модели.

Высококачественный, но слишком лёгкий датасет может не дать прироста. Большой synthetic dataset может ухудшить модель, если он однообразен, содержит ошибки или дублирует evaluator.

> [!tip] Переход от критерия пользы к поиску примеров
> Если мы знаем, какой навык хотим улучшить, следующий вопрос — где взять соответствующие примеры и как не потерять информацию об их происхождении.

## 2. Источники data

### Production logs

Плюсы: реальные формулировки и потребности. Минусы: privacy, selection bias, неизвестный ground truth и зависимость от старой модели.

### Expert demonstrations

Плюсы: надёжное поведение и сложные cases. Минусы: дорого и ограниченный объём.

### Crowd labels

Плюсы: масштаб и разнообразие. Минусы: сложные rubrics, disagreement и контроль качества.

### Synthetic generation

Плюсы: управление coverage, сложностью и hard negatives. Минусы: model bias, шаблонность и self-reinforcing errors.

### Rollouts

Плюсы: реальные agent traces. Минусы: ошибки среды, reward hacking и дорогой inference.

## 3. Schema и provenance

Каждый пример должен содержать:

~~~yaml
sample_id: agent_0042
source: human_demo
skill: clarification
failure_type: missing_question
difficulty: long_horizon
input: "..."
target: "..."
metadata:
  generator_version: null
  judge_version: v3
  rubric_version: v2
split: train
~~~

Нужны отдельные versions для raw, filtered и final data. Нужно уметь ответить, откуда взялся каждый пример и почему он попал в train.

## 4. Pipeline очистки

~~~text
raw data
  -> schema validation
  -> language/PII/safety filters
  -> normalization
  -> exact and semantic deduplication
  -> quality judge
  -> human audit
  -> diversity/difficulty balancing
  -> train/dev/test split
~~~

Фильтры не должны использовать held-out test как источник примеров или порогов. Иначе test постепенно превращается в train.

## 5. Failure-driven generation

Генерировать случайные инструкции слабее, чем закрывать конкретную taxonomy ошибку.

Наблюдение:

~~~text
31% failures: агент не уточняет delivery date
~~~

Дальше создаём parameterized generator:

- разные формулировки даты;
- отсутствие даты;
- противоречивые даты;
- локальные и относительные даты;
- дополнительные несущественные детали;
- неоднозначное «как можно скорее».

После генерации добавляем hard negatives и проверяем задачи human/verifier, а не только генератором.

> [!tip] Переход от чистоты к покрытию
> Чистый набор может быть однообразным. После удаления мусора нужно проверить, какие варианты задач он действительно покрывает и какие важные случаи всё ещё отсутствуют.

## 6. Diversity

Разнообразие нужно контролировать по нескольким осям:

- лексика и формулировки;
- длина и структура диалога;
- сложность;
- тип пользовательской цели;
- язык и code-switching;
- состояние среды;
- путь решения;
- частота и редкость объекта.

Embedding diversity сама по себе не гарантирует функционального покрытия: два семантически далёких запроса могут проверять один и тот же навык.

## 7. Synthetic data risks

- генератор повторяет собственные шаблоны;
- judge и generator имеют общие систематические ошибки;
- rationale звучит убедительно, но фактически неверен;
- test leakage появляется через prompt seeds;
- synthetic distribution не похожа на пользовательскую;
- модель обучается оптимизировать evaluator.

Нужны независимые judge/human checks, held-out real-like data и explicit provenance.

## 8. Как проверить пользу данных

Сравниваем минимум три условия:

1. baseline model;
2. training на random subset;
3. training на failure-driven subset.

Смотрим:

- target slice;
- общий held-out score;
- guardrails;
- соседние навыки;
- human preference;
- stability при новых формулировках.

Если training loss уменьшается, но target slice не растёт, примеры не закрывают нужную capability или evaluator не отражает её.

## 9. Практический пример

Failure: VLM путает цену на двух изображениях.

Data iteration:

1. Создать multi-image examples с одинаковыми и похожими ценами.
2. Добавить разные layouts, crop и порядок изображений.
3. Разметить atomic facts с image id.
4. Отфильтровать дубликаты.
5. Заморозить held-out examples с новыми товарами.
6. Проверить improvement именно в grounding, а не только в финальном answer.

## Практическая задача

Собрать 300–500 примеров из 5–8 failure slices. Для каждого сохранить provenance, skill, complexity, filter decisions и human/judge labels. Сравнить random selection и taxonomy-driven selection на независимом eval.

## Типичные ошибки

- **Считать объём качеством:** больше примеров не гарантирует пользу.
- **Не хранить provenance:** нельзя провести аудит.
- **Удалять сложные примеры как шум:** они могут содержать основной обучающий сигнал.
- **Фильтровать только одним judge:** ошибка judge становится ошибкой data.
- **Смешивать data и eval:** результат перестаёт быть честным.
- **Не измерять diversity по навыкам:** embedding clustering не заменяет functional coverage.

## Вопросы для интервью

> [!question] Как выбрать данные для конкретной ошибки?
> Сначала описать failure taxonomy, затем создать контролируемые вариации ошибки, добавить hard negatives, проверить labels и оценить target slice на held-out set.

> [!question] Почему synthetic data может ухудшить модель?
> Из-за ошибок генератора, однообразия, leakage, неправильной сложности и усиления bias собственной модели.

> [!question] Как доказать полезность data pipeline?
> Провести controlled comparison с random baseline и проверить независимый held-out eval, human labels и guardrails.

## Проверка себя

- Спроектировать generator для одного failure slice.
- Описать provenance schema.
- Назвать пять фильтров до обучения.
- Объяснить разницу data quality и data utility.

## Связано

- [[NLP/LLM и Промпт-инжиниринг/Alignment и обучение LLM — Pretraining, SFT, RLHF, DPO, SimPO]]
- [[NLP/LLM и Промпт-инжиниринг/Fine-tuning и PEFT в LLM — BERT, LoRA, QLoRA]]
- [[NLP/LLM и Промпт-инжиниринг/LLM-as-a-Judge — rubric, bias и дообучение]]

## Источники

- [Self-Instruct](https://arxiv.org/abs/2212.10560)
- [DataComp-LM](https://proceedings.neurips.cc/paper_files/paper/2024/file/19e4ea30dded58259665db375885e412-Paper-Datasets_and_Benchmarks_Track.pdf)
- [Automated Data Curation for Robust Language Model Fine-Tuning](https://arxiv.org/abs/2403.12776)

---
[[Interview Reviews/Подготовка/Yandex GenAI/🗺️ Индекс|Назад к разделу]]
