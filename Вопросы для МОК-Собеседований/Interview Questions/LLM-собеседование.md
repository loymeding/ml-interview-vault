---
topic: LLM-собеседование
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/LLM-собеседование|LLM-собеседование]]"
generated: 2026-05-19
source: "user-provided extra mock questions"
---

## Q001
type: fact
difficulty: middle
key_concepts: [начальные символы/байты, частотные пары, итеративные merge-операции, vocabulary size]
optional_concepts: [byte-level BPE, UNK handling]
text: Опишите алгоритм BPE токенизации. Как строится словарь?
follow_up:
  - Почему BPE хорошо работает с редкими словами?
  - Что будет с очень частыми морфемами в процессе merge?

## Q002
type: fact
difficulty: middle
key_concepts: [порядок merge-операций, детерминированная токенизация, сжатие частых пар, воспроизводимость словаря]
optional_concepts: [rank merges]
text: Зачем в BPE нужна merge table?
follow_up:
  - Почему одного vocabulary недостаточно для токенизации?
  - Как merge table применяется к новому слову?

## Q003
type: fact
difficulty: middle
key_concepts: [Query, Key, Value, attention weights, weighted sum]
optional_concepts: [scaled dot-product attention, masking]
text: Объясните механизм self-attention. Что такое Q, K, V?
follow_up:
  - Как из Q и K получаются веса внимания?
  - Почему self-attention позволяет токенам учитывать контекст друг друга?

## Q004
type: fact
difficulty: middle
key_concepts: [масштаб dot product, стабильность softmax, размерность ключей, градиенты]
optional_concepts: [variance of dot product]
text: Зачем в attention делим на sqrt(d_k)?
follow_up:
  - Что произойдёт с softmax без масштабирования при большом d_k?
  - Почему делим именно на корень из размерности?

## Q005
type: fact
difficulty: middle
key_concepts: [несколько attention heads, разные подпространства признаков, параллельные паттерны внимания, concat heads]
optional_concepts: [head dimension, specialization heads]
text: В чём смысл multi-head attention?
follow_up:
  - Почему один большой head не полностью заменяет несколько head?
  - Какие разные зависимости могут ловить разные головы?

## Q006
type: compare
difficulty: middle
key_concepts: [absolute position, relative distance, обобщение на длины, инъекция порядка токенов]
optional_concepts: [sinusoidal embeddings, learned embeddings]
text: В чём разница между абсолютными и относительными позиционными энкодингами?
follow_up:
  - Почему Transformer без positional encoding не знает порядок токенов?
  - Когда относительные позиции могут обобщаться лучше абсолютных?

## Q007
type: fact
difficulty: middle
key_concepts: [rotary embeddings, поворот Q/K, относительная позиционная информация, экстраполяция контекста]
optional_concepts: [complex plane interpretation, NTK scaling]
text: Что такое RoPE (Rotary Position Embedding)? В чём его преимущество?
follow_up:
  - Почему RoPE применяется к Q и K, а не просто добавляется к embeddings?
  - Как RoPE кодирует относительное смещение?

## Q008
type: scenario
difficulty: middle
key_concepts: [encoder-only architecture, CLS representation, classification head, fine-tuning]
optional_concepts: [pooling strategies, frozen encoder]
text: Как адаптировать encoder-only модель (BERT) для классификации?
follow_up:
  - Что именно подаётся в classification head?
  - Когда стоит заморозить encoder и обучать только head?

## Q009
type: fact
difficulty: middle
key_concepts: [разные LR по слоям, меньший LR для нижних слоёв, fine-tuning stability, сохранение pretrained features]
optional_concepts: [discriminative fine-tuning]
text: Что такое layer-wise learning rate decay?
follow_up:
  - Почему нижние слои обычно меняют осторожнее?
  - Как выбрать decay factor между слоями?

## Q010
type: compare
difficulty: middle
key_concepts: [CLS token representation, mean/max pooling, агрегация токенов, classification head]
optional_concepts: [attention pooling]
text: В чём разница между CLS токеном и pooling для классификации?
follow_up:
  - Когда mean pooling может быть лучше CLS?
  - Как padding mask влияет на pooling?

## Q011
type: fact
difficulty: middle
key_concepts: [pretraining, supervised fine-tuning, preference/alignment training, instruction following]
optional_concepts: [RLHF, DPO]
text: Опишите три основных этапа обучения современных LLM.
follow_up:
  - Чем pretraining objective отличается от SFT objective?
  - Почему alignment этап нужен после SFT?

## Q012
type: fact
difficulty: middle
key_concepts: [instruction following, демонстрационные ответы, стартовая политика, стабилизация preference training]
optional_concepts: [reward model]
text: Зачем нужен SFT перед RLHF?
follow_up:
  - Почему RLHF с плохой начальной политикой сложнее?
  - Что модель обычно учит на SFT-датасете?

## Q013
type: compare
difficulty: middle
key_concepts: [PPO with reward model, DPO direct preference optimization, SimPO без reference model, preference pairs]
optional_concepts: [KL regularization, reward hacking]
text: Объясните историю развития alignment-схем: PPO, DPO, SimPO. В чём различия?
follow_up:
  - Почему DPO проще пайплайна RLHF с PPO?
  - В чём идея отказа от reward model?

## Q014
type: fact
difficulty: middle
key_concepts: [заморозка базовых весов, low-rank matrices A и B, delta W, parameter-efficient fine-tuning]
optional_concepts: [rank r, alpha scaling]
text: Что такое LoRA? Как работает low-rank decomposition?
follow_up:
  - Почему LoRA экономит память при fine-tuning?
  - Как rank r влияет на выразительность адаптера?

## Q015
type: fact
difficulty: middle
key_concepts: [одна матрица random init, вторая матрица zero init, нулевой начальный delta W, сохранение поведения base model]
optional_concepts: [A/B convention зависит от реализации]
text: Как инициализируются матрицы в LoRA и зачем так делать?
follow_up:
  - Почему важно, чтобы в начале LoRA не меняла выход модели?
  - Что будет, если обе матрицы инициализировать нулями?

## Q016
type: fact
difficulty: middle
key_concepts: [attention projection layers, q_proj/k_proj/v_proj/o_proj, MLP projections, tradeoff качество/память]
optional_concepts: [target_modules, all-linear]
text: К каким слоям обычно применяют LoRA?
follow_up:
  - Почему часто начинают с Q и V projections?
  - Когда имеет смысл добавлять LoRA в MLP-слои?

## Q017
type: fact
difficulty: middle
key_concepts: [4-bit quantization, замороженная квантованная base model, LoRA adapters, dequantization for compute]
optional_concepts: [NF4, double quantization]
text: Что такое QLoRA? Как работает квантизация?
follow_up:
  - Почему QLoRA позволяет fine-tune большие модели на меньшей VRAM?
  - Какие ошибки может вносить квантизация?

## Q018
type: scenario
difficulty: middle
key_concepts: [rank r, ёмкость адаптера, переобучение, валидация и бюджет памяти]
optional_concepts: [alpha/r scaling, per-layer rank]
text: Как выбрать ранг r в LoRA?
follow_up:
  - Что будет при слишком маленьком rank?
  - Почему большой rank не всегда лучше?

## Q019
type: fact
difficulty: middle
key_concepts: [потеря прежних способностей, fine-tuning на узком датасете, distribution shift, регуляризация/смесь данных]
optional_concepts: [replay data, EWC]
text: Что такое catastrophic forgetting в контексте LLM?
follow_up:
  - Как заметить catastrophic forgetting на evals?
  - Какие методы помогают его уменьшить?

## Q020
type: fact
difficulty: middle
key_concepts: [важность параметров, Fisher information, штраф за изменение важных весов, continual learning]
optional_concepts: [quadratic penalty]
text: Опишите метод Elastic Weight Consolidation (EWC).
follow_up:
  - Почему не все веса нужно защищать одинаково?
  - Как EWC связан с catastrophic forgetting?

## Q021
type: fact
difficulty: middle
key_concepts: [temperature scaling logits, случайность генерации, entropy distribution, детерминизм при низкой температуре]
optional_concepts: [temperature 0, diversity vs quality]
text: Температура и влияние на генерацию.
follow_up:
  - Что произойдёт при повышении temperature?
  - Почему слишком высокая temperature ухудшает связность текста?

## Q022
type: compare
difficulty: middle
key_concepts: [top-k fixed number tokens, top-p cumulative probability, sampling distribution truncation, diversity control]
optional_concepts: [tail tokens, combined top-k/top-p]
text: Nucleus sampling (top-p) vs top-k.
follow_up:
  - Почему top-p адаптивнее top-k?
  - Когда top-k может отрезать полезные токены?

## Q023
type: fact
difficulty: middle
key_concepts: [несколько гипотез, beam width, максимизация sequence score, length bias]
optional_concepts: [length penalty, diverse beam search]
text: Beam search.
follow_up:
  - Почему beam search часто хуже для открытой генерации?
  - Как length penalty влияет на результат?

## Q024
type: fact
difficulty: middle
key_concepts: [cache keys and values, ускорение autoregressive decoding, память растёт с context length, prefill vs decode]
optional_concepts: [multi-query attention, paged attention]
text: Что такое KV-cache?
follow_up:
  - Почему KV-cache не нужен при обучении так же, как при decoding?
  - Как длина контекста влияет на память KV-cache?
