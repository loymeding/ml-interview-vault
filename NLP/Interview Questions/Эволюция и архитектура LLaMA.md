---
topic: Эволюция и архитектура LLaMA
card_ref: "[[Эволюция и архитектура LLaMA]]"
generated: 2026-05-20
---

## Q001
type: fact
difficulty: junior
key_concepts: [decoder-only Transformer, авторегрессия, causal mask]
optional_concepts: [instruction tuning]
text: Что означает, что LLaMA является decoder-only моделью, и почему такая архитектура подходит для генерации текста?
follow_up:
  - Что изменится, если убрать causal mask во время обучения авторегрессионной модели?

## Q002
type: compare
difficulty: junior
key_concepts: [RMSNorm, LayerNorm, масштаб активаций, центрирование]
optional_concepts: [стоимость вычислений]
text: Чем RMSNorm отличается от LayerNorm и почему LLaMA использует именно RMSNorm?
follow_up:
  - Если при обучении большой модели возникают нестабильные градиенты, почему одной замены LayerNorm на RMSNorm может быть недостаточно?

## Q003
type: fact
difficulty: junior
key_concepts: [SwiGLU, gate-механизм, FFN-блок]
optional_concepts: [Swish, поэлементное умножение]
text: Какую роль играет SwiGLU в feed-forward блоке LLaMA и чем gate-ветка полезна для модели?
follow_up:
  - Если заменить SwiGLU на ReLU, какие изменения в качестве или обучении можно ожидать и почему?

## Q004
type: trap
difficulty: junior
key_concepts: [RoPE, Query и Key, относительные расстояния, attention score]
optional_concepts: [абсолютные позиционные эмбеддинги]
text: Верно ли утверждение: “RoPE просто добавляет позиционный вектор к токен-эмбеддингу, как классические positional embeddings”? Объясните.
follow_up:
  - Что может пойти не так, если пытаться использовать модель с RoPE на контексте сильно длиннее обучающего без дополнительных техник адаптации?

## Q005
type: compare
difficulty: middle
key_concepts: [MHA, MQA, GQA, KV-cache]
optional_concepts: [качество против памяти]
text: Сравните Multi-Head Attention, Multi-Query Attention и Grouped Query Attention с точки зрения памяти KV-cache и качества генерации.
follow_up:
  - Какой подход вы бы выбрали для сервиса с длинным контекстом и большим числом одновременных пользователей, если небольшое падение качества допустимо?

## Q006
type: scenario
difficulty: middle
key_concepts: [Chinchilla-подход, баланс параметров и данных, недообученность, число токенов]
optional_concepts: [compute budget]
text: Команда хочет увеличить модель с 13B до 70B параметров, но оставить тот же объём обучающих данных. На внутренних бенчмарках прирост почти отсутствует. Что могло пойти не так?
follow_up:
  - Как бы вы аргументировали бизнесу, что иногда лучше увеличить объём и качество данных, а не размер модели?

## Q007
type: chain
difficulty: middle
key_concepts: [SFT, reward model, RLHF, LLaMA 2 Chat]
optional_concepts: [PPO, rejection sampling]
text: Опишите, как базовую LLaMA превратить в chat-модель: какие этапы нужны и какую проблему решает каждый из них?
follow_up:
  - Что может случиться, если SFT-датасет качественный, но reward model обучена на смещённых предпочтениях разметчиков?
  - Как бы вы диагностировали ситуацию, когда модель стала вежливее, но хуже отвечает на технические вопросы?

## Q008
type: tradeoff
difficulty: senior
key_concepts: [GQA, KV-cache, latency, качество генерации]
optional_concepts: [batching, длинный контекст]
text: Вы деплоите LLaMA-подобную модель в высоконагруженный чат-сервис. Бизнес требует снизить latency и стоимость инференса, но запрещает заметно ухудшать качество ответов. Как бы вы выбирали между MHA, GQA и MQA?
follow_up:
  - Что изменится в вашем решении, если основная нагрузка перейдёт с коротких диалогов на документы с контекстом 32k токенов?
