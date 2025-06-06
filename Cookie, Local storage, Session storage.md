# Cookie
это небольшой фрагмент данных, отправленный веб-сервером и хранимый на компьютере пользователя.

Что может хранить?
1.  управление сеансом (токен авторизации)
2. персонализация (язык)
3. трекинг (страницы, клики) (отслеживание сайтом ваших действий)

## Как приходят в header
Set-Cookie, Cookie headers

## Атрибуты cookie
1. domain=domain.com
2. path=/
3. expires, max-age
4. secure
5. httpOnly (data from server only)
6. SameSite

Как сделать так, чтобы Javascript код на клиенте не смог прочитать cookie?
Должен стоять атрибут httpOnly

Как сделать так, чтобы cookie передавались только по HTTPS?
Должен быть атрибут secure

Может ли в значении cookie содержаться пробел?
?

# Local storage
это хранилище данных в формате ключ/значение без срока давности.
компилируется браузером.



# Session storage
это хранилище данных в формате ключ/значение существующее в рамках текущей вкладки

# Отличия
Кто является поставщиком данных?
Cookie - сервер
Local storage - браузер


Время экспирации
Cookie - имеет срок истечения
Local storage - не имеет срока истечения
Session storage - до закрытия вкладки


|                    | Cookie          | Local storage | Session storage  |
| ------------------ | --------------- | ------------- | ---------------- |
| Контроль           | Сервер/браузер  | Браузер       | Браузер          |
| TTL (срок жизни)   | Контролируемо   | Вечно         | Закрытие вкладки |
| Доступно из        | Любой вкладки   | Любой вкладки | Одной вкладки    |
| Субдомены          | Контролируемо   | Нет           | Нет              |
| Просмотр           | document.cookie | localStorage  | sessionStorage   |
| Отправка на сервер | Да              | Контролируемо | Контролируемо    |
| Размер             | 4 KB            | <5-10 MB?     | <5 MB?           |
| Тип данных         | Строка          | Строка        | Строка           |