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
|                     |                                 |                                                      |              |
# Кто решает какую версию протокола использовать?
Мультиплексирование (multiplexing) - передача нескольких потоков данных по одному каналу связи.