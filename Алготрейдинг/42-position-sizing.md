# Урок 42. Position Sizing

## Зачем
Размер позиции — главный фактор выживания. Плохой sizing убивает даже хорошие стратегии.

## Методы sizing

### 1. Фиксированный размер (Fixed)
- N контрактов на каждую сделку.
- ✅ Просто.
- ❌ Не учитывает волатильность, размер счёта.

```python
size = 1  # всегда 1 контракт
```

### 2. Фиксированная доля (Fractional)
- X% от equity в позиции.

```python
size = int(equity * 0.1 / contract_price)  # 10% в позиции
```

### 3. Фиксированный риск (Fixed Fractional / Risk Parity)
- Risk = X% от equity.
- Size = Risk / Stop_Distance

```python
risk_amount = equity * 0.01  # 1% риск
size = risk_amount / (stop_distance * tick_value / tick_size)
```

### 4. Volatility-based
- Size = (Target_Vol / Realized_Vol) × Base_Size

```python
target_vol = 0.10  # 10% годовых
realized_vol = returns.rolling(20).std() * np.sqrt(252)
size = (target_vol / realized_vol) * base_size
```

- ✅ Адаптивно.
- Уменьшает в кризис.

### 5. Kelly Criterion
- Оптимальный f для max log-growth.

```python
def kelly_fraction(win_rate, win_loss_ratio):
    return win_rate - (1 - win_rate) / win_loss_ratio
```

- ⚠️ Агрессивно. Используйте 0.25–0.5 × Kelly.

### 6. Anti-Kelly (уменьшенный)
- 0.1–0.3 × Kelly.
- Безопаснее.

### 7. Optimal f (Vince)
- f* = max f, при котором не было бы margin call.

```python
def optimal_f(trades):
    # Найти f, max по geometric mean
    ...
```

### 8. Volatility-Adjusted
- Position size обратно пропорционален realized volatility.

## Сравнение

| Метод | Сложность | Агрессивность | Адаптивность |
|---|---|---|---|
| Fixed | Низкая | Низкая | Нет |
| Fractional | Низкая | Низкая | Частично |
| Fixed Risk | Средняя | Средняя | Да |
| Vol-targeting | Средняя | Средняя | Да |
| Kelly | Средняя | Высокая | Нет |
| 0.25 Kelly | Средняя | Средняя | Нет |

## Реализация на фьючерсах

### Параметры
- Equity: $100 000.
- Risk per trade: 1% = $1000.
- ATR(20) = 50 пунктов.
- Stop = 2 × ATR = 100 пунктов.
- Tick value: $12.50 (ES).
- Tick size: 0.25.

### Расчёт
```python
ticks_at_risk = 100 / 0.25 = 400
risk_per_contract = 400 * 12.50 = $5000
size = 1000 / 5000 = 0.2 → 0 контрактов!
```

⚠️ Слишком высокий stop → 0 контрактов. Уменьшите risk или stop.

### Альтернатива
- Risk = 0.5% ($500).
- Stop = 1 × ATR = 50 пунктов.
- Ticks at risk: 50 / 0.25 = 200.
- Risk per contract: 200 × 12.50 = $2500.
- Size = 500 / 2500 = 0.2 → 0.

⚠️ Stop всё ещё велик для $100k. Нужен меньший таймфрейм.

## Volatility targeting

```python
def vol_target_size(target_vol, realized_vol, equity, contract_notional):
    position_notional = (target_vol / realized_vol) * equity
    contracts = position_notional / contract_notional
    return contracts
```

### Пример
- Target vol: 15% годовых.
- Realized vol (20d): 25%.
- Equity: $100k.
- Contract notional: $250k.
- Position = 15/25 × 100k = $60k.
- Contracts = 60k / 250k = 0.24 → 0.

⚠️ Нужен больший счёт или меньше vol target.

## Anti-martingale vs. Martingale

### Martingale (❌)
- Увеличиваем после убытка.
- ❌ Слив в длинной серии убытков.

### Anti-martingale (✅)
- Увеличиваем после прибыли.
- ✅ Используем «горячие» серии.

### Реализация
```python
def anti_martingale_size(base_size, recent_pnl):
    if recent_pnl > 0:
        return base_size * 1.5  # увеличить
    else:
        return base_size * 0.5  # уменьшить
```

## Drawdown-based sizing

### Уменьшение после убытков
```python
def dd_based_size(base_size, current_dd, max_dd=0.25):
    if current_dd < -0.10:
        return base_size * 0.5
    elif current_dd < -0.20:
        return base_size * 0.25
    elif current_dd < -max_dd:
        return 0  # stop
    else:
        return base_size
```

## Корреляция между стратегиями

### Если стратегии коррелированы
- Не суммируйте размеры.
- Используйте marginal risk.

```python
def portfolio_var(positions, correlations, vols):
    # Portfolio VaR
    weights = np.array([p['size'] * p['vol'] for p in positions])
    cov = np.outer(weights, weights) * correlations
    var = np.sqrt(cov.sum())
    return var
```

## Чек-лист урока
- [ ] Risk per trade ≤ 2%.
- [ ] Volatility-based sizing.
- [ ] Drawdown-based sizing.
- [ ] Корреляция учтена.
- [ ] Anti-martingale / Kelly.