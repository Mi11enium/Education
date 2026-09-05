# Урок 16. Exposure, Leverage, Margin

## Основные понятия

### Notional Exposure
Полная номинальная стоимость позиции.
- 1 контракт ES (S&P 500 futures): notional = $50 × price.
- 10 контрактов: 10× экспозиция.

### Equity
Собственный капитал на счёте.

### Margin
- **Initial margin**: требуется для открытия позиции.
- **Maintenance margin**: минимум для удержания.
- **Variation margin**: ежедневный PnL (mark-to-market).

### Leverage
```
Leverage = Notional Exposure / Equity
```

- Leverage 2x: $200k exposure при $100k equity.

## Типы exposure

### 1. Gross Exposure
```
Gross = sum(|long|) + sum(|short|)
```

### 2. Net Exposure
```
Net = sum(long) + sum(short)  # long +, short -
```

### 3. Beta-Adjusted Exposure
Учитывает β каждой позиции.

### Пример
- Long 5 ES, Short 2 ES.
- Gross = 5 + 2 = 7.
- Net = 3.
- Если $500k на счёту, 1 ES = $250k notional: gross = 7 × 250k = $1.75M, leverage = 3.5x.

## Риск leverage

### Линейный риск
При leverage 2x движение актива на 1% даёт ±2% PnL.

### Маржин-колл
Equity падает ниже maintenance margin → брокер требует довнести.
- Не довнесли → принудительное закрытие.

### Margin Call в стресс
- 2020 COVID: многие получили margin call.
- 2008: leverage убил стратегии.

## Futures Margin

### Пример (CME E-mini S&P 500)
- Notional: $50 × price.
- Initial margin: ~$12 000 (2024).
- Maintenance: ~$11 000.

### Расчёт leverage
- Цена 5000 → notional = $250 000.
- Initial margin $12 000.
- Effective leverage = 250/12 = **20.8x**.

⚠️ Фьючерсы — **уже с плечом**. Наличный счёт — без.

## Оптимальное плечо

### Зависит от
- Волатильности стратегии.
- MaxDD tolerance.
- Risk per trade.

### Простая формула
```
Max_Leverage = Risk_per_Trade / Position_Stop_Distance
```

### Пример
- Счёт $100k.
- Risk per trade = 1% = $1000.
- Stop = 20 пунктов.
- 1 пункт = $50.
- Max position = $1000 / (20×$50) = 1 контракт.
- Notional = 1 × $250k = leverage 2.5x.

## Exposure по стратегиям

### Tendency
- Трендследящие: 100% net long/short (всегда в рынке).
- Mean reversion: ±20-50% net.
- Pair trading: ~0% net (market neutral).
- HFT: очень высокий gross, ~0 net.

## Какие лимиты ставить

### Hard limits (нельзя превышать)
- Max gross exposure.
- Max leverage.
- Max position per instrument.
- Max sector exposure.

### Soft limits (warning)
- Target gross/net.
- Average leverage за период.

## Risk-based limits

### VaR-based
- Max 1-day VaR 1% от equity.
- Max 1-week VaR 5%.

### Volatility targeting
- Целевая волатильность (например, 15% годовых).
- Position = (target_σ / realized_σ) × base_size.

```python
def vol_target_size(target_vol, realized_vol, base_size):
    return (target_vol / realized_vol) * base_size
```

## Margin Call и формулы

### Ликвидация при Long
- Position: long 1 ES @ 5000 (notional $250k, margin $12k).
- Equity = $12k + PnL.
- Если ES упал на 240 пунктов ($12k loss) → equity → 0.
- Maintenance: $11k.
- ⚠️ Если equity < $11k → margin call.
- Должны довнести до $12k (initial).

### Правило 2% / 6%
- Risk per trade ≤ 2% equity.
- Total open risk ≤ 6% equity.

## Чек-лист урока
- [ ] Различаете gross и net exposure.
- [ ] Понимаете effective leverage фьючерсов.
- [ ] Считаете leverage через margin.
- [ ] Лимиты на exposure/leverage заданы.
- [ ] Risk per trade ≤ 2% equity.