---
topic: Bias-Variance — RF vs GB
card_ref: "[[Bias-Variance — RF vs GB]]"
generated: 2026-04-24
---

## Q001
type: fact
difficulty: junior
key_concepts: [bias, variance, недообучение, переобучение]
text: Что такое Bias и Variance в контексте bias-variance tradeoff? Какой тип ошибки каждый из них описывает?
follow_up:
  - Как эти два типа ошибки связаны между собой — можно ли одновременно снизить оба?

## Q002
type: compare
difficulty: middle
key_concepts: [Random Forest, Gradient Boosting, усреднение, последовательная коррекция]
text: Random Forest и Gradient Boosting оба используют деревья решений, но решают разные проблемы. Объясните, какую именно составляющую ошибки снижает каждый алгоритм и за счёт какого механизма.
follow_up:
  - Почему RF снижает variance, но не bias — а GB снижает bias, но не variance?

## Q003
type: trap
difficulty: middle
key_concepts: [некоррелированность деревьев, bootstrap, max_features, variance]
text: Коллега говорит: «В Random Forest дисперсия снижается в T раз, потому что мы усредняем T деревьев — чем больше деревьев, тем лучше». Что в этом утверждении неточно или опасно упрощено?
follow_up:
  - Какой параметр RF отвечает за снижение корреляции между деревьями и почему он критичен?

## Q004
type: scenario
difficulty: senior
key_concepts: [шумные данные, early stopping, variance, переобучение GB]
text: Вы обучаете Gradient Boosting на датасете с высоким уровнем шума. После 500 итераций метрика на трейне продолжает расти, а на валидации — падать. Что происходит с точки зрения bias-variance, и какие конкретные шаги вы предпримете?
follow_up:
  - Какие гиперпараметры кроме числа итераций помогут контролировать variance в бустинге?
  - Стоит ли в этом сценарии рассмотреть переход на RF — почему да или нет?

## Q005
type: chain
difficulty: senior
key_concepts: [глубокое дерево, высокий variance, bagging, некоррелированность, RSM]
text: Представьте, что у вас одно глубокое дерево решений. Опишите по шагам, что именно делает Random Forest, чтобы превратить его в стабильную модель с низким variance.
follow_up:
  - Почему для снижения variance важно, чтобы деревья были именно некоррелированы, а не просто разные?
  - Что произойдёт с variance, если убрать случайный отбор признаков (RSM) и оставить только bootstrap?
