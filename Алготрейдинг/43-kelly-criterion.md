# Урок 43. Критерий Келли

## Что это
**Kelly criterion** — математически оптимальный размер ставки, максимизирующий долгосрочный логарифмический рост капитала.

## Формула (binary outcome)

```
f* = (P × b - q) / b
```

где:
- P = вероятность выигрыша.
- q = 1 - P = вероятность проигрыша.
- b = win/loss ratio (например, 2.0).

### Пример
- P = 0.6, b = 1.5.
- f* = (0.6 × 1.5 - 0.4) / 1.5 = (0.9 - 0.4) / 1.5 = **0.333**.

⚠️ 33% от банка на каждую сделку — агрессивно.

## Формула (continuous)

```
f* = μ / σ²
```

где:
- μ = средняя доходность.
- σ = волатильность.

### Пример
- μ = 10% годовых, σ = 15%.
- f* = 0.10 / 0.0225 = **4.44**.

⚠️ Leverage 4.4x.

## Почему Kelly

### Максимизирует
- E[log(W)] = log(W_0) + n × E[log(1 + f × R)].

### Свойства
- Долгосрочно max growth.
- ❌ Высокая волатильность пути.
- ❌ Может быть разорение при завышенных параметрах.

## Half-Kelly и Quarter-Kelly

### Почему уменьшают
- Kelly завышает при неопределённости.
- Реальные параметры оцениваются с ошибкой.

### Half-Kelly
- 0.5 × f*.
- ✅ Меньше просадки.
- ✅ Почти такой же growth (в long-term).

### Quarter-Kelly
- 0.25 × f*.
- ✅✅ Консервативно.

### Что выбрать
- Точные параметры: Kelly или 0.5 Kelly.
- Неточные: 0.25 Kelly.
- Паранойя: 0.1 Kelly.

## Пример на стратегии

### Параметры
- Win rate: 55%.
- Avg win: $200.
- Avg loss: $100.
- b = 2.

### Kelly
- f* = (0.55 × 2 - 0.45) / 2 = (1.1 - 0.45) / 2 = 0.325.

### Half-Kelly
- 0.16.

### На счёте $100k
- Kelly: $32 500 на сделку.
- Half-Kelly: $16 250.
- 0.25 Kelly: $8 125.

## Проблемы Kelly

### 1. Оценка параметров
- P, b оцениваются с ошибкой.
- Неправильная оценка → ruin.

### 2. Variance
- Kelly = max growth, не min variance.
- Реальный путь — сильно колеблется.

### 3. Non-ergodic
- «Долгосрочно» — бесконечность.
- Конкретный трейдер — конечен.

### 4. Ruin Risk
- При leverage > 1 и неблагоприятной серии — margin call.

## Модификации

### 1. Fractional Kelly
- Использовать долю от Kelly.

### 2. Bayesian Kelly
- Параметры как распределения, не точки.
- Интегрирование.

```python
def bayesian_kelly(win_rate_dist, b_dist):
    f_stars = []
    for p, b in zip(win_rate_dist, b_dist):
        f_stars.append((p * b - (1 - p)) / b)
    return np.percentile(f_stars, 25)  # use 25th percentile
```

### 3. Drawdown-constrained Kelly
- Kelly при ограничении MaxDD.

```python
def dd_constrained_kelly(p, b, max_dd=0.25, n_trades=100):
    # f, при котором MaxDD за n_trades < max_dd
    ...
```

## Geometric vs. Arithmetic

### Kelly max growth геометрический
- E[log(1 + f × R)].
- Сложнее оценить.

### Часто используют арифметическое среднее
- E[R] — проще.
- Может отличаться от Kelly.

## Симуляция Kelly

```python
def simulate_kelly(p, b, f, n=1000, n_sims=1000):
    final_wealths = []
    for _ in range(n_sims):
        wealth = 1.0
        for _ in range(n):
            if np.random.random() < p:
                wealth *= (1 + f * b)
            else:
                wealth *= (1 - f)
        final_wealths.append(wealth)
    return np.median(final_wealths), np.percentile(final_wealths, 5)
```

## Когда Kelly опасен

### 1. Leverage > 1
- С leverage 5x и Kelly 0.3 → реальный leverage 1.5x.
- Margin call при серии убытков.

### 2. Неточные P, b
- Переоценили P → oversize.
- Реальная ruin.

### 3. Непрерывная торговля
- В реальности — дискретные тики, спреды.

### 4. Параметры меняются
- Kelly статичен.
- ❌ Не адаптивен.

## Альтернативы

### Optimal f (Vince)
- Подбирается по истории.
- Max f, при котором нет ruin.

### Volatility targeting
- 15% target vol.
- Безопаснее.

### Drawdown-based
- Уменьшение при DD.

## Чек-лист урока
- [ ] Знаете формулу Kelly.
- [ ] Используете 0.25–0.5 Kelly.
- [ ] Учитываете оценку параметров.
- [ ] DD constraint применён.
- [ ] Симулировали Kelly на истории.