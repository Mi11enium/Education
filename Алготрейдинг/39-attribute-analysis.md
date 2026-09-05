# Урок 39. Атрибуция PnL

## Зачем
Зная, **откуда** приходит прибыль, можно:
- Усилить прибыльные компоненты.
- Ослабить убыточные.
- Понять, что именно работает.

## Виды атрибуции

### 1. По времени
- По дням недели.
- По часам дня.
- По месяцам.
- По годам.

### 2. По инструменту
- Если торгуете несколько тикеров.

### 3. По типу сигнала
- Long vs short.
- Trend vs mean reversion.

### 4. По рыночному режиму
- Bull / bear / flat.
- High vol / low vol.

## Реализация

```python
def attribute_by_time(returns, data):
    # По дню недели
    returns.index = pd.to_datetime(returns.index)
    by_day = returns.groupby(returns.index.dayofweek).sum()
    
    # По часу
    by_hour = returns.groupby(returns.index.hour).sum()
    
    # По месяцу
    by_month = returns.groupby(returns.index.month).sum()
    
    return by_day, by_hour, by_month
```

### Визуализация
```python
import seaborn as sns
sns.heatmap(pnl_by_day_hour, annot=True, fmt='.0f')
```

## Brinson Attribution

### Идея
Разложить PnL на компоненты:
- Allocation: выбор тикеров/инструментов.
- Selection: выбор направления.
- Interaction: их комбинация.

```python
def brinson(weights, returns, benchmark_weights):
    allocation = sum((weights - benchmark_weights) * benchmark_returns)
    selection = sum(benchmark_weights * (returns - benchmark_returns))
    interaction = sum((weights - benchmark_weights) * (returns - benchmark_returns))
    return allocation, selection, interaction
```

## По типу стратегии

### Если торгуете 2 стратегии
- Strategy A: trend.
- Strategy B: mean reversion.

```python
pnl_total = pnl_a + pnl_b
contribution_a = pnl_a.sum() / pnl_total.sum()
contribution_b = pnl_b.sum() / pnl_total.sum()
```

## Regime attribution

### Определение режима
```python
def classify_regime(returns, vol_window=20):
    vol = returns.rolling(vol_window).std()
    trend = returns.rolling(vol_window).mean()
    
    regime = []
    for i in range(len(returns)):
        if vol.iloc[i] > vol.quantile(0.75):
            regime.append('high_vol')
        elif vol.iloc[i] < vol.quantile(0.25):
            regime.append('low_vol')
        elif trend.iloc[i] > 0:
            regime.append('bull')
        else:
            regime.append('bear')
    return regime
```

### PnL по режимам
```python
for regime in set(regimes):
    mask = np.array(regimes) == regime
    pnl_regime = pnl[mask]
    print(f'{regime}: total PnL = {pnl_regime.sum()}')
```

## Что искать

### 1. Концентрация прибыли
- 80% PnL от 20% дней.
- ⚠️ Рискованно.
- Стратегия зависит от «удачных моментов».

### 2. Зависимость от режима
- Прибыльна только в bull.
- ❌ Не диверсифицирована.

### 3. Зависимость от ликвидности
- Работает только в активные часы.

### 4. Дни недели
- Эффект понедельника / пятницы.

## Exposure-based attribution

### По типу exposure
- Directional (long/short bias).
- Spread (futures calendar).
- Volatility (option-like).

```python
def exposure_pnl(positions, returns):
    pnl = (positions.shift(1) * returns).sum()
    return pnl
```

## Пример

### Стратегия: trend following на 1d
- 2010-2015: прибыльна.
- 2015-2019: убыточна (chop).
- 2020-2024: прибыльна (тренды).

### Атрибуция
- 60% прибыли — 2020-2021 (bull, тренд).
- 30% прибыли — 2010-2014.
- 10% — 2018 короткий шорт.

⚠️ Зависит от сильных трендов.

## Практические выводы

### 1. Если стратегия прибыльна только в одном режиме
- ⚠️ Diversify across regimes.
- Или явно торговать только в этом режиме.

### 2. Если PnL концентрирован в нескольких днях
- Может быть случайность.
- ⚠️ Недостаточно данных.

### 3. Если 80% PnL от 1 года
- ❌ Подозрительно.
- Может быть overfit на конкретный период.

## Чек-лист урока
- [ ] Атрибуция по времени сделана.
- [ ] По типу стратегии.
- [ ] По режимам рынка.
- [ ] PnL не слишком концентрирован.
- [ ] Стратегия работает в нескольких режимах.