# Урок 28. Технический долг бэктеста

## Что это
**Технический долг (technical debt)** — накопленные упрощения и костыли в коде бэктеста, которые со временем искажают результат.

## Типичный тех. долг

### 1. Упрощённая модель исполнения
```python
# ❌ Долг: всегда fill по close
pnl = position * (close - entry_price)
```

```python
# ✅ Честно: fill на next open + slippage
pnl = position * (next_open + slippage - entry_price)
```

### 2. Игнорирование дивидендов/сплитов
- Акции: нужна корректировка.
- Фьючерсы: ролл-over.

### 3. Фиксированный размер позиции
- Не учитывает волатильность.

### 4. Нет учёта ликвидности
- Позиция 10 000 контрактов на тонком рынке.

### 5. Игнорирование partial fills
- Ордер на 100 лотов, доступно 30 → 70 «исчезают».

### 6. Игнорирование расписания торгов
- Pre-market / after-hours.
- Холидей-эффекты.

## Код-сменты (Code smells)

### Magic numbers
```python
# ❌ Долг
if price > sma * 1.05:  # что такое 1.05?
    sell()

# ✅ Чисто
THRESHOLD = 1.05  # 5% выше MA — порог пробоя
if price > sma * THRESHOLD:
    sell()
```

### Дублирование
```python
# ❌ Один и тот же расчёт в нескольких местах
def strategy_a(data):
    sma = data.rolling(20).mean()
    ...

def strategy_b(data):
    sma = data.rolling(20).mean()
    ...
```

```python
# ✅ Одна функция
def compute_sma(data, period=20):
    return data.rolling(period).mean()
```

### Хардкод данных
```python
# ❌ Пути, тикеры захардкожены
df = pd.read_csv('/home/user/data/ES_2023.csv')
commission = 2.36  # ES

# ✅ Конфиг
df = load_data(config['data_path'], config['ticker'])
commission = config['commissions'][config['ticker']]
```

## Рефакторинг бэктеста

### Архитектура
```
data/
├── loader.py       # загрузка, очистка
├── features.py     # индикаторы
└── adjustments.py  # корпоративные события

strategy/
├── base.py         # BaseStrategy
├── trend.py        # трендследящие
└── meanrev.py      # mean reversion

execution/
├── model.py        # модель исполнения
├── slippage.py     # slippage
└── commission.py   # комиссии

backtest/
├── engine.py       # главный event loop
├── portfolio.py    # управление позицией
└── metrics.py      # метрики
```

### Конфигурация
```yaml
# config.yaml
data:
  ticker: ES
  start: 2010-01-01
  end: 2024-01-01
  
strategy:
  type: sma_cross
  params:
    fast: 20
    slow: 50
    
execution:
  commission: 2.36
  slippage_ticks: 1
  fill_on: 'next_open'
  
risk:
  position_size: 1
  max_leverage: 1.0
```

## Тестирование бэктеста

### Unit-тесты
```python
def test_commission_applied():
    pnl = backtest_no_commission()
    pnl_with_comm = backtest_with_commission()
    assert pnl_with_comm < pnl
    
def test_no_lookahead():
    # Сдвинуть данные на 1 бар вперёд
    # Результат должен сильно отличаться
    pnl1 = backtest(data)
    pnl2 = backtest(data.shift(1))
    assert abs(pnl1 - pnl2) > threshold
```

### Интеграционные тесты
- Реальный бэктест на известных данных.
- Сравнение с reference результатом.

### Регрессионные тесты
- После изменений в коде результат не должен сильно меняться.

## Документация

### Что фиксировать
- ✅ Версия данных.
- ✅ Параметры стратегии.
- ✅ Допущения модели.
- ✅ Известные ограничения.
- ✅ Hash кода и данных.

```python
# Снимок результата
snapshot = {
    'code_hash': git_hash(),
    'data_hash': data_hash(),
    'params': params,
    'metrics': metrics,
    'date': '2024-01-15',
    'notes': 'Initial test'
}
```

## Когда переписывать

### Сигналы
- Бэктест расходится с реальной торговлей.
- Добавился новый тип ордеров.
- Изменились данные / брокер.

### Стратегия
- Не incremental patches, а переписать с нуля.
- ⚠️ Сначала зафиксировать старые результаты.

## Чек-лист урока
- [ ] Код разделён на модули.
- [ ] Конфигурация вынесена.
- [ ] Нет magic numbers.
- [ ] Есть unit-тесты.
- [ ] Регрессионные тесты проходят.
- [ ] Документированы допущения.