---
topic: Эволюция семейства BERT (RoBERTa, ALBERT, ELECTRA, DeBERTa, XLNet)
card_ref: "[[Эволюция семейства BERT (RoBERTa, ALBERT, ELECTRA, DeBERTa, XLNet)]]"
generated: 2026-05-19
---

## Q001
type: fact
difficulty: junior
key_concepts: [MLM, NSP, train-test mismatch]
optional_concepts: [токен MASK]
text: Какие две задачи использовались при обучении оригинального BERT и почему позже к ним появились претензии?
follow_up:
  - Если downstream-задача не содержит токенов MASK, какая именно проблема возникает из-за MLM-предобучения?

## Q002
type: compare
difficulty: junior
key_concepts: [статическое маскирование, динамическое маскирование, разнообразие обучающих примеров]
optional_concepts: [эпохи обучения]
text: Чем динамическое маскирование в RoBERTa отличается от статического маскирования в BERT и почему это улучшает обучение?
follow_up:
  - Что изменится, если у нас маленький корпус и модель видит одни и те же тексты много эпох подряд?

## Q003
type: trap
difficulty: junior
key_concepts: [ALBERT, меньше параметров, скорость инференса]
optional_concepts: [cross-layer sharing]
text: Верно ли утверждение: «ALBERT быстрее BERT на инференсе, потому что у него намного меньше параметров»? Объясни.
follow_up:
  - Что будет узким местом, если ALBERT применяет один и тот же Transformer-блок много раз последовательно?

## Q004
type: compare
difficulty: middle
key_concepts: [NSP, SOP, тематический сдвиг, порядок предложений]
optional_concepts: [негативные примеры из одного документа]
text: Сравни NSP в BERT и SOP в ALBERT: какую слабость NSP исправляет SOP?
follow_up:
  - Если документы в корпусе плохо сегментированы и соседние предложения часто не связаны логически, как это повлияет на SOP?

## Q005
type: scenario
difficulty: middle
key_concepts: [ELECTRA, replaced token detection, обучающий сигнал на всех токенах]
optional_concepts: [генератор и дискриминатор]
text: Команда обучила BERT-подобную модель на небольшом бюджете. MLM loss выглядит нормально, но качество на downstream-задачах слабое, а обучение очень неэффективное: градиентный сигнал приходит только от малой части токенов. Какой подход из семейства BERT здесь уместен и почему?
follow_up:
  - Что изменится, если генератор в ELECTRA будет слишком сильным и почти не будет делать правдоподобных ошибок?

## Q006
type: chain
difficulty: middle
key_concepts: [RoBERTa, удаление NSP, масштаб данных, batch size]
optional_concepts: [долгое обучение]
text: Представь, что кандидат говорит: «RoBERTa лучше BERT, потому что у неё другая архитектура». Как бы ты последовательно разобрал это утверждение?
follow_up:
  - Если нельзя увеличить архитектуру, но можно увеличить данные и batch size, почему RoBERTa всё равно может выиграть?
  - Какие признаки в эксперименте помогли бы отделить вклад архитектуры от вклада процедуры обучения?

## Q007
type: trap
difficulty: senior
key_concepts: [XLNet, autoregressive pretraining, permutation language modeling, отсутствие MASK]
optional_concepts: [двунаправленный контекст без MLM]
text: Почему некорректно говорить, что XLNet — это просто BERT без NSP? Какую проблему MLM он пытается решить иначе?
follow_up:
  - Как XLNet пытается получить двунаправленный контекст, если формально использует авторегрессионную постановку?
  - Какой trade-off может появиться по сравнению с обычным MLM при усложнении процедуры предобучения?

## Q008
type: compare
difficulty: senior
key_concepts: [DeBERTa, disentangled attention, content embedding, position embedding]
optional_concepts: [relative position bias]
text: В чём идея DeBERTa по разделению смысла токена и его позиции, и чем это отличается от классического BERT-подхода к позиционным эмбеддингам?
follow_up:
  - Почему разделение content- и position-представлений может быть особенно важно для задач, где порядок слов меняет смысл?
  - Что может пойти не так при переносе модели на домен с сильно отличающимися длинами и структурами текстов?
