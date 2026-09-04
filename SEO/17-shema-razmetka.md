# Урок 17. Schema.org и структурированные данные

## Что это
**Schema.org** — стандарт семантической разметки данных, понятный поисковым системам. JSON-LD — рекомендуемый формат.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Обзор iPhone 15",
  "author": {"@type": "Person", "name": "Иван Иванов"},
  "datePublished": "2024-09-20"
}
</script>
```

## Зачем нужна Schema
1. **Rich snippets** — расширенные сниппеты (рейтинги, цены, FAQ, рецепты).
2. **Повышение CTR** — заметнее в выдаче.
3. **Помощь ПС** — однозначное понимание сущностей.
4. **Голосовой поиск и AI** — структурированные данные попадают в AI Overviews.

## Основные типы Schema

### Контент
- `Article` — статьи, новости.
- `BlogPosting` — посты блога.
- `NewsArticle` — новости.
- `FAQPage` — FAQ.
- `HowTo` — инструкции.
- `QAPage` — страницы вопрос-ответ.

### Коммерция
- `Product` — карточка товара.
- `Offer` — предложение.
- `AggregateRating` — агрегированный рейтинг.
- `Review` — отзыв.
- `BreadcrumbList` — хлебные крошки.
- `ItemList` — список товаров.

### Бизнес
- `Organization` — компания.
- `LocalBusiness` — локальный бизнес.
- `Person` — автор.
- `ContactPoint` — контакты.

### Медиа
- `ImageObject` — изображение.
- `VideoObject` — видео.
- `Recipe` — рецепты.

## Примеры разметки

### Article
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Как выбрать ноутбук в 2025",
  "image": "https://example.com/img.jpg",
  "datePublished": "2025-01-15",
  "author": {
    "@type": "Person",
    "name": "Иван Петров",
    "url": "https://example.com/authors/ivan"
  },
  "publisher": {
    "@type": "Organization",
    "name": "SiteName",
    "logo": {"@type": "ImageObject", "url": "https://example.com/logo.png"}
  }
}
```

### FAQPage
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "Что такое SEO?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "SEO — это оптимизация сайта для поисковых систем."
      }
    }
  ]
}
```

### Product
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "iPhone 15 Pro",
  "image": "https://example.com/iphone.jpg",
  "description": "Смартфон Apple iPhone 15 Pro 256GB",
  "sku": "IP15-PRO-256",
  "brand": {"@type": "Brand", "name": "Apple"},
  "offers": {
    "@type": "Offer",
    "price": "119990",
    "priceCurrency": "RUB",
    "availability": "https://schema.org/InStock"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "reviewCount": "1523"
  }
}
```

## Как внедрять

1. **Вручную** — JSON-LD в `<head>` или перед `</body>`.
2. **Плагины CMS** — Yoast, Rank Math (WordPress), Bitrix SEO-модуль.
3. **Google Tag Manager** — динамическая вставка.
4. **Schema App, Merkle** — генераторы.

## Проверка
- **Google Rich Results Test**: https://search.google.com/test/rich-results
- **Schema.org Validator**: https://validator.schema.org/
- **Google Search Console → Улучшения** — отчёт по расширенным результатам.

## Распространённые ошибки
- ❌ Разметка не соответствует видимому контенту.
- ❌ Невалидный JSON.
- ❌ Устаревшие форматы (Microdata, RDFa — Google рекомендует JSON-LD).
- ❌ Пометка контента Review без настоящих отзывов.

## Чек-лист урока
- [ ] На каждой странице есть релевантная Schema-разметка.
- [ ] Формат — JSON-LD.
- [ ] Разметка проходит валидацию Google.
- [ ] Контент разметки виден пользователю.