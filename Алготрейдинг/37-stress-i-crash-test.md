# Урок 37. Стресс-тестирование и краш-тесты

## Зачем
Бэктест на нормальных данных может дать ложное чувство безопасности. В кризис всё иначе: ликвидность исчезает, спреды расширяются, корреляции → 1.

## Типы стресс-тестов

### 1. Исторические сценарии
Прогнать стратегию на данных прошлых кризисов:
- 1987 Black Monday.
- 1998 LTCM.
- 2008 GFC.
- 2010 Flash Crash.
- 2015 China.
- 2018 Volmageddon.
- 2020 COVID.
- 2022 Bond crash.

### 2. Синтетические сценарии
- «Цена падает на 10% за 1 час».
- «Spread × 10».
- «Slippage × 5».
- «Ликвидность = 0 на 30 мин».

### 3. Reverse Stress Test
Найти сценарий, который разоряет стратегию.

## Методы

### 1. Bootstrapping кризисных периодов
- Выделить «crash periods» (волатильность > 90 перцентиля).
- Бутстрап только из них.

### 2. Добавление шока
```python
def apply_shock(prices, shock_pct=-0.10, duration=1):
    """Drop prices by shock_pct over duration bars"""
    shocked = prices.copy()
    shock_start = np.random.randint(0, len(prices) - duration)
    for i in range(duration):
        shocked.iloc[shock_start + i] *= (1 + shock_pct)
    return shocked
```

### 3. Volatility Multiplier
```python
def vol_multiplier(returns, multiplier=3):
    # Увеличить волатильность, сохранив направление
    adjusted = returns.copy()
    rolling_vol = returns.rolling(20).std()
    for i in range(20, len(returns)):
        target_vol = rolling_vol.iloc[i] * multiplier
        if target_vol > 0:
            adjusted.iloc[i] *= target_vol / abs(returns.iloc[i])
    return adjusted
```

### 4. Correlation Breakdown
```python
def breakdown_correlation(returns_dict):
    # Сделать все корреляции = 1
    for symbol in returns_dict:
        returns_dict[symbol] = returns_dict['market']
    return returns_dict
```

## Конкретные стресс-сценарии

### Сценарий 1: COVID-like (март 2020)
- Дни -10%, +5%, +10%, -7%, ...
- Spread × 5.
- Ликвидность -50%.

### Сценарий 2: Flash Crash
- 09:30: open -5%.
- 09:35: -10%.
- 09:40: +5% recovery.
- ⚠️ Стратегии со stop-loss могут сработать на -5%, потом -10%.

### Сценарий 3: Gap Down Overnight
- Close = 100.
- Open = 90.
- Stop-loss не сработал.

### Сценарий 4: Liquidity Crisis
- 1 час нет bid/ask.
- Потом открытие с -15%.

### Сценарий 5: Margin Call Cascade
- Принудительное закрытие → ещё падение.

## Анализ результатов

### Для каждого сценария
- MaxDD.
- Recovery time.
- Margin call?
- Slippage total.

### Worst-case aggregation
```python
worst_dd = min(all_max_dd_across_scenarios)
worst_pnl = min(all_final_pnl_across_scenarios)
```

## Reverse Stress Test

### Подход
1. Определить «катастрофический» исход (-50% equity).
2. Какие параметры рынка к нему приведут?
3. Насколько реалистичны эти параметры?

### Инструменты
- Sensitivity analysis.
- What-if scenarios.

## Stress-Test в бэктесте

### Добавление
```python
def stress_backtest(strategy, base_data, scenarios):
    results = {}
    for name, scenario in scenarios.items():
        modified_data = scenario(base_data)
        pnl = backtest(strategy, modified_data)
        results[name] = {
            'total_pnl': pnl.sum(),
            'maxdd': max_drawdown(pnl),
            'final_equity': pnl.sum() + 100000
        }
    return results
```

## Защитные механизмы

### 1. Position limits
- Max position per instrument.
- Max gross exposure.

### 2. Auto-stop при больших убытках
```python
def check_emergency_stop(equity, initial_capital):
    drawdown = (equity - initial_capital) / initial_capital
    if drawdown < -0.20:  # -20%
        return 'STOP_TRADING'
    return 'CONTINUE'
```

### 3. Volatility-based sizing
- Уменьшать размер при росте vol.

### 4. Liquidity check
- Не входить, если объём < X.

### 5. Kill switch
- Ручной или автоматический stop.

## Регулярность

### Когда проводить
- ✅ Перед запуском.
- ✅ Раз в квартал.
- ✅ После значимых рыночных событий.
- ✅ При изменении leverage/exposure.

## Чек-лист урока
- [ ] Проведены исторические стресс-тесты.
- [ ] Reverse stress test выполнен.
- [ ] Сценарии: COVID, Flash Crash, Gap, Liquidity.
- [ ] Worst-case PnL < 30% equity.
- [ ] Kill-switch настроен.