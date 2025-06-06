
# Internet
Что такое интернет?
<mark style="background: #FF5582A6;">Это сеть сетей, которая связывает между собой множество устройств, посредством проводного и беспроводного соединения для обмена данными.</mark>
# IP (Internet Protocol)
<mark style="background: #FF5582A6;">IP - уникальный числовой идентификатор устройства в сети интернет</mark>
IP - адрес выдается провайдером и является уникальным в сети в текущий момент времени.
Может быть статическим или динамическим.


У провайдера есть зарезервированный список IP-адресов, которые он может выдавать пользователям. <mark style="background: #ADCCFFA6;">Знание IP-адреса необходимо для подключения одного устройства к другому в сети интернет.</mark>

В свою очередь эти IP-адреса провайдер согласовывает с вышестоящим провайдером (магистральным).

Бывает двух версий:
IPv4
например, `192.168.1.1` (4 числа от 0 до 255, разделённые точками)
и
IPv6
например, `2001:0db8:85a3::8a2e:0370:7334` (шестнадцатеричный формат)

# DNS (Domain Name System)
<mark style="background: #FF5582A6;">DNS - это система доменных имен, которая преобразует названия сайтов (google.com)
в машиночитаемые IP-адреса</mark> (142.250.15.2)

### Как работает DNS?

1. **Пользователь вводит адрес** (например, `youtube.com`) в браузере.
2. **Браузер запрашивает DNS-сервер** (обычно предоставляемый провайдером или публичный, как `8.8.8.8` от Google).
3. **DNS-сервер ищет IP-адрес** этого домена в своей базе или запрашивает у других серверов
4. **Возвращается IP-адрес**, и браузер загружает сайт.

### Основные компоненты DNS:

- **Доменное имя** (например, `example.com`) – удобный для запоминания адрес.
- **DNS-серверы** – хранят записи о соответствии доменов и IP.
- **Записи DNS** (типы):
    - **A** – связывает домен с IPv4.
    - **AAAA** – связывает домен с IPv6.
    - **CNAME** – перенаправляет на другой домен (например, `www.example.com` → `example.com`).
    - **MX** – указывает почтовый сервер для домена.
    - **TXT** – содержит текстовые данные (например, для проверки владения доменом).

# Domain
Домен
<mark style="background: #FF5582A6;">Это текстовая замена числового IP-адреса.</mark>
### **Из чего состоит домен?**
Домен строится по иерархии справа налево, через точки:
subdomain.second-level-domain.top-level-domain.root

**Пример для `shop.example.com`:**

1. **`.com`** — домен верхнего уровня (TLD). 
2. **`example`** — домен второго уровня (ваше уникальное имя).
3. **`shop`** — поддомен (третий уровень).

Чем больше точек, тем «глубже» уровень:
- `example.com` — 2-й уровень.
- `blog.example.com` — 3-й уровень.
- `dev.api.example.com` — 4-й уровень.

Первым уровнем является корневой домен в виде точки. (com.)

# URL
URL
<mark style="background: #FF5582A6;">Стандартный способ указания адреса ресурса в интернете.</mark>
### **Из чего состоит URL?**
Пример:  
`https://www.example.com:443/path/to/page?query=value#section`

| Часть URL            | Описание                                                              | Пример из URL выше                  |
| -------------------- | --------------------------------------------------------------------- | ----------------------------------- |
| **Схема (Protocol)** | Протокол передачи данных (HTTP, HTTPS, FTP и др.)                     | `https://`                          |
| **Домен (Host)**     | Адрес сервера (может включать поддомен)                               | `www.example.com`                   |
| **Порт (Port)**      | Номер порта (необязательно, по умолчанию: 80 для HTTP, 443 для HTTPS) | `:443` (скрыт, так как стандартный) |
| **Путь (Path)**      | Путь к конкретной странице или файлу на сервере                       | `/path/to/page`                     |
| **Запрос (Query)**   | Параметры запроса (начинаются с `?`, разделяются `&`)                 | `?query=value`                      |
| **Якорь (Fragment)** | Ссылка на конкретную часть страницы (начинается с `#`)                | `#section`                          |

## URI URL URN

<iframe width="560" height="315" src="https://www.youtube.com/embed/AdaXK48broc?si=9ahjQVpQcdKlomdU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>