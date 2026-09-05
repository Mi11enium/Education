# Урок 36. Walk-Forward vs. Cross-Validation

## Когда что применять
- **Walk-Forward (WF)** — для адаптивных стратегий.
- **Cross-Validation (CV)** — для оценки переобучения.
- **Combined** — лучший вариант.

## Walk-Forward подробно

### Параметры
- Train size: 3–10 лет.
- Test size: 3–6 мес.
- Step: 3–6 мес.

### Anchored
- Train: [0, T1].
- Test: [T1, T2].
- Train: [0, T2].
- Test: [T2, T3].
- ...
- Train всегда начинается с 0.

### Rolling
- Train: [0, T1].
- Test: [T1, T2].
- Train: [T1, T2].
- Test: [T2, T3].
- ...
- Train «катится».

### Реализация
```python
def walk_forward(data, train_size, test_size, step, strategy, param_grid):
    n = len(data)
    out_of_sample = []
    
    for start in range(0, n - train_size - test_size + 1, step):
        train = data.iloc[start:start + train_size]
        test = data.iloc[start + train_size:start + train_size + test_size]
        
        # Optimize on train
        best_params = optimize(strategy, train, param_grid)
        
        # Test on out-of-sample
        oos_result = backtest(strategy, test, best_params)
        out_of_sample.append(oos_result)
    
    return out_of_sample
```

## Cross-Validation подробно

### K-Fold (с purging)

```python
def purged_kfold(data, n_splits=5, embargo_pct=0.01):
    n = len(data)
    embargo = int(n * embargo_pct)
    fold_size = n // n_splits
    
    for i in range(n_splits):
        test_start = i * fold_size
        test_end = test_start + fold_size
        train_indices = list(range(0, test_start - embargo)) + \
                        list(range(test_end + embargo, n))
        test_indices = list(range(test_start, test_end))
        yield train_indices, test_indices
```

### Combinatorial Purged CV (CPCV)
- Несколько комбинаций train/test.
- Лучше оценивает overfitting.

## Сравнение

| Параметр | Walk-Forward | K-Fold CV |
|---|---|---|
| Учёт времени | ✅ | ⚠️ С purging |
| Реализм | ✅ | ⚠️ |
| Данные для train | Меньше | Больше |
| Адаптивность | ✅ | ❌ |
| Скорость | Медленно | Быстрее |
| Overfit detection | ⚠️ | ✅ |

## Когда что

### Используйте Walk-Forward если
- ✅ Стратегия адаптивная.
- ✅ Меняются режимы рынка.
- ✅ Реальная торговля.

### Используйте K-Fold CV если
- ✅ Оценка overfitting.
- ✅ Сравнение стратегий.
- ✅ Подбор гиперпараметров.

### Используйте оба если
- ✅ CV для подбора параметров.
- ✅ WF для финальной оценки.

## Метрики WF

### Walk-Forward Efficiency (WFE)
```
WFE = OOS_Sharpe / IS_Sharpe
```

- > 0.5: хорошо.
- < 0.3: плохо.

### Robustness
- Sharpe стабилен в окнах?
- Параметры стабильны?

### Combined OOS
- Склеить все test-периоды.
- Sharpe на combined.

## Метрики CV

### Sharpe по фолдам
- mean, std.
- Если std > 50% от mean — нестабильно.

### Deflated Sharpe
- С учётом количества тестов.

## Пример pipeline

### Stage 1: K-Fold для подбора
```python
best_params = {}
for param in param_grid:
    sharpe_scores = []
    for train_idx, val_idx in purged_kfold(data, n_splits=5):
        score = backtest(data.iloc[train_idx], data.iloc[val_idx], param)
        sharpe_scores.append(score)
    best_params[param] = np.mean(sharpe_scores)
```

### Stage 2: WF для оценки
```python
oos_results = walk_forward(data, train_size=1500, test_size=200, ...)
combined_sharpe = aggregate(oos_results)
```

## Failure modes

### WF даёт плохой результат
- Параметры не адаптируются.
- Слишком большой train (медленная адаптация).
- Слишком маленький test (нестабильные оценки).

### CV даёт плохой результат
- Overfitting.
- Неправильный purging.
- Нерепрезентативные фолды.

## Ансамбль моделей

### Идея
- Обучить на разных окнах.
- Объединить предсказания.

```python
def ensemble_backtest(data, n_models=10, train_size=1500, test_size=200):
    predictions = []
    for i in range(n_models):
        start = i * (test_size // 2)
        model = train_model(data.iloc[start:start+train_size])
        predictions.append(model.predict(data.iloc[start+train_size:]))
    return ensemble(predictions)
```

## Чек-лист урока
- [ ] Реализован Walk-Forward.
- [ ] WFE > 0.5.
- [ ] K-Fold с purging для CV.
- [ ] Pipeline: CV → WF → Live.
- [ ] Robustness оценена.