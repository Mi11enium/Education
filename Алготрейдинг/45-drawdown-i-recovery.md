# Урок 45. Drawdown и восстановление

## Что такое Drawdown
**Drawdown (DD)** — падение equity от предыдущего максимума.

```
DD(t) = (Equity(t) - Max_Equity(t)) / Max_Equity(t)
```

## Виды Drawdown

### MaxDD
Максимальная просадка за весь период.
- Главная метрика.
- Пример: -25%.

### AvgDD
Средняя просадка.
- Менее «пугающая».

### DD Duration
Сколько времени в просадке.

### Time to Recovery
Сколько времени до нового максимума.

## Recovery Factor
```
RF = Total_PnL / |MaxDD|
```

- RF > 3 — хорошо.
- RF < 1 — плохо (PnL меньше MaxDD).

## Психологические аспекты

### Уровни DD и психология
- -10%: дискомфорт.
- -20%: тревога.
- -30%: паника.
- -50%: «всё пропало».

### Решения
- Уменьшить размер.
- Закрыть стратегию.
- Психотерапевт.

## Подготовка к DD

### 1. Pre-trade capital allocation
- Не класть всё в одну стратегию.
- 50% в одной, 30% в другой, 20% cash.

### 2. Liquidity buffer
- Держать X% в cash.
- Для подстраховки / докупа.

### 3. MaxDD limit
- При DD > -20% → уменьшить size.
- При DD > -30% → остановить стратегию.

## Восстановление (Recovery)

### Скорость восстановления
```
Recovery_Time ≈ MaxDD / (avg_daily_return × trading_days)
```

### Пример
- MaxDD = -25%.
- Avg daily return = 0.1%.
- Recovery = 25 / 0.1 = 250 дней ≈ 1 год.

### Как ускорить
- Увеличить size (⚠️ рискованно).
- Добавить средств (внешний capital).
- Изменить стратегию.

## Underwater Curve

### Визуализация
```python
ax.fill_between(dd.index, dd * 100, 0, color='red', alpha=0.5)
ax.axhline(0, color='black', linestyle='--')
```

- Показывает % времени под водой.
- Хорошая стратегия: 30–50% времени в DD.

## Equity Curve Quality

### Smoothness
- Чем гладче equity, тем лучше.
- ❌ Горбы = нестабильность.

### Cointegration
- Slope / MaxDD.
- Чем выше, тем стабильнее.

### Profit-to-DD
```
Profit_to_DD = Total_Return / |MaxDD|
```

## DD-контроль на лету

### Auto-reduce
```python
def get_size_multiplier(current_dd):
    if current_dd < -0.10:
        return 0.75
    elif current_dd < -0.20:
        return 0.5
    elif current_dd < -0.30:
        return 0.25
    elif current_dd < -0.40:
        return 0  # STOP
    else:
        return 1.0
```

### Re-invest
- После восстановления — постепенно возвращать размер.

## Recovery Strategies

### 1. Добавление капитала
- Если уверены в стратегии.
- ⚠️ Только в «хорошие» стратегии.

### 2. Снижение размера
- ✅ Консервативный подход.

### 3. Hedging
- Купить опционы.
- Защита от падения.

### 4. Переход в cash
- Полностью выйти.
- Ждать лучших времён.

## MaxDD при оптимизации

### Penalties
- Sharpe - alpha × MaxDD.
- Или Calmar = CAGR / MaxDD.

### Multi-objective
- Sharpe и MaxDD как цели.
- Pareto front.

## Monte Carlo MaxDD

```python
def mc_maxdd_distribution(trades, n_sims=1000):
    maxdds = []
    for _ in range(n_sims):
        sim_trades = np.random.choice(trades, len(trades), replace=True)
        equity = np.cumsum(sim_trades) + 100000
        running_max = np.maximum.accumulate(equity)
        dd = (equity - running_max) / running_max
        maxdds.append(dd.min())
    return maxdds
```

## Сценарии «катастрофического DD»

### COVID-like
- DD может быть в 2–3× обычного.
- Если стратегия пережила COVID — robust.

### Flash Crash
- 1 день = -10%.
- Проверьте, есть ли stop.

## Drawdown vs. Strategy

### Trend following
- Типично: 20–30% DD.
- Восстановление: 3–6 мес.

### Mean reversion
- Типично: 10–20% DD.
- Быстрое восстановление.

### HFT
- DD < 5% (если работает).
- ❌ Если больше — algo-проблема.

## Чек-лист урока
- [ ] MaxDD < 25% для большинства стратегий.
- [ ] DD-контроль на лету.
- [ ] Recovery Factor > 3.
- [ ] План действий при DD > лимита.
- [ ] Monte Carlo оценка MaxDD.