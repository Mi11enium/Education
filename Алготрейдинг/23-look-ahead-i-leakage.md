# Урок 23. Look-ahead bias и утечка данных

## Что это
**Look-ahead bias** — использование данных, которых **не было** в момент принятия решения. Завышает бэктест-результаты.

## Типичные источники

### 1. Индикаторы на текущем баре

#### ❌ Плохо
```python
# close текущего бара — будущего для решения
signal = (data['close'] > data['sma_20'])
```

В реальной торговле вы видите только close **закрытого** бара. Нельзя использовать close текущего.

#### ✅ Хорошо
```python
signal = (data['close'].shift(1) > data['sma_20'].shift(1))
```

### 2. Корпоративные события

#### ❌ Плохо
- Использовать adjusted close (с дивидендами) для решений в прошлом.
- Тогда как в реальном времени вы видели **unadjusted** цену.

#### ✅ Хорошо
- Либо использовать только unadjusted (и учитывать события отдельно).
- Либо применять adjustment **только** к PnL после события.

### 3. Survivorship bias
- Данные содержат только тикеры, которые существуют сейчас.
- Тикеры, которые **обанкротились** — удалены.
- Реальный инвестор не знал будущего.

#### Решение
- Использовать delisted tickers (CRSP, Bloomberg).

### 4. Index rebalancing
- Знать, какие тикеры войдут в индекс в будущем.
- ❌ Нельзя.

### 5. Earnings
- Знать, что earnings будут хорошими.
- ❌ Нельзя.

### 6. Spliced data
- Соединение tick данных с минутными → сдвиг.

### 7. Метки времени
- Last timestamp vs. bar open timestamp.
- На графике close, на самом деле trade был в 09:35:23.

## Как обнаружить look-ahead

### 1. Аномально высокие метрики
- Sharpe > 3–4 → почти всегда утечка.

### 2. Странные паттерны
- Стратегия «угадывает» гэпы.
- Закрытие точно в минимуме/максимуме.

### 3. Equity curve без просадок
- Слишком гладкая = подозрительно.

### 4. «Magic» parameters
- Параметр, идеально совпадающий с чем-то.

## Правила безопасности

### 1. Точка решения
Решение принимается **на закрытии бара N**.
Исполнение — на открытии бара N+1.

```python
signal = generate_signal(data.iloc[:n])  # только данные до n
entry_price = data['open'].iloc[n+1]    # fill на следующем open
```

### 2. Никогда не используйте `iloc[-1]` для решений
- `iloc[-1]` = текущий бар = look-ahead.

### 3. Сдвигайте всё
- `.shift(1)` на все признаки.

### 4. Pipeline
```python
# 1. Raw data
# 2. Shift (предотвращение look-ahead)
# 3. Indicators
# 4. Signals
# 5. Returns
# 6. PnL
```

## Другие виды leakage

### 1. Target leakage
- Признак, который сам по себе — прокси для target.
- Пример: «средняя цена за день» при решении внутри дня.

### 2. Train-test contamination
- Признак рассчитан на **полных** данных, включая test.
- ❌ Скалирование, fit_transform на test.

### 3. Multiple lookback overlap
- Если индикатор смотрит на 20 баров, а период расчёта — 5 баров, может быть утечка.

## Time-series CV с purging

```python
def purged_kfold(data, n_splits=5, embargo_pct=0.01):
    n = len(data)
    embargo = int(n * embargo_pct)
    fold_size = n // n_splits
    
    for i in range(n_splits):
        test_start = i * fold_size
        test_end = (i + 1) * fold_size
        train = np.concatenate([
            data[:test_start - embargo],  # до test с отступом
            data[test_end + embargo:]      # после test с отступом
        ])
        test = data[test_start:test_end]
        yield train, test
```

## Конкретные примеры

### 1. Z-score cross-section
```python
# ❌ Использует ВСЕ тикеры сегодня, включая будущие относительно других
zscore = (price - mean(all_prices_today)) / std(all_prices_today)

# ✅ Только данные, доступные на момент
zscore = (price.shift(1) - mean(all_prices.shift(1))) / std(all_prices.shift(1))
```

### 2. Rolling correlation
```python
# ❌ Окно включает текущий бар
corr = rolling_corr(a, b, 20)

# ✅ Окно только прошлых
corr = rolling_corr(a.shift(1), b.shift(1), 20)
```

### 3. Volatility estimate
- Используйте `return.shift(1).rolling(20).std()` — не `return.rolling(20).std()`.

## Чек-лист урока
- [ ] Все индикаторы сдвинуты на 1 (shift).
- [ ] Fill = next open.
- [ ] Нет adjusted close в решениях.
- [ ] Нет target leakage.
- [ ] Применяется purged CV.
- [ ] Описана точка принятия решения.