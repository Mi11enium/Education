Виртуальный роутер
Необходим для изоляции своей лаборатории.


https://atxfiles.netgate.com/mirror/downloads/pfSense-CE-2.7.2-RELEASE-amd64.iso.gz

Распаковка архива в bash
```bash
gunzip pfSense-CE-2.7.2-RELEASE-amd64.iso.gz
```

## Установка pfSense
Установка запарная.
Используем AUTO (UFS)

## Веб-интерфейс
После установки заходим в веб-интерфейс на ВМ клиенте.
Там почти все оставляем как есть
Меняем Domain на свой
1 DNS 1.1.1.1
2 DNS 1.0.0.1



Проверка в конце установки.
![[Pasted image 20251113133304.png]]