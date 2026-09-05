# Урок 5. Языки программирования и инструменты

## Основные языки

### Python 🐍
- ✅ Огромная экосистема: pandas, numpy, scipy, scikit-learn, ta-lib.
- ✅ Быстрая разработка.
- ✅ Бэктест: backtrader, vectorbt, zipline, backtesting.py.
- ❌ Latency: GIL замедляет на CPU-bound задачах.
- ❌ Не подходит для HFT.
- **Лучший выбор для 90% стратегий**.

### C++ ⚙️
- ✅ Максимальная скорость.
- ✅ Используется в HFT.
- ❌ Долгая разработка.
- ❌ Сложно поддерживать.

### Rust 🦀
- ✅ Скорость как у C++.
- ✅ Безопаснее память.
- ❌ Экосистема для трейдинга растёт, но пока ограничена.

### C# / .NET
- ✅ NinjaTrader, QuantConnect Lean.
- ✅ Средне-высокая скорость.
- ❌ Платформозависимость.

### Java / Scala
- ✅ Spark, Kafka, big data.
- ❌ Не для latency-critical.

### MQL4/MQL5
- ✅ MetaTrader (форекс, СНГ).
- ❌ Слабая экосистема для серьёзного алго.

## Рекомендация по выбору

| Задача | Язык |
|---|---|
| Бэктест, research | Python |
| Live-торговля (среднечастотная) | Python + asyncio |
| Live HFT | C++ / Rust |
| MetaTrader | MQL5 |
| Распределённая система | Python + Kafka + Go |

## Python-экосистема

### Анализ данных
- `pandas` — таблицы, resample, rolling.
- `numpy` — массивы, векторизация.
- `polars` — быстрая альтернатива pandas.
- `scipy` — статистика, оптимизация.

### Финансы
- `pandas-datareader`, `yfinance` — котировки.
- `ta-lib` — индикаторы (быстрая).
- `pandas-ta` — pure Python.
- `backtrader`, `vectorbt`, `zipline-reloaded`, `backtesting.py` — бэктесты.
- `ccxt` — криптобиржи.
- `ib_insync` — IBKR API.
- `tinvest` — Tinkoff API.

### ML
- `scikit-learn` — классические модели.
- `xgboost`, `lightgbm`, `catboost` — градиентный бустинг.
- `pytorch`, `tensorflow` — нейросети.

### Визуализация
- `matplotlib`, `seaborn`, `plotly` — графики.
- `mplfinance` — свечные графики.

### Live-инфраструктура
- `asyncio` — асинхронность.
- `aiohttp` — HTTP клиент.
- `websockets` — WS клиент.
- `Redis` — кэш/очередь.
- `PostgreSQL`, `TimescaleDB` — хранение.

## IDE и инструменты разработки

- **VS Code** — универсальный.
- **PyCharm** — для Python.
- **Jupyter / Google Colab** — для research.
- **Git** — версионирование.
- **Docker** — контейнеризация.
- **pytest** — тестирование.
- **pre-commit** — линтинг.
- **black, ruff** — форматирование.
- **mypy** — типизация.

## Структура проекта

```
algo_project/
├── config/
│   └── settings.yaml
├── data/
│   ├── raw/             # сырые данные
│   └── processed/       # обработанные
├── src/
│   ├── data/            # загрузка, хранение
│   ├── features/        # индикаторы
│   ├── strategy/        # стратегии
│   ├── risk/            # риск-менеджмент
│   ├── execution/       # отправка ордеров
│   ├── backtest/        # бэктест фреймворк
│   └── live/            # live-трейдинг
├── tests/               # unit-тесты
├── notebooks/           # research
├── logs/
├── docker-compose.yml
└── pyproject.toml
```

## Чек-лист урока
- [ ] Выбран язык программирования.
- [ ] Установлены основные библиотеки.
- [ ] Настроена IDE.
- [ ] Есть структура проекта.
- [ ] Git репозиторий инициализирован.