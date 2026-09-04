# Урок 43. Google Analytics 4 (GA4) для SEO

## Что такое GA4
**Google Analytics 4** — обновлённая версия Universal Analytics (закрыт с июля 2023). Основана на событиях и пользователях, а не на сессиях.

`https://analytics.google.com`

## Ключевые отличия от UA

| Параметр | UA | GA4 |
|---|---|---|
| Модель | Сессии + просмотры страниц | События + пользователи |
| Cookie | Свои | Свои + Google Signals |
| Идентификатор | Client ID | User ID |
| Приватность | Слабая | Privacy-first |
| ML/AI | Базовые | Расширенные |
| Funnels | Цели | Conversion events |

## Установка
1. Создать аккаунт GA4.
2. Создать ресурс.
3. Получить Measurement ID (G-XXXXXXX).
4. Установить код (gtag.js, GTM или плагин CMS).
5. Связать с GSC для данных по запросам.

## События и параметры

### Автоматические события
- `page_view` — просмотр страницы.
- `scroll` — скролл (90%).
- `click` — клик.
- `view_search_results` — внутренний поиск.
- `file_download` — скачивание.
- `video_progress` — видео.

### Рекомендуемые события
- `sign_up` — регистрация.
- `login` — вход.
- `purchase` — покупка.
- `add_to_cart`, `begin_checkout`.
- `search` — поиск.
- `share` — шейр.

### Custom события
Вы сами определяете события через GTM или код:
```js
gtag('event', 'cta_click', {
  'cta_text': 'Купить',
  'cta_location': 'header'
});
```

## Конверсии (Conversion events)
В GA4 нет «целей», но есть **conversion events** — это просто отмеченные события.
1. Администратор → События.
2. Найдите нужное событие.
3. Переключите «Mark as conversion».

## Отчёты

### Готовые отчёты
- **Realtime** — что происходит сейчас.
- **Acquisition** — откуда пришли.
- **Engagement** — что делают на сайте.
- **Monetization** — покупки (для e-commerce).
- **Retention** — возвраты.
- **Demographics** — демография.

### Исследования (Explore)
- Свободные отчёты.
- Сегменты, воронки, пути.

## Organic Search отчёт

### Где найти
- Отчёты → Привлечение → Привлечение трафика → Источник/Medium.

### Что смотреть
- **Сессии** с `medium = organic`.
- **Engaged sessions** — сессии >10 сек или с событиями.
- **Engagement rate** — % engaged sessions.
- **Avg engagement time** — среднее время взаимодействия.

### Фильтры
- Только `organic search`.
- Только `google / organic`.
- Только `yandex / organic`.

## Кастомный отчёт по SEO

### Настройка
1. Отчёты → Создать отчёт.
2. Вкладки: обзор + детализация.
3. Метрики: Organic Sessions, Engaged Sessions, Engagement Rate.
4. Разбивка: Landing Page, Source/Medium.
5. Фильтр: Source contains 'google' OR 'yandex'.

## Атрибуция

### Доступные модели
- **Data-driven** — алгоритм (по умолчанию, требует данных).
- **Last click** — последний клик.
- **First click** — первый.
- **Linear, position-based, time decay**.

### Для SEO
- ⚠️ Last non-direct click → SEO получает сессию после любого другого источника.
- Лучше — data-driven.

## Связь с GSC
1. GA4 → Администратор → Связь с Search Console.
2. После связки — в отчётах появляются данные GSC (запросы, CTR, позиции).

## Настройка для контент-менеджера

### Важные события
1. `page_view` (автоматический).
2. `cta_click` (на все кнопки).
3. `form_submit` (заявка).
4. `scroll` (90% страницы).
5. `share` (поделились статьёй).
6. `download` (скачали материал).

### Воронка
- Страница → Скролл 90% → CTA клик → Форма → Submit.

## Что НЕ работает в GA4
- ❌ Bounce rate (заменён на engagement rate).
- ❌ Просмотры страниц по умолчанию.
- ❌ Цели (заменены conversion events).

## Альтернативы
- **Яндекс.Метрика** — для РФ.
- **Matomo** — open source.
- **Plausible / Fathom** — privacy-friendly.

## Чек-лист урока
- [ ] GA4 установлен.
- [ ] Связан с GSC.
- [ ] Настроены conversion events.
- [ ] Умеете строить organic search отчёт.
- [ ] Внедрены custom события для CTA, форм, скролла.