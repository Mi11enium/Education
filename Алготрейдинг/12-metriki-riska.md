# Урок 12. Метрики риска

## Зачем
PnL сам по себе не говорит о **риске**. $1000 за неделю с просадкой 50% хуже, чем $500 за ту же неделю с просадкой 5%.

## Drawdown (Просадка)

### Определение
Падение equity от предыдущего максимума.

```python
equity = pnl.cumsum()
running_max = equity.cummax()
drawdown = (equity - running_max) / running_max  # %
```

### Max Drawdown (MaxDD)
Максимальная просадка за период.
- Главная метрика риска для большинства трейдеров.
- Влияет на психологию и риск разорения.

### Avg Drawdown
Средняя просадка.

### Drawdown Duration
Сколько времени equity ниже максимума.

### Recovery Factor
```
Recovery = Total_PnL / Max_DD
```

- Чем выше, тем лучше.

## Volatility (Волатильность)

### Дневная
```
σ_daily = returns.std()
```

### Годовая
```
σ_annual = σ_daily × sqrt(252)
```

### Полезные уровни
- <10% годовых — низкая.
- 10–20% — средняя.
- 20–40% — высокая.
- >40% — очень высокая (только для агрессивных).

## VaR и CVaR

### VaR (Value at Risk)
Максимальный убыток с вероятностью (1 - α) за период.

```python
import numpy as np
returns = pnl / initial_capital
var_95 = np.percentile(returns, 5)
```

### CVaR (Expected Shortfall)
Средний убыток в худших (1-α)% случаев.

```python
cvar_95 = returns[returns <= var_95].mean()
```

- CVaR > VaR (хвостовой риск).

## Beta и корреляция с рынком

### Beta
```
β = cov(strategy, market) / var(market)
```

- β=1: движется с рынком.
- β=0: независим.
- β<0: хедж.

### Correlation
- ρ с бенчмарком (S&P 500, MOEX, BTC).
- Помогает понять, нужен ли хедж.

## Stress-сценарии

### Исторические
- 2008, 2020 COVID, Flash Crash 2010, Archegos 2021.

### Синтетические
- «Если рынок упадёт на 20% за день...».
- «Если позиция не закроется...».

## Risk-Adjusted метрики (вводная)

### Sharpe (см. урок 14)
```
Sharpe = (R - Rf) / σ
```

### Sortino (см. урок 15)
Только downside volatility.

### Calmar (см. урок 16)
```
Calmar = CAGR / |MaxDD|
```

## Дополнительные метрики

### Ulcer Index
```
UI = sqrt(mean(drawdown²))
```

### Sterling Ratio
```
Sterling = CAGR / Avg_Annual_DD
```

### Burke Ratio
```
Burke = CAGR / sqrt(sum(drawdown²))
```

## R-Multiple

### Определение
PnL сделки в единицах первоначального риска:
```
R = PnL / Risk
```

- Risk = размер стоп-лосса (в $).
- Если risk = $100 и PnL = $250, R = 2.5.

### Total R
Сумма R-multiple всех сделок.
- Total R > 0 → edge.
- Total R < 0 → нет edge.

## Годовые метрики

### CAGR (Compound Annual Growth Rate)
```
CAGR = (Final / Initial)^(1/years) - 1
```

## Чек-лист урока
- [ ] Считаете MaxDD и время в просадке.
- [ ] Volatility annualized.
- [ ] VaR/CVaR для хвостового риска.
- [ ] Знаете beta/correlation с рынком.
- [ ] Применяете R-Multiple анализ.