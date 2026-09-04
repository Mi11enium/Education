# Урок 19. robots.txt

## Что это
**robots.txt** — текстовый файл в корне сайта, который указывает поисковым роботам, какие страницы/разделы можно или нельзя сканировать.

`https://example.com/robots.txt`

## Не путать
- **robots.txt** — запрещает **сканирование**.
- **noindex мета-тег** — запрещает **индексирование** (но требует сканирования).
- **Canonical** — указывает **предпочтительный** URL.

Если страница закрыта в robots.txt, поисковик её НЕ увидит и не сможет проиндексировать (даже если она должна быть).

## Структура файла

```
# Комментарий (строки начинаются с #)
User-agent: *
Disallow: /admin/
Disallow: /private/
Disallow: /search?
Allow: /public/

User-agent: Yandex
Disallow: /temp/

Sitemap: https://example.com/sitemap.xml
```

### Директивы
- `User-agent` — имя робота (`*` для всех, `Googlebot`, `YandexBot`).
- `Disallow` — запретить.
- `Allow` — разрешить (внутри запрещённого раздела).
- `Sitemap` — путь к sitemap.xml (необязательно).
- `Crawl-delay` — задержка между запросами (для Яндекса, не для Google).

## Что обычно закрывают

```
Disallow: /admin/
Disallow: /wp-admin/
Disallow: /wp-includes/
Disallow: /cart/
Disallow: /checkout/
Disallow: /search/
Disallow: /?s=
Disallow: /*?sort=
Disallow: /*&sort=
Disallow: /login
Disallow: /register
Disallow: /*.pdf$
Disallow: /tmp/
Disallow: /cgi-bin/
```

## Что НЕЛЬЗЯ закрывать

❌ CSS и JS файлы — без них Google не отрендерит страницу.
```
Allow: /css/
Allow: /js/
```
❌ Страницы с контентом, которые хотите ранжировать.
❌ Sitemap.xml (но он не запрещается — просто указывайте явно).
❌ Сами по себе картинки, которые должны попасть в Google Images.

## Важные нюансы

1. **User-agent: Googlebot-Image** — отдельный бот для картинок. Закрытие в нём не закрывает от Googlebot.
2. **Регистр важен**: `/Admin/` ≠ `/admin/`.
3. **Wildcards**:
   - `*` — любой символ.
   - `$` — конец строки.
4. **Disallow: /** — запретить весь сайт (используется на dev/staging).
5. **Пустой robots.txt** или его отсутствие = разрешено всё.

## Проверка
- **Google Search Console → robots.txt Tester** (в старой версии).
- **Яндекс.Вебмастер → Анализ robots.txt**.
- **Google Search Console → Настройки → robots.txt** — показывает текущую версию.

## Частые ошибки

| Ошибка | Последствие |
|---|---|
| Закрыл весь сайт `Disallow: /` | Сайт исчез из индекса |
| Закрыл CSS/JS | Плохой рендеринг → падение позиций |
| Случайно закрыл /blog/ | Блог не индексируется |
| Закрыл пагинацию | Не индексируются страницы 2+ |
| Не обновил robots.txt при смене URL | Старые URL продолжают краулиться |

## Чек-лист урока
- [ ] robots.txt доступен по /robots.txt.
- [ ] Закрыты только ненужные разделы.
- [ ] CSS и JS не закрыты.
- [ ] Указан путь к sitemap.
- [ ] Нет опечаток и ошибок в синтаксисе.