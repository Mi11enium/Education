# Урок 26. Monte Carlo симуляции

## Зачем
Бэктест даёт **одну** equity curve. Monte Carlo строит тысячи, чтобы оценить **диапазон** возможных исходов.

## Виды Monte Carlo в трейдинге

### 1. Перемешивание сделок (Bootstrap trades)
- Берём N сделок из истории.
- Случайно сэмплируем с возвращением N сделок.
- Строим equity curve.
- Повторяем 1000+ раз.

### 2. Перемешивание дней
- Если PnL = f(день) — перемешиваем дни.

### 3. Bootstrap returns
- Ресэмплинг returns с возвращением.

### 4. Синтетические данные
- Генерация из предполагаемого распределения.

## Реализация

### Базовый bootstrap
```python
import numpy as np

def mc_bootstrap(trades, n_simulations=1000, n_trades=None):
    if n_trades is None:
        n_trades = len(trades)
    
    final_pnls = []
    for _ in range(n_simulations):
        # Сэмплируем с возвращением
        sim_trades = np.random.choice(trades, size=n_trades, replace=True)
        equity = np.cumsum(sim_trades)
        final_pnls.append(equity[-1])
    
    return final_pnls
```

### Bootstrap с датами
```python
def mc_with_dates(daily_returns, n_sims=1000):
    sim_equities = []
    for _ in range(n_sims):
        sim = np.random.choice(daily_returns, size=len(daily_returns), replace=True)
        equity = np.cumprod(1 + sim)
        sim_equities.append(equity)
    return np.array(sim_equities)
```

## Анализ результатов

### 1. Confidence Intervals
```python
ci_low, ci_high = np.percentile(final_pnls, [2.5, 97.5])
```

### 2. Probability of Profit
```python
pop = np.mean(np.array(final_pnls) > 0)
```

### 3. Distribution
- Гистограмма final PnL.
- Median, percentiles.

## Конкретные применения

### 1. Оценка MaxDD
- В каждой симуляции считаем MaxDD.
- Получаем распределение MaxDD.

```python
def mc_maxdd(trades, n_sims=1000):
    dd_list = []
    for _ in range(n_sims):
        sim = np.random.choice(trades, size=len(trades), replace=True)
        equity = np.cumsum(sim)
        running_max = np.maximum.accumulate(equity)
        dd = (equity - running_max) / running_max
        dd_list.append(dd.min())
    return dd_list
```

### 2. Оценка Risk of Ruin
- Доля симуляций, где equity < X (margin call level).

```python
def mc_risk_of_ruin(trades, ruin_level, n_sims=1000):
    ruin_count = 0
    for _ in range(n_sims):
        sim = np.random.choice(trades, size=len(trades), replace=True)
        equity = np.cumsum(sim)
        if np.any(equity < ruin_level):
            ruin_count += 1
    return ruin_count / n_sims
```

### 3. Time to Recovery
- В каждой симуляции — сколько времени equity ниже максимума.

## Проблема автокорреляции

### Перемешивание независимых сделок
- ❌ Не учитывает кластеризацию убытков.

### Решения
1. **Блочный bootstrap**: перемешивать блоки сделок.
2. **GARCH-моделирование**: учитывать волатильность.
3. **Markov chain**: моделировать режимы.

### Блочный bootstrap
```python
def block_bootstrap(returns, block_size=5, n_sims=1000):
    n = len(returns)
    n_blocks = n // block_size
    sims = []
    for _ in range(n_sims):
        sim = []
        for _ in range(n_blocks):
            start = np.random.randint(0, n - block_size)
            sim.extend(returns[start:start + block_size])
        sims.append(np.array(sim))
    return sims
```

## Синтетический Monte Carlo

### Из предполагаемого распределения
```python
def synthetic_mc(mu, sigma, n_sims=1000, n_days=252):
    sims = []
    for _ in range(n_sims):
        # Нормальное распределение
        daily_returns = np.random.normal(mu/252, sigma/np.sqrt(252), n_days)
        equity = np.cumprod(1 + daily_returns)
        sims.append(equity)
    return sims
```

### С толстыми хвостами
```python
def t_dist_returns(df, loc, scale, n):
    return stats.t.rvs(df, loc=loc, scale=scale, size=n)
```

## Stress Monte Carlo

### Что если
- Волатильность × 2.
- Среднее = 0.
- Slippage × 3.
- Win Rate = 0.4 (хуже).

```python
def stress_mc(trades, scenarios):
    results = {}
    for name, params in scenarios.items():
        modified_trades = trades * params.get('mean_factor', 1)
        if 'vol_factor' in params:
            modified_trades = modified_trades * params['vol_factor']
        # Run MC
        results[name] = mc_bootstrap(modified_trades)
    return results
```

## Что вы узнаёте из Monte Carlo

### 1. Реалистичные ожидания
- 95% CI final PnL.
- Медиана вместо «лучшего» сценария.

### 2. Probability of Profit
- В 70% случаев стратегия прибыльна → нормально.
- В 30% — задумайтесь.

### 3. Worst-case scenarios
- 99% percentile MaxDD.
- Stress tests.

### 4. Надёжность оценки
- Чем уже CI — тем надёжнее стратегия.

## Чек-лист урока
- [ ] Реализован bootstrap MC.
- [ ] Получены CI на final PnL и Sharpe.
- [ ] Оценён MaxDD через MC.
- [ ] Risk of Ruin рассчитан.
- [ ] Учтена автокорреляция (блочный bootstrap).