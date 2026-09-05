# Урок 20. Проскальзывание (Slippage)

## Что это
**Slippage** — разница между ожидаемой ценой исполнения и фактической.
- Может быть **положительным** (лучше ожидания, бонус).
- Обычно **отрицательное** (хуже, издержка).

## Виды slippage

### 1. Market Impact Slippage
- Большой ордер двигает цену.
- 1000 лотов ES не закроются по одной цене.

### 2. Bid-Ask Slippage
- Покупка по ask, продажа по bid.
- Спред = минимальное slippage.

### 3. Delay Slippage
- Между сигналом и исполнением проходит время.
- Цена успела уйти.

### 4. Volatility Slippage
- В периоды высокой волатильности стоп-лоссы хуже.

### 5. Gap Slippage
- Цена «перепрыгнула» уровень (open > stop-loss).

## Формула
```
Slippage = Actual_Fill_Price - Expected_Fill_Price
```

Для лонга:
- Expected: 100.
- Actual: 100.5.
- Slippage: +0.5 (негативно).

## Оценка slippage

### Эмпирические правила

| Тип стратегии | Типичный slippage (тиков) |
|---|---|
| Лимитный ордер на ликвидном рынке | 0 |
| Marketable limit | 0–1 |
| Market order (ликвидный) | 1–2 |
| Market order (тонкий рынок) | 2–10 |
| Стоп-лосс (нормальный) | 0–2 |
| Стоп-лосс (волатильный) | 5–20 |
| Стоп-лосс (gap) | 20+ |

### По bid-ask спреду
- Slippage ≈ 0.5 × spread (для market orders).
- Slippage ≈ 0.1 × spread (для marketable limits).

## Моделирование в бэктесте

### Простая модель
```python
def apply_slippage(price, side, slippage_ticks, tick_size):
    if side == 'buy':
        return price + slippage_ticks * tick_size
    else:
        return price - slippage_ticks * tick_size
```

### Реалистичная модель
```python
def realistic_slippage(volume, avg_volume, base_slippage=0.5):
    # Чем больше ордер относительно среднего объёма, тем хуже
    participation = volume / avg_volume
    if participation < 0.01:
        return base_slippage
    elif participation < 0.1:
        return base_slippage * (1 + 2 * participation)
    else:
        return base_slippage * (1 + 5 * participation)
```

### Квадратичная модель (Almgren-Chriss)
```
Slippage = σ × sqrt(T) × f(participation_rate)
```

## Спреды на разных рынках

| Рынок | Spread (тиков) | Стоимость тика |
|---|---|---|
| ES | 0.25 (1 tick) | $12.50 |
| NQ | 0.25 (1 tick) | $5.00 |
| CL (нефть) | 0.01 (1 tick) | $10.00 |
| GC (золото) | 0.10 (1 tick) | $10.00 |
| 6E (EUR/USD) | 0.0001 (1 tick) | $12.50 |
| BTC futures | $0.50–$5.00 | varies |

## Учёт slippage в стратегии

### 1. По реальным данным
- Возьмите свою историю исполнения.
- Средний slippage по типам ордеров.

### 2. Консервативная оценка
- Заложите 2–5× обычного slippage.
- Если стратегия прибыльна с этим — надёжно.

### 3. Stress-test
- При crisis volatility slippage × 5–10.

## Hidden slippage

### Что не учитывается в бэктесте
- Частичное исполнение.
- Отменённые ордера.
- Пропущенные сигналы.

### Как учесть
- Занижайте fill rate на 5–10%.
- В кризис — на 30–50%.

## Способы уменьшить slippage

### 1. Лимитные ордера
- ✅ Не платите за spread.
- ❌ Риск неисполнения.

### 2. Iceberg ордера
- Скрытие размера ордера.
- Для крупных позиций.

### 3. TWAP / VWAP
- Нарезка крупного ордера во времени.
- Уменьшает market impact.

### 4. Торговля в активные часы
- Лучшая ликвидность.
- Меньше slippage.

### 5. Выбор брокера
- Smart Order Routing (SOR).
- Direct market access.

## Stop-loss vs. Stop-market

### Stop-loss → stop-market
- Превращается в рыночный при триггере.
- ⚠️ Slippage.

### Stop-loss → stop-limit
- Превращается в лимитный.
- ❌ Может не исполниться.

### Компромисс
- Stop-market с запасом.
- Например, stop = 99.0, исполнить до 98.5.

## Пример: влияние slippage на стратегию

### Стратегия
- 50 сделок/мес.
- Avg trade PnL (gross): $100.
- Slippage: 2 ticks × $12.50 (ES) = $25/сделка.
- Total slippage: 50 × $25 = $1250/мес.

### Влияние
- 100 сделок × $100 = $10 000 gross.
- Slippage = 50 × $25 = $1250.
- Net = $8750.
- Slippage_pct = 12.5%.

⚠️ При высокочастотной стратегии slippage **доминирует** над комиссией.

## Чек-лист урока
- [ ] Оцениваете slippage эмпирически по своей истории.
- [ ] Бэктест включает реалистичный slippage.
- [ ] Stress-test: slippage × 5–10.
- [ ] Stop-loss проверяются на gap-риск.
- [ ] Используете лимиты/iceberg/TWAP где нужно.