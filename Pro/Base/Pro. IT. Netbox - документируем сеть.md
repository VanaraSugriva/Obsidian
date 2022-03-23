id: 202203021334
# # Netbox — документируем сеть
***Source: *** [Netbox — документируем сеть | Лаборатория сисадмина | Яндекс Дзен (yandex.ru)](https://zen.yandex.ru/media/internet_lab/netbox--dokumentiruem-set-600ebcf227add74df618264e)

## Netbox — open source web app, for manging and documenting networks. Special for sys admins.
__

Охватывает аспекты:

-   IP address management (IPAM) — IP сети и адреса, VRFs, и VLAN
-   DataCenter infrastructure management (DCIM) — организация стоечного оборудования по группам и устройствам
-   Устройства — типы устройств и место установки
-   Соединения — сеть, консоль, силовые соединения
-   Виртуализация — виртуальные машины и кластеры
-   Схемы передачи данных — схемы дальней связи и провайдеры
-   Секреты — зашифрованное хранение конфиденциальных учетных данных

Возможна интеграция с LDAP. Небольшой минус — отсутствие локализации.

Стек приложений:

-   HTTP service — nginx или Apache
-   WSGI service — gunicorn или uWSGI
-   Application — Django/Python
-   Database — PostgreSQL 9.6+
-   Task queuing — Redis/django-rq
-   Live device access — NAPALM

![[Pasted image 20220302134800.png]]

__

Keys: [[Netbox]] [[REST API]] [[LinkMeUp]] [[Automation]] [[Net]]

***Status: *** 
[[Perspective]]

***Type: *** 
[[IT]][[Job]]

***Tags: *** 
[[Pro]][[IT]][[Sobes]]

