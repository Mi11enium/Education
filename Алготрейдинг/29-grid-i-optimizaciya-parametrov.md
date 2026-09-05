# Урок 29. Оптимизация параметров

## Зачем
Стратегии обычно имеют параметры (MA period, RSI threshold, ATR multiplier). Нужно найти «хорошие» значения.

## Типы оптимизации

### 1. Grid Search
- Перебор всех комбинаций.
- ✅ Простой, полный.
- ❌ Экспоненциально растёт с числом параметров.

```python
import itertools

params = {
    'fast': [5, 10, 15, 20],
    'slow': [30, 50, 70, 100]
}

results = []
for fast, slow in itertools.product(*params.values()):
    metrics = backtest(strategy, {'fast': fast, 'slow': slow})
    results.append(((fast, slow), metrics))
```

### 2. Random Search
- Случайный набор параметров.
- ✅ Быстрее Grid.
- ✅ Часто лучше для high-dim.

```python
import random
for _ in range(1000):
    fast = random.randint(5, 50)
    slow = random.randint(50, 200)
    ...
```

### 3. Bayesian Optimization
- Строит surrogate model (GP, TPE).
- Выбирает следующую точку на основе предыдущих.
- ✅ Эффективнее.
- ❌ Сложнее.

### Optuna (TPE-based)
```python
import optuna

def objective(trial):
    fast = trial.suggest_int('fast', 5, 50)
    slow = trial.suggest_int('slow', 50, 200)
    metrics = backtest(strategy, {'fast': fast, 'slow': slow})
    return metrics['sharpe']

study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=200)
print(study.best_params)
```

### 4. Genetic Algorithm
- Эволюционный подход.
- Работает с population параметров.

## Защита от переобучения при оптимизации

### 1. Walk-Forward
- Train / test на каждом окне.
- См. урок 25.

### 2. In-sample + Out-of-sample
- Оптимизация на in-sample.
- Проверка на out-of-sample.
- WFE > 0.5.

### 3. Плато параметров
- Ищите **область**, где Sharpe хороший.
- ❌ Не выбирайте острый пик.

### 4. Robustness test
- Проверить чувствительность: ±10% к параметрам.
- Если результат не меняется — robust.

### 5. Cross-validation
- K-fold (с purging).

## Heatmap параметров

### Визуализация 2D
```python
import seaborn as sns
import numpy as np

results_matrix = np.zeros((len(fast_range), len(slow_range)))
for i, fast in enumerate(fast_range):
    for j, slow in enumerate(slow_range):
        results_matrix[i, j] = backtest(...).sharpe

sns.heatmap(results_matrix, xticklabels=slow_range, yticklabels=fast_range)
```

### 3D plot
```python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(...)
```

## In-Sample Optimization Bias

### Проблема
Оптимизация на in-sample завышает результат. **OOS** всегда хуже.

### Decay factor
```
Expected_OOS = 0.5 × IS_Sharpe  (примерно)
```

### Реалистичные ожидания
- IS Sharpe 2.0 → OOS 0.5–1.0.
- IS Sharpe 1.0 → OOS 0.3–0.7.

## Regularization

### Идея
Штрафовать за сложность модели.

### AIC / BIC
```python
aic = 2 * n_params - 2 * log_likelihood
bic = n_params * log(n) - 2 * log_likelihood
```

- Лучше — меньше AIC/BIC.

### Early stopping
- В ML: остановить обучение, когда validation начинает расти.

## Hyperband / Successive Halving
- Распределение budget между конфигурациями.
- Лучшим — больше ресурсов.

## Robustness через эмпирические правила

### 1. Экономическая интуиция
- MA(20) = месяц.
- MA(50) = квартал.
- MA(200) = год.
- Не выбирайте MA(73).

### 2. Round numbers
- Предпочитайте 10, 20, 50, 100, 200.

### 3. Параметры, устойчивые к слегка изменённым условиям
- Если оптимум MA = 23.7 — fragile.
- Если MA от 15 до 30 работает — robust.

## Когда не оптимизировать

- ❌ Стратегия и так хороша.
- ❌ Параметры имеют экономический смысл.
- ❌ Данных мало (overfit).

## Чек-лист урока
- [ ] Применяется walk-forward оптимизация.
- [ ] WFE > 0.5.
- [ ] Параметры стабильны.
- [ ] Используется плато, не пик.
- [ ] Экономическое обоснование параметров.