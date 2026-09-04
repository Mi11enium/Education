# Урок 20. Sitemap.xml — карта сайта

## Что это
**Sitemap.xml** — файл, в котором перечислены все важные URL сайта с дополнительной информацией (дата обновления, частота изменений, приоритет).

Помогает поисковикам **находить и понимать структуру сайта**.

## Виды sitemap
- **Sitemap.xml** — основной.
- **Image sitemap** — для картинок.
- **Video sitemap** — для видео.
- **News sitemap** — для новостников.
- **Sitemap index** — индексный файл с ссылками на другие sitemap (для больших сайтов).

## Структура

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2025-01-15</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/about/</loc>
    <lastmod>2024-12-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

### Теги
- `<loc>` — URL страницы (обязательно, абсолютный).
- `<lastmod>` — дата последнего изменения (рекомендуется).
- `<changefreq>` — частота изменений (`always`, `hourly`, `daily`, `weekly`, `monthly`, `yearly`, `never`).
- `<priority>` — приоритет от 0.0 до 1.0 (относительно других URL вашего сайта). **Google игнорирует.**

## Лимиты
- Максимум **50 000 URL** на один sitemap.
- Максимум **50 MB** размер файла (в несжатом виде).
- Для больших сайтов — Sitemap Index.

## Sitemap Index (для больших сайтов)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://example.com/sitemap-products.xml</loc>
    <lastmod>2025-01-15</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://example.com/sitemap-blog.xml</loc>
    <lastmod>2025-01-15</lastmod>
  </sitemap>
</sitemapindex>
```

## Что включать

✅ Включать:
- Все публичные страницы, которые хотите индексировать.
- Канонические URL.
- Страницы с уникальным контентом.

❌ Не включать:
- 404 / 410 страницы.
- Редиректы (301/302).
- Canonicalized (неканонические) дубли.
- Закрытые в robots.txt.
- Noindex страницы.
- Страницы пагинации (можно, но не обязательно).
- Тонкий/некачественный контент.

## Где разместить
- `https://example.com/sitemap.xml` (стандарт).
- Можно несколько — sitemap1.xml, sitemap2.xml, ...
- Указать в **robots.txt**: `Sitemap: https://example.com/sitemap.xml`.

## Куда добавить
- **Google Search Console → Файлы Sitemap**.
- **Яндекс.Вебмастер → Файлы Sitemap**.
- **Bing Webmaster Tools**.

## Генерация
- **CMS-плагины**: Yoast, Rank Math, All in One SEO (WordPress); стандартный модуль 1С-Битрикс.
- **Сервисы**: Screaming Frog, xml-sitemaps.com.
- **Скрипты**: на Python/Node.js — обход сайта + генерация.

## Обновление
- Динамический sitemap — обновляется автоматически при изменениях.
- Для блогов — каждый новый пост добавляется.
- Для магазинов — синхронизация с каталогом.
- Google перечитывает sitemap **не мгновенно** (дни/недели).

## Частые ошибки
- ❌ URL в sitemap отдают 404.
- ❌ URL в sitemap закрыты в robots.txt.
- ❌ Содержат canonical на другой URL (путаница).
- ❌ Не обновляется (lastmod всегда одна дата).
- ❌ Указан HTTP вместо HTTPS.

## Чек-лист урока
- [ ] Sitemap.xml доступен и валиден.
- [ ] Включает только канонические публичные URL.
- [ ] Указан в robots.txt.
- [ ] Добавлен в GSC и Яндекс.Вебмастер.
- [ ] Обновляется автоматически.