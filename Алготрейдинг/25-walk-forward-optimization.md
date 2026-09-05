# Урок 25. Walk-Forward Optimization

## Зачем
Простой train/test split имеет недостаток: фиксированная граница. **Walk-forward** — более честный метод адаптивной оптимизации.

## Идея
1. Обучить стратегию на окне [0, T1].
2. Протестировать на [T1, T2].
3. Сдвинуть окно: обучить на [T1, T2], протестировать на [T2, T3].
4. И так далее.
5. Склеить результаты тестов — это и есть walk-forward equity.

## Визуально

```
Train [════════] Test[═]  →  результат
        Train [════════] Test[═]  →  результат
              Train [════════] Test[═]  →  результат
                    ...
                            Train [════════] Test[═]  →  результат
```

## Типы walk-forward

### 1. Anchored (якорный)
- Train всегда начинается с начала данных.
- Test — следующий кусок.
- ✅ Максимум данных для train.
- ❌ Зависимость от старых данных.

### 2. Rolling (скользящий)
- Train и Test скользят вместе.
- Фиксированный размер train.
- ✅ Адаптивность.
- ✅ Более реалистично.

### 3. Expanding + Rolling комбинация

## Реализация

```python
def walk_forward(data, train_size, test_size, strategy, params_grid):
    n = len(data)
    results = []
    
    for start in range(0, n - train_size - test_size, test_size):
        train = data.iloc[start:start + train_size]
        test = data.iloc[start + train_size:start + train_size + test_size]
        
        # Оптимизация на train
        best_params = optimize(strategy, train, params_grid)
        
        # Тест на test
        test_pnl = run_strategy(strategy, test, best_params)
        results.append({
            'period': (test.index[0], test.index[-1]),
            'params': best_params,
            'pnl': test_pnl
        })
    
    return results
```

## Шаг оптимизации

### 1. Выбор метрики для оптимизации
- Sharpe (на train).
- Sortino.
- Calmar.
- Profit Factor.

### 2. Поиск параметров
- Grid Search.
- Random Search.
- Bayesian Optimization (Optuna).

### 3. Стабильность параметров
- Параметры не должны «прыгать» от окна к окну.
- Если прыгают — переобучение.

## Шаг оценки

### Walk-Forward Efficiency (WFE)

```
WFE = OOS_Strategy_Sharpe / IS_Sharpe
```

- WFE > 0.5 — хороший.
- WFE 0.3–0.5 — средний.
- WFE < 0.3 — плохо (overfit).

### Decay
- Если IS Sharpe 2, OOS Sharpe 0.4 → сильный decay.
- Если IS Sharpe 1, OOS Sharpe 0.8 → нормально.

## Что считать результатом

### Склеенный OOS equity
```python
total_oos_equity = concat(all_test_results)
```

### Метрики
- Sharpe (annualized).
- MaxDD.
- CAGR.
- WFE.

## Параметры walk-forward

### Размер окна train
- **Минимум**: 3–5 лет для дневных стратегий.
- **Лучше**: 5–10 лет.

### Размер окна test
- **Слишком короткое** (месяц): нестабильные метрики.
- **Слишком длинное** (год): медленная адаптация.
- **Оптимум**: 3–6 месяцев.

### Шаг сдвига
- Обычно = test_size.
- Или меньше (для более частых ребалансировок).

## Пример

Данные: 2010–2024.
- Train: 5 лет.
- Test: 6 месяцев.
- Шаг: 6 месяцев.
- Всего: ~28 окон.

## Проблемы

### 1. Параметры сильно «прыгают»
- Признак нестабильности.
- ❌ Не доверяйте.

### 2. Часть окон прибыльна, часть нет
- Нормально.
- Главное — общий PnL positive.

### 3. Look-ahead при оптимизации
- Параметры подобраны на train, но если в test включаются будущие данные — bias.
- ⚠️ Следите за корректностью окон.

## Anchored vs. Rolling

### Anchored
- Окно 1: 2010–2017 train, 2018 test.
- Окно 2: 2010–2018 train, 2019 test.
- Окно 3: 2010–2019 train, 2020 test.

### Rolling
- Окно 1: 2010–2014 train, 2015 test.
- Окно 2: 2011–2015 train, 2016 test.
- Окно 3: 2012–2016 train, 2017 test.

**Рекомендация**: Rolling ближе к реальности.

## Walk-Forward + Monte Carlo

1. Получить склеенный OOS equity.
2. Перемешать сделки.
3. Построить 1000+ equity curves.
4. CI на final PnL.

## Чек-лист урока
- [ ] Реализован walk-forward (rolling).
- [ ] Train ≥ 5 лет.
- [ ] Test = 3–6 месяцев.
- [ ] WFE > 0.5.
- [ ] Параметры стабильны между окнами.
- [ ] Склеенный OOS Sharpe > 0.