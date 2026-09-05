# Урок 44. Stop-Loss и Take-Profit

## Зачем
Без SL/TP — неограниченный риск. **Stop-loss обязателен** для каждой сделки.

## Типы SL/TP

### 1. Фиксированный
- SL = Entry - X% (или X тиков).
- TP = Entry + Y × X (R:R).

### 2. ATR-based
- SL = Entry - k × ATR.
- TP = Entry + m × ATR.

### 3. Support / Resistance
- SL за уровнем.
- TP на уровне.

### 4. Volatility
- Bollinger Bands.
- Keltner Channel.

### 5. Time-based
- Закрыть через N баров.

### 6. Trailing
- SL двигается за ценой.

### 7. Chandelier
- SL = Highest High - k × ATR.
- Trailing на основе ATR.

## Реализация Stop-Loss

### Hard stop (на бирже)
- Stop-market / stop-limit ордер.
- ✅ Исполняется автоматически.
- ❌ Может быть slippage.

```python
order = {
    'type': 'stop_market',
    'side': 'sell',
    'quantity': 1,
    'stop_price': 99.00,
    'tag': 'STOP_LOSS'
}
```

### Soft stop (в коде)
- Проверяем цену каждый тик.
- ✅ Можем учесть microstructure.
- ❌ Не сработает при обрыве связи.

```python
def check_stop(price, position, params):
    if position['side'] == 'long' and price <= position['stop']:
        close_position()
```

## Trailing Stop

### Простой
```python
trailing_stop = max(trailing_stop, current_price - atr * k)
```

### Chandelier
```python
highest_high = max(high, highest_high)
chandelier_stop = highest_high - 3 * atr
```

### Percentage trailing
```python
trailing_stop = current_price * (1 - 0.02)  # 2% trailing
```

## Stop-Loss и gap risk

### Проблема
- Цена «перепрыгнула» стоп.
- Stop-market исполнился по -7% вместо -3%.
- ⚠️ Loss > ожидаемый.

### Защита
- Stop-limit с запасом.
- Volatility-based SL.
- Hedge (опционы).

## ATR-based SL

```python
def atr_stop(entry, atr, k=2, side='long'):
    if side == 'long':
        return entry - k * atr
    else:
        return entry + k * atr
```

### Типичные значения k
- 1.0: tight.
- 2.0: medium.
- 3.0: wide.

## Подбор k

### Метод
- Walk-forward optimization.
- K=1, 1.5, 2, 2.5, 3.
- Оптимизировать Sharpe.

### Robust
- Не берите оптимальный k из train.
- Берите «середину плато».

## R-Multiple

### Определение
```
R = PnL / Risk
```

- Risk = (Entry - Stop) × Size.
- R=1: прибыль = риск.
- R=2: двойной риск.
- R=-1: убыток = риск (stop).

### Использование
- Стандартизация сделок.
- Сравнение стратегий.

## Take-Profit

### Фиксированный
- TP = Entry + X × (Entry - SL).
- R:R = X.

### Зависимость от R:R

#### R:R = 1:1
- BE WR = 50%.
- ✅ Для mean reversion.

#### R:R = 1:2
- BE WR = 33%.
- ✅ Для trend following.

#### R:R = 1:3
- BE WR = 25%.
- ✅ Для swing.

### Trailing TP
- Не ограничивать прибыль.
- Закрыть по trailing stop.

## Симметричные vs. Asymmetric

### Symmetric
- SL = TP = X%.
- ❌ Не учитывает edge.

### Asymmetric
- SL ≠ TP.
- R:R > 1 для трендследящих.
- R:R < 1 для mean reversion.

## Альтернативы SL/TP

### 1. Time-based exit
- Закрыть через N баров.
- Если стратегия не сработала.

### 2. Signal-based exit
- Противоположный сигнал.
- ✅ Логично.

### 3. Volatility-based exit
- Закрыть, если vol > X.

### 4. Profit target
- Фиксированный target %.

## Защита от emotional override

### Нельзя
- ❌ Перемещать SL дальше «в надежде».
- ❌ Убирать SL «на одну минуту».
- ✅ Все правила — в коде.

## Запись в бэктесте

```python
def backtest_with_sl_tp(data, strategy, sl_atr_mult=2, tp_atr_mult=4):
    trades = []
    position = None
    
    for i in range(len(data)):
        bar = data.iloc[i]
        
        if position is None:
            signal = strategy.get_signal(data.iloc[:i])
            if signal != 0:
                atr = data['atr'].iloc[i-1]
                entry = bar['open']
                sl = entry - sl_atr_mult * atr if signal > 0 else entry + sl_atr_mult * atr
                tp = entry + tp_atr_mult * atr if signal > 0 else entry - tp_atr_mult * atr
                position = {'side': signal, 'entry': entry, 'sl': sl, 'tp': tp}
        else:
            # Check SL
            if (position['side'] > 0 and bar['low'] <= position['sl']) or \
               (position['side'] < 0 and bar['high'] >= position['sl']):
                pnl = (position['sl'] - position['entry']) * position['side']
                trades.append(pnl)
                position = None
            
            # Check TP
            elif ...
```

## Чек-лист урока
- [ ] SL обязателен.
- [ ] ATR-based SL.
- [ ] R:R выбран осознанно.
- [ ] Trailing SL для тренда.
- [ ] Защита от emotional override.