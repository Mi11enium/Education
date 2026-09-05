# Урок 46. Корреляция и диверсификация

## Зачем
Диверсификация снижает риск портфеля без потери доходности. Без неё — весь портфель в одной стратегии/инструменте.

## Корреляция

### Определение
```
ρ(x, y) = cov(x, y) / (σ_x × σ_y)
```

- ρ = 1: движутся вместе.
- ρ = -1: противоположно.
- ρ = 0: независимо.

### Особенности финансовых активов
- Корреляция **нестационарна**.
- В кризис → ρ → 1 (всё падает).

## Risk Parity

### Идея
Каждая позиция вносит одинаковый риск.

```
w_i × σ_i = const
w_i = (1/σ_i) / Σ(1/σ_j)
```

### Пример
- A: σ = 10%, B: σ = 20%.
- w_A = (1/10) / (1/10 + 1/20) = 0.667.
- w_B = 0.333.

### На фьючерсах
- ES: σ = 15%, NQ: σ = 20%, CL: σ = 30%.
- Позиции: ES = 1, NQ = 0.75, CL = 0.5.

## Диверсификация по стратегиям

### Пример портфеля
- Trend (TF): σ = 12%, E[R] = 8%.
- Mean Reversion (MR): σ = 8%, E[R] = 5%.
- Carry: σ = 6%, E[R] = 3%.

### С корреляциями
- ρ(TF, MR) = -0.3.
- ρ(TF, Carry) = 0.1.
- ρ(MR, Carry) = 0.2.

### Оптимальные веса
- Risk Parity или Max Utility.

## Correlation breakdown

### Проблема
- Корреляции «взрываются» в кризис.
- Диверсификация не спасает, когда нужнее всего.

### Решение
- Stress-test на корреляцию = 1.

## Position-level diversification

### По типу
- Directional (long/short bias).
- Spread (futures calendar).
- Volatility.

### По инструменту
- Акции, облигации, commodities, FX.

### По таймфрейму
- 5min, 1h, 1d.

## Risk Budgeting

### Концепция
- Выделить «бюджет риска» на каждую позицию.
- Vol-targeting × Inverse volatility.

```python
def risk_parity_weights(vols, max_weight=0.4):
    inv_vol = 1 / np.array(vols)
    weights = inv_vol / inv_vol.sum()
    # Cap
    weights = np.minimum(weights, max_weight)
    weights = weights / weights.sum()
    return weights
```

## Marginal VaR

### Вклад каждой позиции в общий VaR
```
MVar_i = ∂VaR / ∂w_i
```

- Если MVar_i > 0 → позиция увеличивает риск.
- Перебалансировать.

## Cluster risk

### Если стратегии группируются
- Например, все трендследящие.
- ⚠️ Фактически одна стратегия.

### Решение
- Добавить uncorrelated стратегию (e.g., market-neutral).

## Динамическая корреляция

### Rolling correlation
```python
rolling_corr = data['a'].rolling(60).corr(data['b'])
```

### Regime-based
- Bull: ρ_low.
- Bear: ρ_high.

### Учёт
- Периодически пересчитывать risk parity.

## Cross-asset

### Примеры
- Long stocks + long bonds (обычно отрицательная корреляция).
- Gold + stocks (низкая корреляция).
- USD + commodities (отрицательная).

## Практическая реализация

```python
# Portfolio volatility
def portfolio_vol(weights, cov_matrix):
    return np.sqrt(weights @ cov_matrix @ weights.T)

# Risk Parity
def risk_parity(vols, cov, target_vol=0.10):
    n = len(vols)
    weights = np.ones(n) / n
    for _ in range(100):
        port_vol = portfolio_vol(weights, cov)
        marginal_risk = cov @ weights
        for i in range(n):
            weights[i] = target_vol * weights[i] / (n * marginal_risk[i])
        weights /= weights.sum()
    return weights
```

## Чек-лист урока
- [ ] Корреляции посчитаны.
- [ ] Risk Parity или weighted.
- [ ] Stress test на ρ = 1.
- [ ] Diversification по типу, инструменту, таймфрейму.
- [ ] Динамическая перебалансировка.