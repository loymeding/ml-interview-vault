---
topic: RAG
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/RAG|RAG]]"
generated: 2026-05-19
source: "user-provided extra mock questions"
---

## Q001
type: system_design
difficulty: middle
key_concepts: [retrieval, chunking, embeddings/vector search, generation with context]
optional_concepts: [reranking, hybrid search]
text: Как работает RAG?
follow_up:
  - Где в RAG pipeline чаще всего возникают ошибки качества?
  - Как проверить, что проблема в retrieval, а не в генерации?

## Q002
type: compare
difficulty: middle
key_concepts: [актуализация знаний, ссылки на источники, меньше hallucinations, без полного fine-tuning]
optional_concepts: [access control, стоимость обновления базы]
text: В чём преимущества использования системы RAG?
follow_up:
  - Почему RAG удобен для часто меняющейся документации?
  - Какие риски RAG не решает полностью?

## Q003
type: scenario
difficulty: middle
key_concepts: [изменение поведения модели, стиль/формат ответов, domain adaptation, retrieval не нужен для знания]
optional_concepts: [LoRA, SFT dataset]
text: Когда лучше использовать Fine-tuning вместо RAG?
follow_up:
  - Почему fine-tuning не лучший способ добавлять часто обновляемые факты?
  - Когда RAG и fine-tuning стоит совмещать?

## Q004
type: fact
difficulty: middle
key_concepts: [naive RAG, advanced RAG, agentic RAG, hybrid search]
optional_concepts: [GraphRAG, multi-hop retrieval]
text: Типы RAG систем.
follow_up:
  - Чем agentic RAG отличается от простого retrieve-then-generate?
  - Когда нужен reranker или multi-hop retrieval?
