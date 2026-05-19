---
topic: SVM
card_ref: "[[Вопросы для МОК-Собеседований/Interview Questions/SVM|SVM]]"
generated: 2026-05-19
source: "Classic Machine Learning/Interview Questions/Вопросы для МОК-собеседования.md"
---

## Q001
type: compare
difficulty: middle
key_concepts: [максимизация margin, support vectors, primal formulation, dual formulation]
optional_concepts: [hinge loss]
source_refs:
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#1. Максимизация зазора|Маржа"
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#5. Primal vs Dual задача и алгоритм SMO|Primal/Dual"
text: Максимизация маржи, primal vs dual?
follow_up:
  - Почему решение SVM определяется только support vectors?
  - Когда dual formulation удобнее primal?

## Q002
type: fact
difficulty: middle
key_concepts: [soft margin, slack variables, параметр C, tradeoff margin vs violations]
optional_concepts: [hinge loss]
source_refs:
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#3. Параметр C (Регуляризация)|Ответ"
text: Soft-margin, параметр C?
follow_up:
  - Что произойдёт при очень большом C?
  - Как soft-margin помогает при неразделимых классах?

## Q003
type: fact
difficulty: middle
key_concepts: [kernel trick, RBF kernel, polynomial kernel, скалярное произведение в feature space]
optional_concepts: [линейное ядро]
source_refs:
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#4. Kernel Trick (Ядерный переход) и ядра|Ответ"
text: Ядра: линейное, RBF, полиномиальное, trick?
follow_up:
  - Почему kernel trick не требует явно строить новые признаки?
  - Как gamma в RBF влияет на сложность границы?

## Q004
type: fact
difficulty: middle
key_concepts: [feature space, нелинейное отображение, линейный разделитель в новом пространстве, kernel matrix]
optional_concepts: [размерность пространства признаков]
source_refs:
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#4. Kernel Trick (Ядерный переход) и ядра|Ответ"
text: Перевод задачи в косое пространство?
follow_up:
  - Почему линейная граница в feature space может быть нелинейной в исходном?
  - Какая цена у перехода к kernel matrix?

## Q005
type: fact
difficulty: middle
key_concepts: [квадратичная оптимизация, kernel matrix, SMO, масштабирование по числу объектов]
optional_concepts: [линейные SVM solvers]
source_refs:
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#5. Primal vs Dual задача и алгоритм SMO|SMO"
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#Каверзные вопросы|Сложность"
text: Сложность обучения O(N^3), SMO?
follow_up:
  - Почему kernel SVM плохо масштабируется на большие N?
  - Как SMO разбивает исходную задачу?

## Q006
type: fact
difficulty: middle
key_concepts: [class weights, дисбаланс классов, one-class SVM, novelty detection]
optional_concepts: [nu parameter]
source_refs:
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#6. Дисбаланс классов и One-class SVM|Ответ"
text: Class weights / one-class SVM?
follow_up:
  - Чем class weights отличаются от one-class SVM по постановке задачи?
  - Когда one-class SVM уместнее бинарного классификатора?

## Q007
type: fact
difficulty: middle
key_concepts: [kernel matrix memory, число support vectors, latency inference, масштабирование обучения]
optional_concepts: [approximate kernels]
source_refs:
  - "Classic Machine Learning/Модели/SVM (Метод опорных векторов)#Каверзные вопросы|Ответ"
text: Недостатки: память, масштабирование?
follow_up:
  - Почему много support vectors замедляют inference?
  - Как можно приблизить kernel SVM для больших данных?
