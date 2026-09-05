# Урок 32. Реалистичная модель проскальзывания

## Виды slippage

### 1. Bid-Ask Slippage
- Market buy: платите ask.
- Market sell: получаете bid.
- = 0.5 × spread в среднем.

### 2. Delay Slippage
- Задержка между сигналом и исполнением.
- Цена ушла.

### 3. Market Impact
- Большой ордер двигает цену.

### 4. Volatility Slippage
- В периоды высокой vol — больше.

### 5. Gap Slippage
- Стоп-лосс «проскочили».

## Моделирование

### Модель 1: Constant slippage
```python
slippage = fixed_ticks  # e.g., 1 tick
```

- ❌ Не реалистично.

### Модель 2: Volatility-adaptive
```python
def slippage_vol(returns, base_slip=0.5, vol_lookback=20):
    realized_vol = returns.rolling(vol_lookback).std()
    return base_slip * (realized_vol / returns.std())
```

- ✅ Лучше.

### Модель 3: Volume impact
```python
def slippage_volume(order_size, avg_volume, base=0.5):
    participation = order_size / avg_volume
    if participation < 0.01:
        return base
    return base * (1 + 10 * participation**0.5)
```

- ✅ Для крупных ордеров.

### Модель 4: Square-root impact (Almgren)
```python
def almgren_slippage(quantity, daily_volume, sigma, eta=0.01):
    # Permanent impact
    permanent = eta * quantity
    # Temporary impact
    temporary = sigma * np.sqrt(quantity / daily_volume)
    return permanent + temporary
```

## Реалистичные значения

### Ликвидный рынок (ES, в активную сессию)
- Market order: 0.5–1 tick.
- Limit order (hit): 0–0.5 tick.
- Stop market: 1–2 ticks.

### Тонкий рынок (small-cap акции, exotic futures)
- Market order: 2–10 ticks.
- Stop market: 5–50 ticks.

### В кризис
- ES: 5–20 ticks.
- Exotic: 50+ ticks.

## Slippage в зависимости от размера

### Square-root model
```
Slippage = σ × √(Q / V)
```

где:
- Q — размер ордера.
- V — объём.
- σ — волатильность.

### Пример
- σ = 0.5% (дневная).
- Q = 100 контрактов.
- V = 10 000 контрактов (среднедневной).
- Slippage = 0.005 × √(100/10000) = 0.005 × 0.1 = 0.05%.

## Stress-slippage

### В кризис
- Bid-ask spread × 3–10.
- Volatility × 2–5.
- Slippage × 5–20.

### Учёт в стратегии
- Заложить в бэктест 2×–3× обычного.
- Если стратегия прибыльна с этим — robust.

## Эмпирический подход

### Из реальной торговли
```python
# Из своей истории:
slippage_actual = (filled_price - expected_price) * sign(side)
slippage_mean = slippage_actual.mean()
slippage_std = slippage_actual.std()
```

### Исторический slippage (тиков)
- ES: ~0.5 tick в среднем, 2 tick в кризис.
- CL: ~1 tick.
- GC: ~0.5 tick.

## Slippage в бэктесте

### Добавление
```python
def realistic_fill(bar, signal, side, params):
    if side == 'buy':
        if signal == 'market':
            fill = bar['open'] + params['slippage']
        elif signal == 'limit':
            if bar['low'] <= params['limit_price']:
                fill = min(params['limit_price'], bar['open'])
            else:
                fill = None
    elif side == 'sell':
        if signal == 'market':
            fill = bar['open'] - params['slippage']
        elif signal == 'limit':
            if bar['high'] >= params['limit_price']:
                fill = max(params['limit_price'], bar['open'])
            else:
                fill = None
    return fill
```

## Мониторинг в продакшне

### Метрики
- Avg slippage per trade.
- Slippage по типу ордера.
- Slippage по часу / дню.
- Slippage в кризис.

### Alerts
- Slippage > X% → проблема.
- Резкий рост slippage → ревизия execution.

## Как уменьшить

### 1. Лимитные ордера
- Не платите за spread.
- ❌ Не исполнение.

### 2. Iceberg
- Скрытие объёма.
- Только для крупных ордеров.

### 3. TWAP / VWAP
- Нарезка во времени.
- Уменьшает market impact.

### 4. SOR (Smart Order Routing)
- Лучшие цены между venues.

### 5. Торговля в активные часы
- Лучшая ликвидность.

### 6. Pre-funded liquidity
- Заранее знать, где bid/ask.

## Чек-лист урока
- [ ] Slippage моделируется в бэктесте.
- [ ] Stress-slippage заложен.
- [ ] Мониторинг реального slippage настроен.
- [ ] Используете SOR / iceberg / TWAP.
- [ ] Slippage < 10% от gross PnL.