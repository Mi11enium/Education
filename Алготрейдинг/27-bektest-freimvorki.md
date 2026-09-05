# Урок 27. Фреймворки для бэктестинга

## Python фреймворки

### 1. vectorbt
**Плюсы:**
- ✅ Очень быстрый (Numba-оптимизация).
- ✅ Векторизованный.
- ✅ Хорошая визуализация.
- ✅ Поддержка walk-forward.

**Минусы:**
- ❌ Сложная отладка.
- ❌ Не подходит для сложной execution-логики.

```python
import vectorbt as vbt
import pandas as pd

prices = pd.Series([...])
fast_ma = vbt.MA.run(prices, 10)
slow_ma = vbt.MA.run(prices, 50)
entries = fast_ma.ma_crossed_above(slow_ma)
exits = fast_ma.ma_crossed_below(slow_ma)

pf = vbt.Portfolio.from_signals(prices, entries, exits)
pf.total_return()
pf.sharpe_ratio()
```

### 2. backtrader
**Плюсы:**
- ✅ Event-driven.
- ✅ Гибкий.
- ✅ Большое сообщество.
- ✅ Множество индикаторов.

**Минусы:**
- ❌ Медленнее vectorbt.
- ❌ API иногда неудобен.

```python
import backtrader as bt

class MyStrategy(bt.Strategy):
    def __init__(self):
        self.sma = bt.indicators.SMA(period=20)
    
    def next(self):
        if self.data.close[0] > self.sma[0]:
            self.buy()
        elif self.data.close[0] < self.sma[0]:
            self.sell()

cerebro = bt.Cerebro()
cerebro.addstrategy(MyStrategy)
cerebro.adddata(data)
cerebro.run()
```

### 3. backtesting.py
**Плюсы:**
- ✅ Простой API.
- ✅ Хорошая документация.
- ✅ Подходит для быстрого прототипирования.

**Минусы:**
- ❌ Ограниченная кастомизация.
- ❌ Только дневные данные «из коробки».

```python
from backtesting import Backtest, Strategy

class SmaCross(Strategy):
    def init(self):
        self.sma = self.I(lambda: self.data.Close.rolling(20).mean())
    
    def next(self):
        if self.data.Close[-1] > self.sma[-1]:
            self.buy()
        elif self.data.Close[-1] < self.sma[-1]:
            self.position.close()

bt = Backtest(data, SmaCross, commission=0.001)
stats = bt.run()
bt.plot()
```

### 4. zipline-reloaded
**Плюсы:**
- ✅ Quantopian legacy.
- ✅ Реалистичный (event-driven).
- ✅ Pipeline API.

**Минусы:**
- ❌ Сложный setup.
- ❌ Меньше сообщество.

### 5. QuantConnect Lean
**Плюсы:**
- ✅ Поддержка multi-asset.
- ✅ Cloud platform.
- ✅ Поддержка Options/Forex/Crypto.

**Минусы:**
- ❌ C# основной язык (есть Python).

### 6. bt (backtesting for humans)
**Плюсы:**
- ✅ Дерево стратегий.
- ✅ Аллокация между стратегиями.

**Минусы:**
- ❌ Не так популярен.

## Сравнение

| Фреймворк | Скорость | Гибкость | Простота | Реализм |
|---|---|---|---|---|
| vectorbt | ★★★★★ | ★★★ | ★★★ | ★★ |
| backtrader | ★★★ | ★★★★ | ★★★ | ★★★★ |
| backtesting.py | ★★★ | ★★ | ★★★★★ | ★★★ |
| zipline | ★★★ | ★★★★ | ★★ | ★★★★ |
| QuantConnect | ★★★★ | ★★★★★ | ★★★ | ★★★★★ |
| Свой код | ★★ | ★★★★★ | ★★ | ★★★★★ |

## Выбор фреймворка

### Исследование (research)
- **vectorbt** или свой код на pandas.

### Прототипирование
- **backtesting.py** для быстрых тестов.

### Серьёзный бэктест
- **backtrader** или **zipline**.

### Live trading
- Свой код или QuantConnect.

### HFT
- Свой код на C++/Rust.

## Свой фреймворк

### Когда стоит писать
- ✅ Уникальная execution-логика.
- ✅ Нестандартные данные.
- ✅ Полный контроль.

### Архитектура
```python
class Backtester:
    def __init__(self, data, strategy, params):
        self.data = data
        self.strategy = strategy
        self.params = params
    
    def run(self):
        # 1. Генерация сигналов
        signals = self.strategy.generate(self.data, self.params)
        # 2. Симуляция исполнения
        trades = self.simulate(signals)
        # 3. Расчёт PnL с комиссиями
        pnl = self.calculate_pnl(trades)
        # 4. Метрики
        metrics = self.metrics(pnl)
        return trades, metrics
    
    def simulate(self, signals):
        # event loop
        ...
```

## Специализированные библиотеки

### TA-Lib / pandas-ta
- Индикаторы.
- Быстро.

### empyrical / pyfolio
- Метрики.
- Tearsheet.

### QuantStats
- Анализ returns.
- HTML-отчёты.

```python
import quantstats as qs
qs.reports.html(returns, output='report.html')
```

## Чек-лист урока
- [ ] Выбран фреймворк под задачу.
- [ ] Бэктест реалистичный (slippage, комиссии).
- [ ] Walk-forward реализован.
- [ ] QuantStats отчёт генерируется.
- [ ] Результаты визуализированы.