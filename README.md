## OTUS Administrator Linux. Professional ДЗ №19: Фильтрация трафика - iptables

**Задание**

1. реализовать knocking port
2. centralRouter может попасть на ssh inetrRouter через knock скрипт (пример: https://wiki.archlinux.org/title/Port_knocking)
3. добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост.
4. запустить nginx на centralServer.
5. пробросить 80й порт на inetRouter2 8080.
6. дефолт в инет оставить через inetRouter.
7. (\*) реализовать проход на 80й порт без маскарадинга

Формат сдачи ДЗ - vagrant + ansible

**_Решение_**
