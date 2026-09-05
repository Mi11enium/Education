# Урок 40. Отчёты и дашборды

## Зачем
Регулярные отчёты:
- Видеть, что работает.
- Обнаруживать проблемы.
- Отчитываться перед клиентом / инвестором / командой.

## Типы отчётов

### 1. Live Performance Report
- Текущая PnL.
- Открытые позиции.
- Margin.
- Сегодняшние сделки.

### 2. Strategy Performance Report
- Sharpe, Sortino, Calmar.
- Equity curve.
- Drawdown chart.
- PnL distribution.

### 3. Risk Report
- Exposure.
- Leverage.
- VaR.
- Margin usage.

### 4. Trade Analysis
- Win/loss breakdown.
- Best/worst trades.
- Time in trade.

### 5. Operational Report
- Up-time.
- Latency.
- Error rate.
- Slippage.

## Ежедневный отчёт

```python
def daily_report(date):
    pnl_today = pnl[date]
    equity_today = equity[date]
    drawdown_today = dd[date]
    
    return {
        'date': date,
        'pnl_today': pnl_today,
        'equity': equity_today,
        'drawdown_pct': drawdown_today * 100,
        'n_trades': len(trades[trades['date'] == date]),
        'open_positions': get_open_positions(),
        'margin_used': get_margin(),
    }
```

## Недельный отчёт

### Метрики
- PnL week.
- Sharpe week.
- Equity chart.
- Top-5 trades.
- Bottom-5 trades.
- New highs/lows.
- New positions.

## Месячный отчёт

### Полный set
- PnL month.
- Sharpe MTD / YTD.
- Equity curve.
- DD curve.
- Attribution (по типу, по инструменту).
- Compare с benchmark.
- Risk metrics (VaR, Sortino, Calmar).
- Operational stats.

## Визуализация

### Equity Curve
```python
import matplotlib.pyplot as plt
fig, ax = plt.subplots(figsize=(12, 6))
ax.plot(equity, label='Strategy')
ax.plot(benchmark_equity, label='Benchmark', alpha=0.5)
ax.fill_between(dd.index, equity, equity.cummax(), 
                where=equity < equity.cummax(), color='red', alpha=0.3)
ax.legend()
ax.set_title('Equity Curve')
ax.set_ylabel('Equity ($)')
ax.grid(True)
```

### Drawdown Plot
```python
ax.fill_between(dd.index, 0, dd * 100, color='red', alpha=0.5)
ax.set_title('Drawdown (%)')
ax.set_ylabel('Drawdown %')
```

### Monthly Returns Heatmap
```python
import seaborn as sns
monthly_returns = returns.resample('M').sum()
sns.heatmap(monthly_returns.pivot_table(
    index=monthly_returns.index.year,
    columns=monthly_returns.index.month
), annot=True, fmt='.1%', cmap='RdYlGn')
```

### Rolling Sharpe
```python
rolling_sharpe = returns.rolling(252).mean() / returns.rolling(252).std() * np.sqrt(252)
ax.plot(rolling_sharpe)
ax.axhline(0, color='red', linestyle='--')
ax.set_title('Rolling 1y Sharpe')
```

## QuantStats (автоматический отчёт)

```python
import quantstats as qs

# Полный HTML отчёт
qs.reports.html(returns, output='report.html', 
                title='Strategy Performance')

# Или:
qs.reports.basic(returns)
qs.reports.full(returns, benchmark=benchmark)
```

## Live Dashboard

### Streamlit
```python
import streamlit as st

st.title('Trading Dashboard')
st.metric('Equity', f'${equity:.0f}', delta=f'${today_pnl:.0f}')
st.line_chart(equity)
st.metric('Drawdown', f'{dd:.1%}')
st.metric('Sharpe', f'{sharpe:.2f}')
st.dataframe(trades.tail(10))
```

### Grafana
- Сложнее, но production-grade.
- Подключение к БД.

## Alerts

### Что мониторить
- ⚠️ Daily PnL < -X%.
- ⚠️ Drawdown > -X%.
- ⚠️ Margin > X%.
- ⚠️ Connection lost > X min.
- ⚠️ Error rate > X%.
- ⚠️ Latency > X ms.
- ⚠️ Position > limit.

```python
def check_alerts():
    if daily_pnl < -0.02 * equity:
        send_alert('Daily loss limit hit')
    if drawdown < -0.15:
        send_alert('Drawdown 15%')
```

## Автоматизация

### Schedule
- Ежедневный отчёт в 9:00.
- Недельный в пятницу.
- Месячный 1-го числа.

### Email / Telegram
- Готовые отчёты.
- Alerts в Telegram.

## Чек-лист урока
- [ ] Ежедневный отчёт.
- [ ] Недельный отчёт.
- [ ] Месячный отчёт с атрибуцией.
- [ ] Дашборд (Streamlit / Grafana).
- [ ] Alerts настроены.
- [ ] QuantStats HTML генерируется.