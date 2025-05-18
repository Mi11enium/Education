# HTTP versions
Версий было много, но они не используются.
Основные версии:

|                   | 1.1                             | 2                                                    | 3            |
| ----------------- | ------------------------------- | ---------------------------------------------------- | ------------ |
| Transport         | TCP                             | TCP                                                  | QUIC         |
| Format            | Text                            | Binary                                               | Binary       |
| Parallel requests | Sequential via 1 TCP connection | Multiplexing (parallel request via 1 TCP connection) | Multiplexing |
| Requests priority | No                              | Yes                                                  | Yes          |
