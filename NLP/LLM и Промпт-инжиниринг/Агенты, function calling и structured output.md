---
tags: [llm, agents, rag, function-calling, structured-output, observability, langchain, langgraph]
тип: теория и практика
уровень: middle+
сложность: средняя
статус: готово
готовность: 90
создано: 2026-05-28
источники:
  - "Sber/Yandex interview notes"
предпосылки:
  - "RAG — retrieval, hybrid search, RRF и embeddings"
связано:
  - "RAG — генерация, оценка качества и production"
сравнить-с: []
---

# Агенты, function calling и structured output

> [!abstract] Суть
> LLM-агент — это не "модель стала умной сама по себе", а контролируемый цикл: модель выбирает действие, вызывает инструмент, читает результат и решает, что делать дальше. Для production важны structured output, трассировка, ограничения на инструменты и проверка качества.

## Ответ

### 1. Что такое агентский RAG

Обычный RAG:

```text
query -> retrieve -> generate
```

Агентский RAG:

```text
query
  -> decide what is missing
  -> choose retriever/tool
  -> maybe rewrite query
  -> retrieve
  -> inspect result
  -> maybe retrieve again / use SQL / ask clarifying question
  -> answer
```

Агент полезен, когда:
- вопрос многошаговый;
- нужно выбрать между несколькими источниками;
- нужно уточнять фильтры;
- нужен SQL/table tool;
- нужен web/API/tool call;
- один retrieval часто не дает ответа.

В tool для RAG можно заложить не только переписывание запроса, но и параметры:
- `query`;
- `filters`: продукт, дата, регион, тип документа;
- `top_k`;
- `retriever_type`: vector, bm25, hybrid, sql, graph;
- `search_mode`: exact, semantic, broad, strict;
- `metadata_boost`;
- `rerank`: true/false;
- `time_range`;
- `language`;
- `required_citations`;
- `parent_document_id`;
- `table_name` или `sheet_name`.

### 2. Архитектуры агентов

**ReAct**: модель чередует reasoning и actions: подумала, выбрала tool, получила observation, продолжила.

**Plan-and-execute**: сначала строится план, потом отдельный executor выполняет шаги.

**Router agent**: выбирает путь: RAG, SQL, calculator, code, web, human handoff.

**Reflection / self-critique**: модель проверяет свой ответ или план и исправляет ошибки.

**Multi-agent**: роли разделены: planner, retriever, coder, critic, summarizer.

**Workflow/state-machine agent**: жесткий граф состояний, где LLM принимает решения только в ограниченных местах. Для production часто надежнее свободного ReAct.

Для агентского RAG обычно удобен router + state machine:

```text
classify intent -> choose retrieval tools -> verify evidence -> answer with citations
```

### 3. Фреймворки для прототипирования

Часто используют:
- LangChain: много интеграций и быстрый старт;
- LangGraph: state-machine/graph agents, удобно контролировать циклы;
- LlamaIndex: сильный фокус на ingestion, retrieval и индексы;
- Haystack: production-oriented RAG pipelines;
- Semantic Kernel: интеграция с enterprise/.NET/планированием;
- DSPy: оптимизация prompts/pipelines через метрики;
- кастомный FastAPI + vLLM + vector DB, если нужен контроль и простота.

Для быстрого прототипа RAG-агента обычно берут LlamaIndex/LangChain. Для production, где важен контроль шагов, чаще переходят к LangGraph/state machine или собственному orchestration.

### 4. Structured output

Structured output — это режим, где модель должна вернуть не произвольный текст, а объект заданной структуры:

```json
{
  "answer": "Карту можно закрыть через...",
  "citations": [
    {"doc_id": "policy_12", "page": 5}
  ],
  "confidence": "medium"
}
```

Способы добиться структуры:
- prompt + examples;
- JSON schema / Pydantic schema;
- function/tool calling;
- constrained decoding по грамматике;
- post-validation + retry;
- repair parser.

Prompt сам по себе не дает 100% гарантии. Гарантии появляются, когда декодер ограничивает допустимые следующие токены так, чтобы невозможно было выйти за схему.

### 5. Можно ли добиться 100% валидности по regex

Если просто попросить модель "верни email", 100% гарантии нет.

Если используется constrained decoding по регулярному языку/грамматике, можно гарантировать синтаксическую валидность строки относительно заданного regex. Технически система строит автомат/grammar mask и на каждом шаге разрешает только токены, которые оставляют возможность завершить валидную строку.

Но есть нюансы:
- regex должен описывать именно синтаксис, а не смысл;
- токенизация может усложнять маскирование;
- валидный email по regex не означает, что домен существует;
- сложные regex могут быть дорогими или неоднозначными;
- бизнес-валидность все равно проверяется внешним валидатором.

Правильная схема:

```text
constrained generation -> parser validation -> business validation -> retry/fallback
```

### 6. Function calling на уровне HTTP

Идея: модель не сама вызывает API. Она возвращает структурированное намерение вызвать инструмент, а приложение выполняет вызов.

Шаг 1. Клиент отправляет в LLM:

```json
{
  "messages": [
    {"role": "user", "content": "Какая погода в Москве завтра?"}
  ],
  "tools": [
    {
      "name": "get_weather",
      "description": "Возвращает прогноз погоды",
      "parameters": {
        "type": "object",
        "properties": {
          "city": {"type": "string"},
          "date": {"type": "string"}
        },
        "required": ["city", "date"]
      }
    }
  ]
}
```

Шаг 2. Модель отвечает не финальным текстом, а tool call:

```json
{
  "tool_calls": [
    {
      "name": "get_weather",
      "arguments": {
        "city": "Москва",
        "date": "2026-05-29"
      }
    }
  ]
}
```

Шаг 3. Приложение вызывает реальный weather API.

Шаг 4. Результат tool call отправляется обратно модели:

```json
{
  "role": "tool",
  "name": "get_weather",
  "content": "{\"temp_c\": 18, \"condition\": \"rain\"}"
}
```

Шаг 5. Модель формирует финальный ответ пользователю.

### 7. Формы tool invocation

LLM может инициировать tool use в разных форматах:
- JSON function call;
- XML-like разметка;
- специальные токены модели;
- текстовый ReAct формат `Action: search[...]`;
- structured output с полем `tool_name`;
- выбор action из фиксированного enum;
- план, который затем исполняет внешний orchestrator.

Для production лучше формат, который легко валидировать и логировать.

### 8. Agent observability и evaluation

Инструменты:
- LangSmith;
- Langfuse;
- Arize Phoenix;
- OpenTelemetry traces;
- Helicone/Portkey-подобные proxy;
- RAGAS/DeepEval для offline eval;
- собственные dashboards по логам.

Что мониторить:
- полный trace шагов агента;
- входные/выходные сообщения;
- tool calls и arguments;
- latency каждого tool;
- token usage и cost;
- retrieval top-k и score;
- reranker score;
- долю retries;
- tool errors/timeouts;
- hallucination/faithfulness judge;
- citation coverage;
- user feedback;
- зацикливания агента;
- privacy/security violations.

Для агентского RAG важна не только оценка финального ответа, но и диагностика: агент выбрал неправильный инструмент, плохо переписал запрос, retrieval не нашел источник или генератор проигнорировал контекст.

## Типичные ошибки

- Давать агенту слишком много инструментов без router/permissions.
- Полагаться на prompt вместо schema validation.
- Считать function calling реальным вызовом API внутри модели.
- Не логировать tool arguments и потом не понимать, почему агент ошибся.
- Делать свободный ReAct там, где нужен простой state machine.
- Оценивать только финальный текст без трассировки шагов.

## Проверка себя

- Чем agentic RAG отличается от обычного RAG?
- Какие параметры можно передавать в RAG-tool кроме текста запроса?
- Почему structured output через prompt не гарантирует валидный JSON?
- Как выглядит цикл function calling?
- Что смотреть в observability агента?

## Связано

- [[NLP/LLM и Промпт-инжиниринг/RAG — retrieval, hybrid search, RRF и embeddings]]
- [[NLP/LLM и Промпт-инжиниринг/RAG — генерация, оценка качества и production]]

## Источники

- Sber/Yandex interview notes

---
[[🗺️ Индекс|Назад к разделу]]
