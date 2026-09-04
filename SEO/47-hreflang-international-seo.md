# Урок 47. Hreflang и международное SEO

## Что такое hreflang
**Hreflang** — HTML-атрибут `<link rel="alternate" hreflang="...">`, который сообщает поисковику о языковой/региональной версии страницы.

```html
<link rel="alternate" hreflang="ru" href="https://example.com/ru/page/" />
<link rel="alternate" hreflang="en" href="https://example.com/en/page/" />
<link rel="alternate" hreflang="x-default" href="https://example.com/page/" />
```

## Зачем нужен

### Проблемы без hreflang
- ❌ Пользователь из Франции видит англоязычную версию в Google.fr.
- ❌ Дублирование: ru и ua версии конкурируют.
- ❌ Google сам выбирает «неправильную» версию.

### Решает hreflang
- ✅ Показывает правильную версию нужной аудитории.
- ✅ Группирует локализованные страницы.
- ✅ Снижает риск дублей между локалями.

## Форматы значений hreflang

### Язык
`hreflang="ru"` — только русский.

### Язык + регион
`hreflang="ru-ru"` — русский для России.
`hreflang="ru-ua"` — русский для Украины.
`hreflang="en-us"` — английский для США.

### x-default
`hreflang="x-default"` — для пользователей, не подходящих ни под одну локализацию (catch-all).

## Структуры сайта для мультиязычности

### 1. Поддомены
```
ru.example.com — русский
en.example.com — английский
de.example.com — немецкий
```
✅ Простая локализация.
⚠️ Google может воспринимать как отдельные сайты.

### 2. Подпапки
```
example.com/ru/
example.com/en/
example.com/de/
```
✅ Один сайт, легче SEO-эффект.
✅ Рекомендуется Google.

### 3. Отдельные домены (ccTLD)
```
example.ru — Россия
example.de — Германия
example.com — международный
```
✅ Лучше для локального SEO.
❌ Дорого, сложно поддерживать.

### 4. URL-параметры
```
example.com/page?lang=ru
```
❌ **Не рекомендуется** Google (сложно определять).

## Как внедрять hreflang

### Способ 1. HTML-теги в `<head>`
```html
<link rel="alternate" hreflang="ru" href="https://example.com/ru/page/" />
<link rel="alternate" hreflang="en" href="https://example.com/en/page/" />
<link rel="alternate" hreflang="x-default" href="https://example.com/page/" />
```

### Способ 2. HTTP-заголовок
```http
Link: <https://example.com/en/page/>; rel="alternate"; hreflang="en"
```
Для не-HTML файлов.

### Способ 3. Sitemap
```xml
<url>
  <loc>https://example.com/ru/page/</loc>
  <xhtml:link rel="alternate" hreflang="en" href="https://example.com/en/page/"/>
  <xhtml:link rel="alternate" hreflang="ru" href="https://example.com/ru/page/"/>
</url>
```

## Правила hreflang

### 1. Возвратные ссылки (reciprocal)
Каждая страница должна ссылаться на все альтернативы (включая себя).

### 2. Self-reference
Каждая страница ссылается на саму себя с правильным hreflang.

### 3. Canonical
Каждая версия должна иметь canonical на саму себя (self-canonical).

### 4. Только релевантные коды
Не указывайте hreflang, если версии нет.
❌ `hreflang="fr"` без французской страницы — ошибка.

## Распространённые ошибки
❌ Неправильный формат кода (`ru-RU` с пробелом).
❌ Страница ссылается на другую, но та не ссылается обратно.
❌ Hreflang на 404 или canonicalized.
❌ Hreflang на noindex.
❌ Противоречие с canonical.

## Проверка
- **hreflang Tags Testing Tool** (Merkle).
- **Google Search Console → Международный таргетинг** → раздел «Язык».
- **Screaming Frog** — парсинг hreflang.

## Контент для разных локалей

### Перевод vs. локализация
- **Перевод** — точный перевод текста.
- **Локализация** — адаптация под культуру, валюту, единицы, примеры.

### Правило
Для ранжирования в конкретной стране — локализованный контент, а не автоматический перевод.

### Примеры
- Цены в национальной валюте.
- Примеры и кейсы для региона.
- Телефоны с местным кодом.
- Даты в местном формате.
- Учёт местного законодательства.

## Чек-лист урока
- [ ] У каждой страницы есть hreflang на все локали + x-default.
- [ ] Self-reference и reciprocal.
- [ ] Каждая страница имеет canonical на себя.
- [ ] Локализованный (не просто переведённый) контент.
- [ ] Структура — подпапки или поддомены.