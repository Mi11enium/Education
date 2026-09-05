# Урок 41. Риск-менеджмент: основы

## Зачем
**Risk management** — единственное, что отделяет трейдера от разорения. Без него даже прибыльная стратегия убьёт.

## Главные правила

### 1. Правило 2% / 6%
- **Risk per trade** ≤ 2% от equity.
- **Total open risk** ≤ 6% от equity.

### 2. Правило MaxDD
- Max допустимая просадка: 20–30%.
- При превышении → остановка стратегии.

### 3. Diversification
- Не класть все яйца в одну корзину.
- Разные стратегии / инструменты / таймфреймы.

## Компоненты риск-менеджмента

### 1. Pre-trade risk
- Проверка перед открытием позиции.
- Не превышены ли лимиты.

### 2. Position sizing
- Сколько контрактов/акций.

### 3. Stop-loss
- Обязателен для каждой сделки.

### 4. Take-profit
- Необязателен, но желателен.

### 5. Risk monitoring
- Реальное время.
- Alerts.

## Pre-trade risk checks

```python
def pre_trade_check(signal, portfolio):
    # Position size
    if signal['size'] > portfolio['max_position']:
        return False, 'Position too large'
    
    # Total exposure
    new_exposure = portfolio['current_exposure'] + signal['exposure']
    if new_exposure > portfolio['max_exposure']:
        return False, 'Total exposure too high'
    
    # Margin
    new_margin = portfolio['current_margin'] + signal['margin_required']
    if new_margin > portfolio['max_margin']:
        return False, 'Margin insufficient'
    
    # Risk per trade
    if signal['risk'] > portfolio['equity'] * 0.02:
        return False, 'Risk per trade > 2%'
    
    # Open risk
    total_open_risk = portfolio['open_risk'] + signal['risk']
    if total_open_risk > portfolio['equity'] * 0.06:
        return False, 'Open risk > 6%'
    
    return True, 'OK'
```

## Position sizing

### Фиксированный размер
- N контрактов на сделку.
- ❌ Не учитывает волатильность.

### Фиксированный риск
- Risk = X% от equity.
- Size = Risk / (Stop_distance × Tick_value)

```python
def position_size(equity, risk_pct, stop_distance, tick_value, tick_size):
    risk_amount = equity * risk_pct
    ticks_at_risk = stop_distance / tick_size
    dollar_risk_per_contract = ticks_at_risk * tick_value
    contracts = risk_amount / dollar_risk_per_contract
    return int(contracts)
```

### Volatility-based
- Size = (Target_Vol / Realized_Vol) × Base_Size

## Stop-loss

### Фиксированный
- Stop на N% от входа.
- Просто, не адаптивно.

### ATR-based
- Stop = Entry - k × ATR.
- Адаптивно к волатильности.

### Trailing
- Stop двигается за ценой.
- Защита прибыли.

### Time-based
- Выход через N баров.
- Если стратегия не сработала.

## Take-profit

### Фиксированный
- TP на N% от входа.

### Risk/Reward
- TP = Entry + 2 × Stop_Distance (R:R = 1:2).

### Trailing TP
- TP двигается за ценой.

### Time-based
- Закрыть в конце дня / недели.

## Risk Monitoring

### В реальном времени
- Текущий PnL.
- Drawdown.
- Margin.
- Exposure.

### Alerts
- DD > X%.
- Daily PnL < -Y%.
- Margin > Z%.
- Latency > N ms.

## Kill Switch

### Автоматический
```python
def check_kill_switch(equity, initial_equity, max_dd=0.25):
    dd = (equity - initial_equity) / initial_equity
    if dd < -max_dd:
        return 'STOP_ALL'
    return 'CONTINUE'
```

### Ручной
- Кнопка в UI.
- Telegram-бот.

## Чек-лист урока
- [ ] Risk per trade ≤ 2%.
- [ ] Total open risk ≤ 6%.
- [ ] MaxDD лимит задан.
- [ ] Stop-loss на каждой сделке.
- [ ] Pre-trade check автоматизирован.
- [ ] Kill switch настроен.