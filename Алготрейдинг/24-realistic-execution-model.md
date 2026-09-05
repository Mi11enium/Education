# Урок 24. Реалистичная модель исполнения

## Зачем
Большинство бэктестов используют «идеальное» исполнение: ровно по цене, без задержек, с полным объёмом. В реале — всё хуже. Завышает PnL в 2–5 раз.

## Компоненты реалистичной модели

### 1. Сигнал → решение → ордер → исполнение

```
Signal (t=0) → Decision (t=0+δ₁) → Order sent (t=0+δ₂) → Fill (t=0+δ₃)
```

Задержки (latency):
- δ₁: время на принятие решения.
- δ₂: время на отправку ордера.
- δ₃: время на исполнение.

Итого: **total latency**.

### Типичные latency
- HFT: <1 мс.
- Скальпинг: 1–10 мс.
- Интрадей: 10–100 мс.
- Свинг: секунды.

## Модели исполнения

### 1. Market Order Fill
```python
def fill_market(signal_price, side, slippage, bid, ask):
    if side == 'buy':
        fill = ask + slippage
    else:
        fill = bid - slippage
    return fill
```

### 2. Limit Order Fill

```python
def fill_limit(order_price, side, slippage, bar_data):
    if side == 'buy':
        # Исполнение, если low <= order_price
        if bar_data['low'] <= order_price:
            return min(order_price, bar_data['open'])
    else:
        # Исполнение, если high >= order_price
        if bar_data['high'] >= order_price:
            return max(order_price, bar_data['open'])
    return None  # не исполнился
```

### 3. Stop Order Fill

```python
def fill_stop(stop_price, side, bar_data, gap_allowance):
    if side == 'long_stop':
        # Sell if low <= stop_price
        if bar_data['low'] <= stop_price:
            # Может быть slippage
            fill = min(stop_price - slippage, bar_data['open'])
        elif bar_data['open'] < stop_price - gap_allowance:
            # Gap
            fill = bar_data['open']
        else:
            fill = None
    return fill
```

## Модели для тестов

### 1. Constant Slippage
- `slippage = 0.5 × spread` для всех сделок.
- ❌ Слишком оптимистично.

### 2. Volume-Impact
- Чем больше ордер / средний объём, тем хуже.

```python
def slippage_volume(order_size, avg_volume, base_slip=0.5):
    participation = order_size / avg_volume
    return base_slip * (1 + 5 * participation**0.5)
```

### 3. Volatility-Adaptive
- В периоды высокой волатильности — больше slippage.

```python
def slippage_volatility(realized_vol, base_vol, base_slip):
    return base_slip * (realized_vol / base_vol)
```

### 4. Queue Position (для limit orders)
- Чем позже вы в очереди, тем меньше шанс на исполнение.

## Частичное исполнение

```python
def partial_fill(order_size, available_volume):
    if order_size <= available_volume:
        return order_size, 0
    return available_volume, order_size - available_volume
```

## Неисполнение (Missed trades)

```python
def fill_probability(volume, time_window, urgency):
    # Модель зависит от стратегии
    base_prob = 0.7 if urgency == 'high' else 0.4
    vol_factor = min(1, volume / target_size)
    return base_prob * vol_factor
```

## Когда выставляется ордер

### По close бара
```python
# Сигнал рассчитан на close бара N
signal_at_close = ...
# Отправка ордера
order_time = bar_N_close_time
# Fill на открытии бара N+1
fill_price = data['open'].iloc[N+1]
```

### По open бара
```python
# Решение на открытии бара N
signal = check_conditions(data.iloc[N-1])
# Отправка ордера в open_N
order_time = bar_N_open_time
# Fill зависит от условий
```

## Реалистичные допущения

### Округление
- `round(price, tick_size)`.
- Тик ES = 0.25.

### Лимиты
- Position size не больше X% объёма.
- Не больше max_position.

### Time-of-day
- Меньше ликвидности на open/close.
- Holiday effects.

## Пример реалистичного бэктеста

```python
def realistic_backtest(data, strategy, params):
    initial_capital = 100_000
    capital = initial_capital
    position = 0
    trades = []
    
    for i in range(1, len(data)):
        bar = data.iloc[i]
        prev_bar = data.iloc[i-1]
        
        # Сигнал на закрытии предыдущего бара
        signal = strategy.generate_signal(data.iloc[:i], params)
        
        if signal == 'buy' and position <= 0:
            # Fill на открытии текущего бара
            fill_price = bar['open'] + slippage
            capital -= fill_price * contract_size + commission
            position = 1
            entry_price = fill_price
            
        elif signal == 'sell' and position > 0:
            # Fill на открытии
            fill_price = bar['open'] - slippage
            capital += fill_price * contract_size - commission
            position = 0
            pnl = (fill_price - entry_price) * contract_size - 2*commission
            trades.append(pnl)
    
    return trades
```

## Чек-лист урока
- [ ] Slippage зависит от объёма и волатильности.
- [ ] Limit orders имеют вероятность исполнения.
- [ ] Stop orders учитывают gap-риск.
- [ ] Partial fill моделируется.
- [ ] Задержки учтены.
- [ ] Fill = next open (не close).