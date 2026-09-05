# Урок 31. Реалистичная модель комиссий

## Уровни комиссий

### 1. Per Contract / Per Share
- Фиксированная плата за единицу.
- Пример: ES = $1.18 / сторона.

### 2. Tiered (ступенчатая)
- Зависит от объёма.
- 1–100 контрактов: $2.50/контракт.
- 100–500: $1.50.
- 500+: $1.00.

### 3. % от оборота
- 0.1% от notional.
- Для акций / крипты.

### 4. Subscription + Per-Trade
- Платите подписку, низкие per-trade.
- NinjaTrader: $60/мес + $0.50/trade.

## Из чего складывается комиссия

```
Total_Cost = Broker_Commission + Exchange_Fee + 
             Clearing_Fee + Regulatory_Fee + Data_Fee
```

### Биржевой сбор (CME)
- ES: $0.42 / сторона.
- NQ: $0.42 / сторона.

### Clearing fee
- Обычно входит в комиссию.

### Regulatory (SEC/FINRA)
- Для акций: SEC fee ≈ $8 / $1M.
- NFA для фьючерсов: $0.04 / сторона.

### Data
- Real-time данные: $50–500 / мес.
- Historical: разово или по подписке.

## Effective cost per trade

### Пример для ES
- Брокер: $1.18/сторона.
- Exchange: $0.42/сторона.
- Итого: $1.60/сторона.
- Round-trip: $3.20.
- Tick value: $12.50.
- Cost = 3.20 / 12.50 = 0.256 тика.

### Для NQ
- Брокер: $1.18/сторона.
- Exchange: $0.42/сторона.
- Round-trip: $3.20.
- Tick value: $5.00.
- Cost = 3.20 / 5.00 = 0.64 тика.

⚠️ **NQ дороже относительно тика**, чем ES.

## Break-even analysis

### Минимальный edge для покрытия комиссий
```
Min_Edge = 2 × Commission_per_side
```

### Пример
- ES: 2 × $1.60 = $3.20.
- Если avg_win = $50, то break-even WR для покрытия = 6.4% edge.

## Запись в бэктесте

### Простой
```python
pnl_net = pnl_gross - 2 * commission
```

### С учётом partial fills
```python
pnl_net = (fill_size * price_diff) - (fill_size * 2 * commission)
```

### С учётом tier
```python
def calculate_commission(contracts, tiers):
    remaining = contracts
    total = 0
    for tier_limit, rate in tiers:
        in_tier = min(remaining, tier_limit)
        total += in_tier * rate
        remaining -= in_tier
        if remaining <= 0:
            break
    return total
```

## Комиссия vs. прибыль

### Commission % от PnL
```
Comm_pct = Total_Commission / Total_PnL_net
```

### Здоровые значения
- <10% — отлично.
- 10–20% — нормально.
- >30% — много.

### Влияние на стратегию
- Скальпинг: комиссия = 30–50% от gross PnL.
- Свинг: комиссия = 2–5%.

## Скрытые комиссии

### 1. Валютная конвертация
- 0.1–0.5% за конвертацию.
- ⚠️ Если торгуете не в национальной валюте.

### 2. Withdraw fee
- За вывод средств.

### 3. Inactivity fee
- Если не торгуете.

### 4. Margin interest
- На занятую маржу.
- Фьючерсы: обычно 0% (маржа не кредитуется).

## Сравнение брокеров

### IBKR
- Tiered: $0.62–$1.40/контракт.
- Pro: лучше для больших объёмов.

### NinjaTrader Brokerage
- $0.59 + exchange fees.
- Подходит для активных.

### Tradovate
- $0.59/контракт + данные.
- Cloud.

### AMP / Optimus
- CQG: $0.59/контракт.
- Rithmic: $0.59/контракт.

## Оптимизация расходов

### 1. Выбрать правильного брокера
- Сравнить effective rate.
- Объёмные скидки.

### 2. Volume tiers
- Больше торгуете — меньше per-trade.

### 3. Frozen contracts
- Некоторые брокеры дают фиксированную цену за месяц.

### 4. Избегать переторговки
- Каждая сделка стоит.
- Подумайте, нужна ли она.

## Чек-лист урока
- [ ] Знаете exact round-trip commission.
- [ ] Бэктест учитывает комиссии.
- [ ] Commission % < 20%.
- [ ] Учтены скрытые комиссии.
- [ ] Сравнили брокеров по effective rate.