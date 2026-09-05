# Урок 33. Влияние издержек на стратегию

## Зачем учитывать
Бэктест «без издержек» — **иллюзия**. Каждый тик комиссии и slippage может превратить прибыльную стратегию в убыточную.

## Формула «эффективного» edge

```
Net_Edge = Gross_Edge - 2 × Commission - Slippage - Spread_Cost
```

### Пример
- Gross edge: 0.5 тика на сделку.
- Commission: 0.1 тика.
- Slippage: 0.1 тика.
- Spread: 0.25 тика (для market orders).
- Net: 0.5 - 0.1 - 0.1 - 0.25 = **0.05 тика**.

⚠️ 90% gross edge «съедено».

## Break-Even Win Rate

### С комиссией и slippage
```
BE_WR = (Avg_Loss + Cost) / (Avg_Win - Avg_Loss + 2 × Cost)
```

### Пример
- Avg_Win = $100, Avg_Loss = $50.
- Cost per trade = $10.
- BE_WR = (50 + 10) / (100 - 50 + 20) = 60/70 = **85.7%**.

⚠️ С издержками стратегия должна быть почти «идеальной».

## Влияние на Sharpe

### Sharpe ratio с издержками
```
Sharpe_net ≈ Sharpe_gross - (2 × Cost × sqrt(N)) / (σ × N)
```

где N — число сделок в год.

### Пример
- Sharpe gross = 2.0.
- Cost per trade = 0.2% (от equity).
- N = 200 trades/год.
- σ = 15%.
- Sharpe_net ≈ 2.0 - (0.4 × 14.1) / (0.15 × 200) = 2.0 - 0.19 = **1.81**.

### Чем чаще торгуете — тем больше издержки

## Издержки vs. стратегия

### Скальпинг (1000+ сделок/мес)
- Издержки = **30–50%** от gross PnL.
- ❌ Высокий риск.

### Интрадей (50–200/мес)
- Издержки = 10–25%.
- ⚠️ Средний риск.

### Свинг (5–20/мес)
- Издержки = 2–8%.
- ✅ Низкий риск.

### Позиционная (1–5/мес)
- Издержки < 2%.
- ✅✅ Минимальный.

## Оптимизация стратегии для снижения издержек

### 1. Увеличить holding period
- Меньше сделок → меньше комиссий.
- ⚠️ Может изменить профиль стратегии.

### 2. Торговать только в активные часы
- Лучше исполнение.
- Меньше slippage.

### 3. Использовать лимиты
- 0 комиссии за spread.
- ❌ Не исполнение.

### 4. Торговать ликвидные инструменты
- ES > GC > exotic.
- Меньше slippage.

### 5. Снизить turnover
- Пересмотр параметров.
- Может быть достаточно меньше сделок.

## Cost-aware оптимизация

### Целевая функция
```python
def objective(params):
    gross_metrics = backtest(params)
    cost_metrics = apply_costs(gross_metrics, params['frequency'])
    return cost_metrics['sharpe']
```

### Штраф за частоту
```python
def objective_penalized(params):
    sharpe = backtest(params)
    n_trades = len(params['trades'])
    if n_trades > MAX_TRADES:
        sharpe -= PENALTY
    return sharpe
```

## Impact на разные классы стратегий

### Trend following
- Низкая частота.
- ✅ Издержки не критичны.

### Mean reversion
- Высокая частота.
- ⚠️ Издержки критичны.

### Arbitrage
- Очень высокая частота.
- ❌ Издержки могут всё уничтожить.

### HFT
- Колотизация, скорость.
- ❌ Каждый микросекунд = деньги.

## Сравнение gross vs. net

### Графики
- Equity gross.
- Equity net (после издержек).
- Если сильно расходятся — пересмотр стратегии.

```python
import matplotlib.pyplot as plt
plt.plot(equity_gross, label='Gross')
plt.plot(equity_net, label='Net')
plt.legend()
```

## Sensitivity к издержкам

### Анализ
- Пересчитать Sharpe при cost × 0.5, 1, 2, 5.
- Если стратегия прибыльна при cost × 5 — robust.

```python
for cost_mult in [0.5, 1, 2, 5]:
    cost = base_cost * cost_mult
    sharpe = backtest_with_cost(cost)
    print(f'Cost × {cost_mult}: Sharpe {sharpe:.2f}')
```

## Когда стратегия убыточна

### Симптомы
- Gross Sharpe > 1, но net < 0.
- ❌ Издержки убивают edge.
- Решение: торговать реже / лимиты / ликвидные инструменты.

## Чек-лист урока
- [ ] Net edge > 0 с учётом всех издержек.
- [ ] Cost × 5 sensitivity: стратегия прибыльна.
- [ ] Commission % < 20%.
- [ ] Slippage учтён через модель.
- [ ] Cost-aware оптимизация применена.