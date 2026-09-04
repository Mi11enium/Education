# Урок 48. JavaScript и SEO

## Проблема
Поисковые роботы не рендерят JavaScript так же, как браузер. Это может приводить к:
- ❌ Частичной или полной невидимости контента.
- ❌ Медленной индексации.
- ❌ Неполным сниппетам.
- ❌ Пустым страницам в выдаче.

## Как ПС рендерят JS
Современная двухэтапная модель:
1. **Crawl** — бот получает HTML (с сырым контентом).
2. **Render** — позже бот рендерит JS (если есть ресурсы, очередь).
3. **Index** — Google использует результат рендеринга.

⚠️ Это занимает **минуты → дни → недели**. JS-контент может индексироваться с задержкой.

## Какие технологии опасны

### ❌ Плохо для SEO
- **Angular, React, Vue, Svelte (SPA)** — без SSR.
- **CSR (client-side rendering)** — контент появляется только в браузере.
- **Хешированные ссылки** (`/#/page`).
- **Сложный JS** для важного контента.

### ✅ Хорошо
- **SSR (Server-Side Rendering)** — Next.js, Nuxt.js, SvelteKit.
- **SSG (Static Site Generation)** — Gatsby, Astro.
- **Dynamic Rendering** — fallback для ботов.
- **Pre-rendering** — Prerender.io.

## Решения

### 1. SSR / SSG
Контент генерируется на сервере → бот сразу видит HTML.

```js
// Next.js example
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}
```

✅ **Лучший вариант.**

### 2. Dynamic Rendering
Сервер определяет User-Agent → для ботов отдаёт HTML-снимок, для пользователей — обычный SPA.
- **Rendertron** (Google).
- **Prerender.io** (коммерческий).
- ⚠️ Google не рекомендует — это маскировка.

### 3. Hydration
SSR + клиентская гидратация (React 18, Next.js).
✅ Оптимально для современных приложений.

### 4. Incremental Static Regeneration (ISR)
Next.js: статические страницы с автообновлением.
✅ Быстро + свежо.

## Что проверить

### 1. Отключите JS
- Chrome DevTools → Disable JavaScript.
- Перезагрузите страницу.
- Виден ли основной контент?

### 2. View Source vs. Inspect
- View Source — что получает бот.
- Inspect — что рендерится.
- Если сильно различается — проблема.

### 3. Search Console → URL Inspection → Live Test
Показывает, что Google реально видит.

### 4. Lighthouse → SEO
- robots.txt валиден.
- Meta description есть.
- Ссылки имеют href.
- Alt у изображений.

## Что должно работать без JS
- ✅ Заголовок (Title).
- ✅ Meta description.
- ✅ Основной контент (хотя бы первый экран).
- ✅ Навигация (ссылки).
- ✅ Хлебные крошки.
- ✅ Alt у изображений.
- ✅ Canonical.

## Боты, которые НЕ рендерят JS
- **Bingbot** — почти не рендерят JS.
- **Соцсети-боты** (Twitter, Facebook, Telegram).
- ❗ Если для превью в соцсетях важен контент — нужен SSR.

## Лучшие практики

### 1. Progressive Enhancement
1. Базовый HTML доступен без JS.
2. CSS улучшает вид.
3. JS добавляет интерактивность.

### 2. Lazy Loading
- Изображения, видео — `loading="lazy"`.
- ⚠️ Не лейзите контент выше первого экрана.

### 3. Internal Linking
- Используйте `<a href="...">`.
- ❌ Не делайте SPA-навигацию через `<div onClick>`.

### 4. Schema
- Вставляйте в SSR-вывод.
- Не генерируйте на клиенте.

### 5. Canonical
- В SSR-выводе.

### 6. Sitemap
- Только канонические URL, доступные боту.

## Когда SPA оправдано
- Web-приложение (почта, редакторы).
- Контент за логином (не нужен в ПС).
- Внутренний инструмент.

❌ Не оправдано для:
- Контентных сайтов.
- E-commerce.
- Лендингов.

## Чек-лист урока
- [ ] Контент доступен без JS (Progressive Enhancement).
- [ ] Title, Description, H1 в SSR-выводе.
- [ ] Canonical в SSR-выводе.
- [ ] Schema в SSR-выводе.
- [ ] Internal links — настоящие `<a href>`.
- [ ] Lighthouse SEO > 95.