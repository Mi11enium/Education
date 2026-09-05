# Урок 34. Bayesian Optimization

## Что это
**Bayesian Optimization (BO)** — метод оптимизации чёрного ящика, использующий байесовские модели для эффективного поиска оптимума.

## Зачем в трейдинге
- Grid Search: перебор всех точек — медленно.
- Random Search: лучше, но не использует информацию.
- ✅ BO: использует результаты предыдущих оценок.

## Основные компоненты

### 1. Surrogate Model
Аппроксимация целевой функции:
- **Gaussian Process (GP)** — классический выбор.
- **Random Forest** — TPE.
- **TPE (Tree-structured Parzen Estimator)** — в Optuna.

### 2. Acquisition Function
Решает, какую точку оценить следующей:
- **Expected Improvement (EI)** — баланс эксплуатации и исследования.
- **Upper Confidence Bound (UCB)** — аналог.
- **Probability of Improvement (PI)** — вероятность улучшения.

### 3. Search Space
- Continuous: `[0, 1]`.
- Discrete: `['A', 'B', 'C']`.
- Conditional: «если выбран A, то B = X».

## Optuna

### Установка
```bash
pip install optuna
```

### Базовый пример
```python
import optuna

def objective(trial):
    # Параметры
    fast = trial.suggest_int('fast', 5, 50)
    slow = trial.suggest_int('slow', 30, 200)
    
    # Backtest
    metrics = backtest({'fast': fast, 'slow': slow})
    return metrics['sharpe']

study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=200)
print(study.best_params, study.best_value)
```

### Pruning (раннее завершение)
```python
def objective(trial):
    fast = trial.suggest_int('fast', 5, 50)
    slow = trial.suggest_int('slow', 30, 200)
    
    for step in range(100):
        metrics = backtest_partial({'fast': fast, 'slow': slow}, step)
        trial.report(metrics['sharpe'], step)
        if trial.should_prune():
            raise optuna.TrialPruned()
    return metrics['sharpe']

study = optuna.create_study(
    direction='maximize',
    pruner=optuna.pruners.MedianPruner()
)
```

### Multi-objective
```python
def objective(trial):
    fast = trial.suggest_int('fast', 5, 50)
    slow = trial.suggest_int('slow', 30, 200)
    metrics = backtest({'fast': fast, 'slow': slow})
    return metrics['sharpe'], -abs(metrics['maxdd'])

study = optuna.create_study(directions=['maximize', 'minimize'])
```

### Visualisation
```python
optuna.visualization.plot_optimization_history(study)
optuna.visualization.plot_param_importances(study)
optuna.visualization.plot_contour(study)
```

## Hyperopt (альтернатива)

```python
from hyperopt import fmin, tpe, hp, STATUS_OK

space = {
    'fast': hp.quniform('fast', 5, 50, 1),
    'slow': hp.quniform('slow', 30, 200, 5)
}

def objective(params):
    metrics = backtest(params)
    return {'loss': -metrics['sharpe'], 'status': STATUS_OK}

best = fmin(objective, space, algo=tpe.suggest, max_evals=200)
```

## Сравнение подходов

| Метод | Скорость | Гибкость | Качество |
|---|---|---|---|
| Grid | ❌ Медленно | Низкая | Хорошо |
| Random | Средне | Средняя | Средне |
| BO (Optuna) | ✅ Быстро | Высокая | Отлично |
| GA | Средне | Высокая | Хорошо |

## Anti-overfitting при BO

### 1. Train / Test Split
- BO на train.
- Валидация на test.

### 2. Walk-forward
- BO на каждом окне.
- Склеенный OOS — главная метрика.

### 3. Robust objective
- Не просто Sharpe.
- `Sharpe - lambda × n_params - mu × overfit_score`

### 4. K-fold CV
- Среднее по фолдам.

```python
def cv_objective(trial):
    params = suggest_params(trial)
    scores = []
    for train, val in cv_splits:
        scores.append(backtest(train, val, params)['sharpe'])
    return np.mean(scores) - np.std(scores)  # mean - std
```

## Когда НЕ использовать BO

- ❌ Менее 3 параметров — Grid проще.
- ❌ Очень быстрый backtest — random OK.
- ❌ Неизвестно пространство — random.

## Когда использовать

- ✅ 5+ параметров.
- ✅ Дорогой backtest (>1 мин на 1 оценку).
- ✅ Continuous parameters.
- ✅ Нет аналитического решения.

## Production pipeline

```python
# 1. Define search space
space = {
    'fast_period': (5, 50),
    'slow_period': (30, 200),
    'rsi_period': (5, 30),
    'threshold': (0.5, 0.95),
}

# 2. Walk-forward + BO
results = []
for train_data, test_data in walk_forward_splits:
    study = optuna.create_study()
    study.optimize(
        lambda trial: objective(trial, train_data),
        n_trials=100
    )
    best = study.best_params
    test_metrics = backtest(test_data, best)
    results.append((best, test_metrics))

# 3. Aggregate
total_sharpe = aggregate(results)
print(f'Walk-forward Sharpe: {total_sharpe}')
```

## Чек-лист урока
- [ ] Optuna настроена.
- [ ] Search space имеет экономический смысл.
- [ ] Pruning включён.
- [ ] Walk-forward + BO применены.
- [ ] Robust objective (Sharpe - penalty).