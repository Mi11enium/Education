# Урок 48. Мониторинг и алерты

## Зачем
Алготрейдинг требует **24/7 мониторинга**. Сбой без алерта = потеря денег.

## Что мониторить

### Торговля
- Открытые позиции.
- PnL (текущий, дневной).
- Margin.
- Exposure.
- Количество сделок.

### Технические
- Latency (отправка ордеров).
- Размер очереди.
- Ошибки.
- CPU / RAM / Disk.
- Network.

### Стратегии
- Сигналы.
- Сделок в час.
- Win rate (rolling).
- Sharpe (rolling).

## Метрики

### Real-time
- Equity: 100,250.
- DD: -2.3%.
- Open positions: 1.
- Margin: 12,000 / 100,000 = 12%.

### За день
- PnL: +500.
- N trades: 5.
- Win rate: 60%.
- Slippage avg: 0.5 ticks.

### За неделю
- PnL: +1,500.
- Sharpe: 1.8.
- DD: -4.2%.

## Алерты

### Категории

#### Critical (всегда будить)
- ❌ Connection lost > 1 min.
- ❌ Margin call.
- ❌ Daily loss > 5%.
- ❌ Position limit breach.
- ❌ Order error rate > 10%.

#### Warning (днём)
- ⚠️ DD > 10%.
- ⚠️ Latency > X ms.
- ⚠️ Unusual volatility.
- ⚠️ Не было сигналов > 24h.

#### Info
- ℹ️ Daily PnL summary.
- ℹ️ Недельный отчёт.
- ℹ️ Новый max equity.

## Реализация

### Структура
```python
class Monitor:
    def __init__(self):
        self.alerts = []
        self.metrics = {}
    
    def check_metrics(self):
        # Critical
        if self.daily_loss < -0.05 * self.equity:
            self.alert('CRITICAL', f'Daily loss {self.daily_loss}')
        
        if self.dd < -0.20:
            self.alert('CRITICAL', f'DD {self.dd:.1%}')
        
        # Warning
        if self.latency > 100:
            self.alert('WARNING', f'Latency {self.latency}ms')
    
    def alert(self, level, message):
        if level == 'CRITICAL':
            send_telegram(message)
            send_email(message)
            send_sms(message)
        elif level == 'WARNING':
            send_telegram(message)
```

## Каналы алертов

### Telegram Bot
- ✅ Быстро, удобно.
- ✅ Можно интерактивно.
- API: python-telegram-bot.

### Email
- ✅ Архив.
- ❌ Медленно.

### SMS
- ✅ Самый быстрый.
- ❌ Дорого.

### Slack
- ✅ Команда.
- ✅ Интеграции.

## Telegram-бот

```python
import telegram

bot = telegram.Bot(token=TOKEN)

def send_alert(message):
    bot.send_message(chat_id=CHAT_ID, text=message)

# При алерте
send_alert(f'⚠️ DD = -15%! Equity: {equity}')
```

## Дашборд

### Streamlit (простой)
```python
import streamlit as st

st.title('Trading Bot Monitor')
st.metric('Equity', f'${equity:,.0f}', f'{pnl_today:,.0f}')
st.metric('Drawdown', f'{dd:.1%}')
st.metric('Open Positions', n_positions)
st.line_chart(equity_history)
```

### Grafana (продвинутый)
- Подключение к БД.
- Сложные графики.
- Alerts в Grafana.

## Health Check

### Периодический
- Heartbeat каждые 60 сек.
- Если пропущен → алерт.

```python
import schedule

def heartbeat():
    send_status('OK', equity, pnl)

schedule.every(1).minutes.do(heartbeat)
```

### Проверка состояния
- Процесс жив.
- API доступен.
- Позиции = ожидаемым.
- PnL = ожидаемому.

## Логирование

### Что логировать
- Каждый тик (или сэмпл).
- Каждый сигнал.
- Каждый ордер.
- Каждое исполнение.
- Каждый fill.
- Каждое изменение позиции.
- Ошибки.

### Структура
```python
log.info({
    'event': 'order_sent',
    'timestamp': ts,
    'order_id': order_id,
    'symbol': 'ES',
    'side': 'buy',
    'qty': 1,
    'price': 4500.25
})
```

### Хранение
- Файлы (rotating).
- БД (TimescaleDB, ClickHouse).
- Cloud (S3 + Athena).

## On-call

### Если что-то пошло не так
1. Получен алерт.
2. Оценить severity.
3. Принять меры:
   - Остановить стратегию.
   - Закрыть позиции.
   - Перезапустить систему.
4. Post-mortem.

### Runbook
- Что делать при каждом типе алерта.
- Контакты брокера.
- API для emergency stop.

## Регулярные проверки

### Ежечасно
- Heartbeat.
- Open positions.
- PnL.

### Ежедневно
- PnL итог.
- Win rate.
- Slippage avg.

### Еженедельно
- Полный отчёт.
- Сравнение с backtest.
- Recalibration при необходимости.

## Чек-лист урока
- [ ] Heartbeat каждые 1–5 мин.
- [ ] Critical alerts по SMS/Telegram.
- [ ] Dashboard (Streamlit / Grafana).
- [ ] Логирование всех событий.
- [ ] Runbook для инцидентов.