# Урок 47. Типы ордеров и исполнение

## Типы ордеров

### Market Order
- Исполнение по лучшей доступной цене.
- ✅ Гарантия исполнения.
- ❌ Slippage.

### Limit Order
- Исполнение по указанной цене или лучше.
- ✅ Контроль цены.
- ❌ Может не исполниться.

### Stop Order
- Активируется при достижении stop price.
- Stop-market → market при триггере.
- Stop-limit → limit при триггере.

### Stop-Limit
- Stop: триггер.
- Limit: цена limit-ордера.
- ⚠️ Gap risk.

### Marketable Limit
- Limit с ценой через спред.
- «Near-touch» order.

## Атрибуты ордера

### Time in Force
- **GTC (Good Till Cancel)**: пока не исполнен или отменён.
- **DAY**: до конца сессии.
- **IOC (Immediate or Cancel)**: исполнить сейчас или отменить.
- **FOK (Fill or Kill)**: всё сейчас или ничего.
- **GTD (Good Till Date)**: до даты.

### Flags
- Reduce-only: только закрыть.
- Post-only: только maker.
- Iceberg: скрыть размер.
- Hidden: невидимый в стакане.

## Алгоритмы исполнения

### TWAP (Time-Weighted Average Price)
- Нарезка ордера на равные части по времени.
- Для минимизации impact.

```python
def twap_exec(total_qty, duration_sec, slice_sec=60):
    n_slices = duration_sec // slice_sec
    slice_qty = total_qty / n_slices
    for _ in range(n_slices):
        send_order(slice_qty, type='limit')
        time.sleep(slice_sec)
```

### VWAP (Volume-Weighted Average Price)
- Нарезка пропорционально историческому объёму.

### POV (Percentage of Volume)
- Не больше X% текущего объёма.

### Implementation Shortfall
- Баланс impact vs. opportunity cost.

## Smart Order Routing (SOR)

### Идея
- Один тикер торгуется на нескольких площадках.
- SOR отправляет туда, где лучшая цена.

### Пример
- ES на CME.
- Микро-контракт MES на CME.
- Smart routing выберет лучшую цену.

## Iceberg orders

### Что это
- Показывать в стакане только малую часть.
- Реальный размер больше.

```python
order = {
    'total_qty': 1000,
    'display_qty': 10,
    'side': 'buy',
    'type': 'limit',
    'price': 99.50
}
```

- ✅ Скрывает намерения.
- ❌ Медленнее исполнение.

## Order Management

### Состояния
- New (создан).
- Submitted (отправлен на биржу).
- Partial (частично исполнен).
- Filled (полностью исполнен).
- Cancelled.
- Rejected.

### OMS (Order Management System)
- Хранит все ордера.
- Синхронизация с биржей.
- Recovery при сбое.

## Pre-trade checks

```python
def validate_order(order, account):
    # Notional
    notional = order['qty'] * order['price'] * contract_multiplier
    if notional > account['max_notional']:
        return False, 'Notional too large'
    
    # Margin
    margin = calc_margin(order, account)
    if margin > account['max_margin']:
        return False, 'Margin insufficient'
    
    # Daily loss limit
    if account['daily_pnl'] < -account['max_daily_loss']:
        return False, 'Daily loss limit hit'
    
    # Rate limits
    if account['n_orders_today'] > account['max_orders']:
        return False, 'Order rate limit'
    
    return True, 'OK'
```

## Latency

### Источники
- Network: 1–50 ms.
- Broker gateway: 1–10 ms.
- Exchange matching: 0.1–5 ms.
- Total: 5–100 ms.

### Уменьшение
- Colocation.
- Direct market access.
- Optimized code (C++).
- FPGA (HFT).

## Error handling

### Типы ошибок
- **Network error**: retry.
- **Invalid order**: log + alert.
- **Risk check fail**: log + skip.
- **Exchange reject**: log + investigate.
- **Fill error**: investigate (discrepancy).

```python
def send_order_safe(order):
    try:
        response = broker.send_order(order)
    except NetworkError:
        return retry_with_backoff(order)
    except InvalidOrderError as e:
        log_alert(f'Invalid order: {e}')
        return None
    except RiskCheckError as e:
        log_alert(f'Risk check fail: {e}')
        return None
    return response
```

## Reconciliation

### Что проверять
- Position по брокеру = position в нашей системе.
- Open orders = отправленные.
- PnL = ожидаемый.
- Cash balance.

### Периодичность
- Каждые 5 минут.
- Каждые 30 минут.
- При сомнениях.

## Чек-лист урока
- [ ] Типы ордеров понятны.
- [ ] TWAP/VWAP для крупных.
- [ ] Pre-trade checks.
- [ ] Latency минимизирована.
- [ ] Reconciliation настроена.