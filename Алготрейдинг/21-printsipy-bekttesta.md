# Урок 21. Принципы бэктестинга

## Что такое бэктест
**Бэктест** — прокрутка стратегии на исторических данных для оценки её эффективности.

## Честный бэктест должен

### 1. Использовать только данные, доступные в реальном времени
- ❌ Знать будущее = look-ahead bias.
- ❌ Использовать откорректированные (adjusted) цены для решений.

### 2. Реалистично моделировать исполнение
- Комиссии.
- Slippage.
- Спреды.
- Задержки.
- Частичные исполнения.

### 3. Учитывать ликвидность
- Не входить в позицию больше, чем оборот за период.

### 4. Правильно обрабатывать дивиденды/корпоративные события
- Сплиты, дивиденды, mergers.

### 5. Не использовать будущие данные в индикаторах
- ❌ `MA(close, 20)` на close текущего бара (смотрит вперёд).
- ✅ `MA(close[:-1], 20)` или `MA(close.shift(1), 20)`.

## Типы бэктестов

### 1. Vectorized (векторизованный)
- Pandas/numpy без итераций.
- ✅ Быстро, легко писать.
- ❌ Сложно моделировать сложную логику.
- Инструменты: vectorbt, backtesting.py.

### 2. Event-driven
- Имитация реальной торговли.
- ✅ Реалистично, гибко.
- ❌ Медленнее.
- Инструменты: backtrader, zipline, lean.

### 3. Hybrid
- Комбинация.

## Требования к данным

### Качество
- ✅ Тиковые или минутные (не дневные) для интрадей.
- ✅ Adjusted prices.
- ✅ Без пропусков.
- ✅ Real volume (не синтетический).

### Хранение
- Parquet/HDF5 (сжатие).
- TimescaleDB / InfluxDB (для больших объёмов).
- Arctic (для финансовых данных).

## Ошибки в бэктестах

### 1. Look-ahead bias
```python
# ❌ ПЛОХО
signal = (close > sma(close, 20))

# ✅ ХОРОШО
signal = (close.shift(1) > sma(close.shift(1), 20))
```

### 2. Survivorship bias
- Данные содержат только «выжившие» тикеры.
- Решение: использовать delisted tickers.

### 3. Неправильный TZ
- Таймзоны. ET vs UTC vs MSK.
- Bar boundaries: 09:30–10:00 ≠ 09:00–10:00.

### 4. Рассогласование времени
- Fill time vs. signal time.
- Если стратегия смотрит close бара, fill = next open.

### 5. Не учитывать дивиденды
- Для акций.

### 6. Double counting
- Использовать обе цены (bid и ask) как «цену».

## Шаблон честного бэктеста

```python
# 1. Загрузить данные
data = load_clean_data()

# 2. Рассчитать индикаторы БЕЗ look-ahead
data['sma'] = data['close'].shift(1).rolling(20).mean()

# 3. Сгенерировать сигналы
data['signal'] = (data['close'].shift(1) > data['sma']).astype(int)

# 4. Рассчитать returns
data['strategy_return'] = data['signal'] * data['close'].pct_change()

# 5. Учесть комиссии и slippage
data['strategy_return_net'] = data['strategy_return'] - commission_per_trade

# 6. Equity curve
data['equity'] = (1 + data['strategy_return_net']).cumprod()
```

## Метрики бэктеста

### Обязательные
- Sharpe, Sortino, Calmar.
- MaxDD, Recovery.
- CAGR.
- Win Rate, Profit Factor.

### Дополнительные
- Avg Trade Duration.
- Exposure.
- Volatility.

## Визуализация

### Equity Curve
- Должна расти вверх.
- ⚠️ Гладкая = возможно, переобучение.

### Drawdown Plot
- Показывает худшие периоды.
- Должна быть < 20–30% (для большинства стратегий).

### Распределение PnL
- Histogram.
- Q-Q plot vs. normal.
- Skewness, kurtosis.

### Monte Carlo equity
- Перемешать сделки.
- Построить 1000+ equity curves.
- Доверительный интервал.

## Когда бэктест **не работает**

- ❌ Рынок структурно изменился.
- ❌ Стратегия на данных, которых не будет в реале (delisted).
- ❌ Ликвидность в бэктесте завышена.
- ❌ Slippage занижен.
- ❌ Сигналы на закрытом баре исполняются на этом же баре.

## Чек-лист урока
- [ ] Нет look-ahead в индикаторах.
- [ ] Учтены комиссии и slippage.
- [ ] Fill = next open (не close).
- [ ] Проверен survivorship bias.
- [ ] Equity curve и DD plot построены.