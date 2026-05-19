---
topic: PCA & линейная алгебра
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/PCA & линейная алгебра|PCA & линейная алгебра]]"
generated: 2026-05-19
source: "Classic Machine Learning/Interview Questions/Вопросы для МОК-собеседования.md"
---

## Q001
type: fact
difficulty: middle
key_concepts: [ковариационная матрица, собственные векторы, собственные значения, направления максимальной дисперсии]
optional_concepts: [центрирование данных]
source_refs:
  - "Classic Machine Learning/Снижение размерности/Понижение размерности (PCA, t-SNE, UMAP)#PCA (Principal Component Analysis)|Ответ"
text: Ковариационная матрица, собственные векторы?
follow_up:
  - Почему перед PCA нужно центрировать признаки?
  - Что означает первое главное направление?

## Q002
type: fact
difficulty: middle
key_concepts: [SVD разложение, экономия памяти, связь с PCA, сингулярные значения]
optional_concepts: [randomized SVD]
source_refs:
  - "Classic Machine Learning/Снижение размерности/Понижение размерности (PCA, t-SNE, UMAP)#PCA (Principal Component Analysis)|Ответ"
text: SVD как обход памяти?
follow_up:
  - Почему SVD можно считать численно устойчивым способом PCA?
  - Когда randomized SVD предпочтительнее полного разложения?

## Q003
type: fact
difficulty: middle
key_concepts: [explained variance ratio, собственные значения, выбор числа компонент, кумулятивная дисперсия]
optional_concepts: [scree plot]
source_refs:
  - "Classic Machine Learning/Снижение размерности/Понижение размерности (PCA, t-SNE, UMAP)#PCA (Principal Component Analysis)|Ответ"
text: Процент объяснённой дисперсии?
follow_up:
  - Почему 95% explained variance не всегда оптимальный критерий?
  - Как число компонент влияет на downstream модель?

## Q004
type: fact
difficulty: middle
key_concepts: [декорреляция компонент, единичная дисперсия, деление на сингулярные значения, потеря масштаба]
optional_concepts: [PCA whitening]
source_refs:
  - "Classic Machine Learning/Снижение размерности/Понижение размерности (PCA, t-SNE, UMAP)#PCA (Principal Component Analysis)|Ответ"
text: Что такое Whitening?
follow_up:
  - Зачем whitening может быть полезен перед моделью?
  - Почему whitening может усилить шум в малых компонентах?

## Q005
type: fact
difficulty: middle
key_concepts: [kernel trick, нелинейное снижение размерности, kernel matrix, pre-image problem]
optional_concepts: [RBF kernel]
source_refs:
  - "Classic Machine Learning/Снижение размерности/Понижение размерности (PCA, t-SNE, UMAP)#PCA (Principal Component Analysis)|Ответ"
text: Kernel-PCA?
follow_up:
  - Чем Kernel PCA отличается от обычного PCA?
  - Почему у Kernel PCA сложнее получить обратное преобразование?

## Q006
type: fact
difficulty: middle
key_concepts: [линейные проекции, масштаб признаков, чувствительность к выбросам, интерпретация компонент]
optional_concepts: [robust PCA]
source_refs:
  - "Classic Machine Learning/Снижение размерности/Понижение размерности (PCA, t-SNE, UMAP)#PCA (Principal Component Analysis)|Ответ"
text: Ограничения: линейность, масштабирование?
follow_up:
  - Почему PCA чувствителен к масштабу признаков?
  - Какие нелинейные структуры PCA не сможет развернуть?
