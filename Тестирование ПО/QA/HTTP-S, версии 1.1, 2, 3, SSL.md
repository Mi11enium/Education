# HTTP versions
Версий было много, но они не используются.
Основные версии:

|                     | 1.1                             | 2                                                    | 3            |
| ------------------- | ------------------------------- | ---------------------------------------------------- | ------------ |
| Transport           | TCP                             | TCP                                                  | QUIC         |
| Format              | Text                            | Binary                                               | Binary       |
| Parallel requests   | Sequential via 1 TCP connection | Multiplexing (parallel request via 1 TCP connection) | Multiplexing |
| Requests priority   | No                              | Yes                                                  | Yes          |
| Headers compression | No                              | Yes                                                  | Yes          |
| Server push         | No                              | Yes                                                  | Yes          |
## Кто решает какую версию протокола использовать?

Клиент (браузер) и сервер договариваются между собой. Это происходит во время
handshake шифрования (TLS).

- TLS
- HTTP/3: TLS + alt-svc header (заголовок  с инфоверсией протокола от сервера)

Мультиплексирование (multiplexing) - передача нескольких потоков данных по одному каналу связи.

Header
Connection: keep-alive - поддержка соединения после окончания запроса
Connection: close - разрыв соединения после окончания запроса

## Сколько параллельных запросов может отправить браузер на один домен?

Зависит от версии HTTP и реализации браузера.
v. 1.1 - 6-8 запросов
v2 - ~100 запросов (ограничение браузеров)
v3 

# HTTP + TLS = HTTPS (Secure HTTP)
TLS - Transport Layer Security - протокол защиты транспортного уровня

HTTPS 1.1: HTTP + TLS (optional) + TCP
HTTPS 2: HTTP + TLS + TCP
HTTPS 3: HTTP + TLS + QUIC

# SSL сертификаты
Что такое SSL - сертификат?
это цифровой документ, который подтверждает подлинность сайта и шифрует данные между клиентом и сервером

Кем выдается?
сертификаты выпускают центры сертификации (Certificate Authorities, CA) - доверенные организации
- Comodo
- DigiCert
- Let's Encrypt
- GlobalSign
- GoDaddy
Они проверяют право владельца на домен и подписывают сертификат своим цифровым ключом

Как используется?
1. Шифрование данных - сертификат создает защищенное соединение HTTPS
2. Аутентификация сайта - подтверждает что пользователь подключен к настоящему сайту
3. Доверие и SEO - сайты без HTTPS очень сильно падают в поисковых системах

## Типы SSL - сертификатов
|Тип|Проверка|Использование|Примеры|
|---|---|---|---|
|**DV (Domain Validated)**|Только домен|Блоги, личные сайты|Let's Encrypt, Comodo DV|
|**OV (Organization Validated)**|Домен + организация|Корпоративные сайты, госучреждения|DigiCert OV, Sectigo OV|
|**EV (Extended Validation)**|Углубленная проверка|Банки, интернет-магазины|GlobalSign EV, Thawte EV|
|**Wildcard**|DV/OV/EV + поддомены|Сайты с поддоменами (`shop.site.com`)|GoDaddy Wildcard|
|**Multi-Domain (SAN)**|Несколько доменов|Несколько сайтов в одном сертификате|DigiCert Multi-Domain|
