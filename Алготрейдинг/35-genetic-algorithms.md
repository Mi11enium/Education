# Урок 35. Генетические алгоритмы в оптимизации

## Что это
**Genetic Algorithm (GA)** — эвристика оптимизации, вдохновлённая эволюцией:
- Популяция «индивидов» (наборов параметров).
- Отбор лучших.
- Скрещивание (crossover).
- Мутации.
- Итерации → лучшее решение.

## Применение в трейдинге

### Когда использовать
- ✅ Много параметров (>10).
- ✅ Нет аналитического решения.
- ✅ Сложный ландшафт целевой функции.
- ✅ Нужно несколько «хороших» решений (не одно).

### Когда не использовать
- ❌ Мало параметров — Grid проще.
- ❌ Один оптимум (BO лучше).
- ❌ Нужна стабильность — GA стохастичен.

## Базовая схема

```python
import random
import numpy as np

def fitness(params):
    return backtest(params)['sharpe']

def init_population(size, gene_ranges):
    return [
        {k: random.uniform(v[0], v[1]) for k, v in gene_ranges.items()}
        for _ in range(size)
    ]

def select(population, scores, k):
    # Tournament selection
    selected = []
    for _ in range(k):
        i, j = random.sample(range(len(population)), 2)
        winner = i if scores[i] > scores[j] else j
        selected.append(population[winner])
    return selected

def crossover(parent1, parent2, rate=0.5):
    child = {}
    for k in parent1:
        if random.random() < rate:
            child[k] = parent1[k]
        else:
            child[k] = parent2[k]
    return child

def mutate(individual, rate=0.1, sigma=0.1):
    for k in individual:
        if random.random() < rate:
            individual[k] *= (1 + random.gauss(0, sigma))
    return individual

def genetic_algorithm(gene_ranges, n_gen=50, pop_size=100):
    population = init_population(pop_size, gene_ranges)
    for gen in range(n_gen):
        scores = [fitness(p) for p in population]
        # Sort by fitness
        population = [p for _, p in sorted(zip(scores, population), reverse=True)]
        # Elitism
        elite = population[:5]
        # New generation
        new_pop = elite.copy()
        while len(new_pop) < pop_size:
            parents = select(population, scores, 2)
            child = crossover(parents[0], parents[1])
            child = mutate(child)
            new_pop.append(child)
        population = new_pop
    return population[0]  # best
```

## Параметры GA

### Population Size
- 50–200.
- Больше = разнообразнее, но медленнее.

### Mutation Rate
- 0.01–0.2.
- Высокая → exploration.
- Низкая → exploitation.

### Crossover Rate
- 0.5–0.9.

### Elitism
- 5–10% лучших переходят без изменений.
- Сохраняет лучшие решения.

### Selection
- **Tournament** (k=2–5).
- **Roulette**.
- **Rank-based**.

## Многоцелевая оптимизация (NSGA-II)

### Идея
Оптимизировать несколько метрик одновременно (Sharpe и MaxDD).

```python
# nsga2 (pymoo)
from pymoo.algorithms.moo.nsga2 import NSGA2
from pymoo.optimize import minimize

def objective(x):
    sharpe, maxdd = backtest(x)
    return -sharpe, abs(maxdd)

problem = Problem(n_var=3, n_obj=2, ...)
algo = NSGA2(pop_size=100)
res = minimize(problem, algo, ('n_gen', 50))
```

### Pareto Front
Множество недоминируемых решений. Позволяет выбрать компромисс.

## Ограничения

### 1. Premature Convergence
- Популяция «схлопывается» в локальный оптимум.
- Решение: увеличить mutation rate / diversity.

### 2. Стохастичность
- Разные запуски → разные результаты.
- Решение: несколько запусков, ensemble.

### 3. Fitness Evaluation дорогая
- Каждая оценка = backtest.
- Решение: caching, surrogate models.

### 4. Без градиента
- Не использует информацию о форме функции.

## GA + другие методы

### Гибрид
- GA для глобального поиска.
- Gradient descent / local search для уточнения.

### Memetic Algorithm
- GA + local search.
- Лучше сходимость.

## Robust GA

### Robust objective
```python
def robust_fitness(params):
    scores = []
    for data_subset in bootstrap_subsets(data, n=10):
        scores.append(backtest(params, data_subset)['sharpe'])
    return np.mean(scores) - 0.5 * np.std(scores)
```

### Multi-asset
- Тестировать на нескольких тикерах.
- Fitness = среднее по тикерам.

## Альтернативы

### CMA-ES
- Covariance Matrix Adaptation Evolution Strategy.
- ✅ Хорошо для continuous.
- ❌ Дороже.

### Differential Evolution
- Простой, эффективный.
- library: `scipy.optimize.differential_evolution`.

```python
from scipy.optimize import differential_evolution
result = differential_evolution(objective, bounds, maxiter=200)
```

## Чек-лист урока
- [ ] GA реализован для 5+ параметров.
- [ ] Elitism включён.
- [ ] Robust objective.
- [ ] Multi-asset тест.
- [ ] Сравнение с BO/Grid.