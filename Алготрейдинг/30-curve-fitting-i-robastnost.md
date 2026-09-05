# Урок 30. Curve Fitting и Robustness

## Curve Fitting (подгонка кривой)

### Определение
Стратегия подогнана под **шум** в исторических данных, а не под реальный сигнал.

### Признаки
- ❌ Много параметров (>5).
- ❌ Острые пики в heatmap.
- ❌ «Magic» параметры (37, 73, 91).
- ❌ Работает только на одном тикере/периоде.
- ❌ Сильный decay IS → OOS.

## Robustness (устойчивость)

### Определение
Стратегия сохраняет свойства при:
- Изменении параметров.
- Изменении периода.
- Изменении тикера.
- Изменении режима рынка.

## Тесты на Robustness

### 1. Parameter Sensitivity
Измените каждый параметр на ±10%, ±25%, ±50%.
- Sharpe не должен сильно меняться.

```python
def sensitivity_test(strategy, base_params, variation=0.1):
    results = {'base': backtest(strategy, base_params)}
    for param in base_params:
        for delta in [-variation, +variation]:
            new_params = base_params.copy()
            new_params[param] *= (1 + delta)
            results[f'{param}_{delta}'] = backtest(strategy, new_params)
    return results
```

### 2. Monte Carlo Robustness
См. урок 26.
- 95% CI должен включать только положительные значения.

### 3. Noise Injection
- Добавьте 1–5% шума к ценам.
- Стратегия не должна сильно меняться.

```python
def noise_injection_test(data, strategy, params, n_sims=100, noise_pct=0.01):
    sharpes = []
    for _ in range(n_sims):
        noise = np.random.normal(0, data * noise_pct)
        noisy_data = data + noise
        sharpes.append(backtest(strategy, noisy_data, params)['sharpe'])
    return np.std(sharpes), np.mean(sharpes)
```

### 4. Deflated Sharpe Ratio
- Коррекция на количество попыток.

### 5. White Reality Check
- Hansen (2005) test.
- Проверяет, что результат не случаен.

### 6. Combinatorially Symmetric CV
- Lopez de Prado.
- PBO (Probability of Backtest Overfitting).

## Сравнение стратегий по Robustness

### Метрика
```
Robustness_Score = mean(metric across perturbations) / std(metric)
```

- Высокий = robust.

## Best practices

### 1. Меньше параметров
- Occam's Razor.

### 2. Логически мотивированные параметры
- MA(20) = 1 месяц.
- ATR multiplier 2 = 2σ.

### 3. Плато, а не пик
- Выбирайте центр плато.

### 4. Подвыборки
- Разные периоды, разные тикеры.
- Среднее по подвыборкам.

### 5. Cross-validation
- Purged K-fold.

### 6. Out-of-sample — финальный
- Один раз, не подглядывать.

## Bias-Variance Tradeoff

### Bias (смещение)
- Насколько стратегия «промахивается» по истине.
- Слишком простая → high bias.

### Variance (дисперсия)
- Насколько сильно меняется при изменении данных.
- Слишком сложная → high variance.

### Tradeoff
- Сложная → low bias, high variance (overfit).
- Простая → high bias, low variance (underfit).

### Цель
- Найти sweet spot.
- Обычно 2–5 параметров.

## Visual Robustness

### Heatmap
- 2D: параметр 1 vs параметр 2.
- Цвет: Sharpe.
- Хорошо: большая жёлтая область.
- Плохо: маленький красный пик.

### 3D Surface
- 3 параметра — сложно визуализировать.
- Используйте pair plots.

### Performance Distribution
- По perturbation: histogram of Sharpe.
- Широкий = не robust.

## Домен Robustness

### Проверка по разным
- **Тикерам** (ES, NQ, CL, GC).
- **Периодам** (5 разных 3-летних окон).
- **Режимам** (bull, bear, flat, volatile).
- **Таймфреймам** (1h, 4h, 1d).

### Если только на одном — fragile

## Чек-лист урока
- [ ] Менее 5–7 параметров.
- [ ] Sensitivity test: Sharpe стабилен.
- [ ] Noise injection test пройден.
- [ ] DSR/PSR > 0.95.
- [ ] Работает на ≥3 тикерах/периодах.