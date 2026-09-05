# Урок 49. Regime Change и адаптация

## Что это
**Regime change** — смена рыночного режима (тренд ↔ боковик, высокая ↔ низкая волатильность). Стратегии, работавшие вчера, ломаются.

## Примеры режимов

### 1. Trend ↔ Range
- Тренд: TREND FOLLOW.
- Range: MEAN REVERSION.

### 2. High Vol ↔ Low Vol
- VIX > 30 vs. VIX < 15.
- Разные стратегии.

### 3. Risk-on ↔ Risk-off
- Растут рисковые активы vs. падают.

### 4. Liquidity regimes
- QE (наличные вливания).
- QT (отток ликвидности).

## Почему важно

### Стратегия перестаёт работать
- Sharpe падает с 2.0 до 0.3.
- Корреляции меняются.
- ❌ Если стратегия статична — убытки.

## Детекция режима

### 1. Hidden Markov Models (HMM)
- 2-3 состояния.
- Вероятности переходов.
- Текущее состояние.

```python
from hmmlearn.hmm import GaussianHMM
model = GaussianHMM(n_components=2, covariance_type='full')
model.fit(returns)
states = model.predict(returns)
```

### 2. Rolling volatility
```python
vol_20 = returns.rolling(20).std()
vol_60 = returns.rolling(60).std()
# Если vol_20 > 1.5 × vol_60 → high vol regime
```

### 3. Trend strength (ADX)
```python
adx = talib.ADX(high, low, close, 14)
# ADX > 25: trend
# ADX < 20: range
```

### 4. Hurst exponent
- H > 0.5: trending.
- H < 0.5: mean-reverting.

### 5. Market regime classifiers
- ML-модели (Random Forest).
- Features: vol, momentum, breadth.

## Адаптивные стратегии

### 1. Multi-strategy portfolio
- Trend + Mean Reversion + Carry.
- В каждом режиме работает одна.
- Risk Parity между ними.

### 2. Regime-conditional parameters
- Разные параметры для разных режимов.

```python
if regime == 'trending':
    ma_period = 50
else:
    ma_period = 20
```

### 3. Regime-conditional signals
- Не торговать в «плохом» режиме.
- Спящий режим.

```python
if regime == 'unfavorable':
    return 'no_signal'
```

### 4. Adaptive sizing
- Увеличить в «своём» режиме.
- Уменьшить в «чужом».

### 5. Adaptive stop/target
- Шире в high-vol.
- Уже в low-vol.

## Online learning

### Идея
- Параметры обновляются по ходу.
- Не «зафиксированы» на train.

### Подходы
- **Exponentially weighted MA**: больше вес недавним.
- **Kalman filter**: адаптивный.
- **Bayesian**: posterior по новым данным.

```python
# EWMA
def ewma(x, alpha=0.1):
    result = [x[0]]
    for xi in x[1:]:
        result.append(alpha * xi + (1 - alpha) * result[-1])
    return result
```

## Когда переобучать стратегию

### Триггеры
- Sharpe < 0.5 за 3 месяца.
- DD > historical threshold.
- Correlation breakdown.
- Drawdown > 25%.

### Что делать
- Recalibrate параметры.
- Переоценить edge.
- Возможно, отключить стратегию.

### Caution
- Не оптимизировать «на лету» (overfit).

## Portfolio rotation

### Идея
- Несколько стратегий.
- Каждый месяц перебалансировать веса.
- Больше вес — «работающим».

```python
def rotate(strategies, lookback=60):
    scores = []
    for s in strategies:
        sharpe = recent_sharpe(s.returns, lookback)
        scores.append(sharpe)
    weights = np.exp(scores) / np.sum(np.exp(scores))
    return weights
```

## Structural breaks

### Тесты
- Chow test.
- CUSUM.
- Bai-Perron (multiple breaks).

### Действие
- При детектированном сдвиге → recalibrate.

## Сложности адаптации

### 1. Режим меняется в тесте
- Вы retrain на новых данных.
- ⚠️ Новая модель тоже может быть overfit.

### 2. False signals
- Режим определён неверно.
- Переключение стратегий = transaction costs.

### 3. Delay
- Режим определён с задержкой.
- Часть движения уже произошла.

## Когда НЕ адаптировать

- ❌ Если стратегия и так robust.
- ❌ Если мало данных.
- ❌ Если regime detection ненадёжна.

## Чек-лист урока
- [ ] Regime detection настроена.
- [ ] Multi-strategy portfolio.
- [ ] Adaptive sizing.
- [ ] Online learning (опционально).
- [ ] Rotation logic.