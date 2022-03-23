id: 202203021413
# # ## АДСМ3. IPAM/DCIM-системы

**_Source:_** [АДСМ3. IPAM/DCIM-системы - linkmeup](https://linkmeup.ru/blog/918/)

## Централизованное управление IP пространством, инвентаризация, хранение и использование инфы (дизайн сети, роли устройств, производителей, LLD (адресация, маршрутизация, номера Автономных Систем)). 

__

# Содержание

-   **[Архитектура системы](https://linkmeup.ru/blog/918/#ARCHITECTURE)**
-   **[Схема данных NetBox](https://linkmeup.ru/blog/918/#SCHEME)**
    -   [DCIM](https://linkmeup.ru/blog/918/#DCIM)
        -   [Регионы и сайты (regions/sites)](https://linkmeup.ru/blog/918/#LOCATION)
        -   [Устройства](https://linkmeup.ru/blog/918/#DEVICES)
        -   [Интерфейсы](https://linkmeup.ru/blog/918/#INTERFACES)
        -   [Консольные порты](https://linkmeup.ru/blog/918/#CONSOLE)
    -   [IPAM](https://linkmeup.ru/blog/918/#IPAM)
        -   [VLAN и VRF](https://linkmeup.ru/blog/918/#VLANSVRFS)
        -   [IP-префиксы](https://linkmeup.ru/blog/918/#PREFIXES)
        -   [IP-адреса](https://linkmeup.ru/blog/918/#ADDRESSES)
    -   [Виртуализация](https://linkmeup.ru/blog/918/#VIRTUALIZATION)
    -   [Дополнительные приятные вещи](https://linkmeup.ru/blog/918/#OTHERS)
        -   [Custom fields](https://linkmeup.ru/blog/918/#CUSTOM_FIELDS)
        -   [Config Context](https://linkmeup.ru/blog/918/#CONFIG_CONTEXTS)
        -   [Теги](https://linkmeup.ru/blog/918/#TAGS)
        -   [Webhooks](https://linkmeup.ru/blog/918/#WEBHOOKS)
-   **[Заключение](https://linkmeup.ru/blog/918/#THEEND)**
-   **[Некоторые нюансы установки NetBox](https://linkmeup.ru/blog/918/#INSTALLATION)**
-   **[Немного о PostgreSQL](https://linkmeup.ru/blog/918/#POSTGRES)**
-   **[Полезные ссылки](https://linkmeup.ru/blog/918/#LINKS)**

1.  Open Source
2.  Обе необходимые части — инвентаризацию и управление.
3.  RESTful API-интерфейс.
4.  Digital Ocean (есть почти всё, что нужно, и почти ничего лишнего).
5.  Поддерживается (Slack, Github, Google-рассылки) и обновляется.
6.  Free

-   NetBox написан на Python3. Что хорошо, потому что ряд других решений написан на php и изменять их при необходимости не так уж просто.
-   Фреймворк для самого сайта — Django.
-   В качестве БД используется PostgreSQL.
-   WEB-frontend (HTTP-сервис) — NGINX — он проксирует запросы в Gunicron.
-   WSGI — Gunicorn — интерфейс между Nginx и самим приложением.
-   Фреймворк для документации по API — Swagger.
-   Чтобы демонизировать NetBox — Systemd.

NetBox отражает не реальное состояние сети, а целевое. Поэтому ничего не подгружается в NetBox из сети — это сеть настраивается в соответствие с содержимым NetBox.  
Таким образом NetBox выступает единственным источником истины (калька с single source of truth).

Задачи: управление и инвентаризация

[Функции NetBox](https://netbox.readthedocs.io/en/stable/#what-is-netbox):

-   **IP address management (IPAM)** — IP-префиксы, адреса, VRF’ы и VLAN’ы
-   **Equipment racks** — Стойки для оборудования, организованные по сайтам, группам и ролям
-   **Devices** — Устройства, их модели, роли, комплектующие и расположение
-   **Connections** — Сетевые, консольные и силовые соединения между устройствами
-   **Virtualization** — Виртуальные машины и вычислительные кластера
-   **Data circuits** — Подключения к провайдерам
-   **Secrets** — Зашифрованное хранилище учётных данных пользователей

### Регионы и сайты (regions/sites)

В парадигме NetBox устройство устанавливается на сайт, сайт принадлежит региону, регионы могут быть вложены. При этом устройство не может быть установлено просто в регионе. Если такая необходимость есть, должен быть заведён отдельный сайт.

Вот так можно вывести список всех регионов:
```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/regions/" -H "Accept: application/json; indent=4"
nb.dcim.regions.all()

```
Получить список сайтов:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/sites/" -H "Accept: application/json; indent=4"
nb.dcim.sites.all()
```

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/sites/?region=ru" -H "Accept: application/json; indent=4"
nb.dcim.sites.filter(region="ru")
```

Slug — это идентификатор, содержащий только безопасные символы: [0-9A-Za-z-_], который можно использовать в URL


### Устройства

Само устройство обладает какой-то [ролью], например, leaf, spine, edge, border.  
Оно, очевидно, является какой-то [моделью] какого-то [вендора]
Таким образом, сначала создаётся вендор, далее внутри него модели.  
[Модель] характеризуется именем, набором сервисных интерфейсов, интерфейсом удалённого управления, консольным портом и набором модулей питания.  
Помимо коммутаторов, маршрутизаторов и хостов, обладающих Ethernet-интерфейсами, можно создавать консольные сервера.

Получить список всех устройств:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/devices/" -H "Accept: application/json; indent=4"
```

```
nb.dcim.devices.all()
```

Всех устройств конкретного сайта:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/devices/?site=mlg" -H "Accept: application/json; indent=4"
```

```
nb.dcim.devices.filter(site="mlg")
```

Всех устройств определённой модели

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/devices/?model=veos" -H "Accept: application/json; indent=4"
```

```
nb.dcim.devices.filter(device_type_id=2)
```

Всех устройств определённой роли:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/devices/?role=leaf" -H "Accept: application/json; indent=4"
```

```
nb.dcim.devices.filter(role="leaf")
```

Устройство может быть в разных статусах: Active, Offline, Planned итд.  
Все активные устройства:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/devices/?status=active" -H "Accept: application/json; indent=4"
```

```
nb.dcim.devices.filter(status="active")
```

### Интерфейсы

NetBox поддерживает множество типов физических [интерфейсов](http://netbox.linkmeup.ru:45127/api/dcim/_choices/) и LAG, однако все виртуальные, такие как Vlan/IRB и loopback объединены под одним типом — Virtual.  
Каждый интерфейс привязан к какому-либо устройству.  
Интерфейсы устройств могут быть подключены друг к другу. Это будет отображаться как в интерфейсе, так и в ответах API (атрибут connected_endpoint).

Интерфейс может быть в различных режимах: Tagged или Access.  
Соответственно, в него могут быть спущены с тегом или без VLAN’ы — данного сайта или глобальные.  
Получить список всех интерфейсов устройства:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/interfaces/?device=mlg-leaf-0" -H "Accept: application/json; indent=4"
```

```
nb.dcim.interfaces.filter(device="mlg-leaf-0")
```

Получить список VLAN’ов конкретного интерфейса.

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/interfaces/?device=mlg-leaf-0&name=Ethernet7" -H "Accept: application/json; indent=4"
```

```
nb.dcim.interfaces.get(device="mlg-leaf-0", name="Ethernet7").untagged_vlan.vid
```

> Обратите внимание, что тут я уже использую метод **get** вместо **filter**. Filter возвращает список, даже если результат — один единственный объект. Get — возвращает один объект или падает с ошибкой, если результатом запроса является список объектов.  
> Поэтому get следует использовать только тогда, когда вы абсолютно уверены, что результат будет в единственном экземпляре.  
> Ещё здесь же прямо после запроса я обращаюсь к атрибутам объекта. Строго говоря, это неправильно: если по запросу ничего не найдено, то pynetbox вернёт None, а у него нет атрибута «untagged_vlan».  
> И ещё обратите внимание, что не везде pynetbox ожидает slug, где-то и name.

Выяснить к какому интерфейсу какого устройства подключен определённый интерфейс:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/interfaces/?device=mlg-leaf-0&name=Ethernet1" -H "Accept: application/json; indent=4" 
```

```
iface = nb.dcim.interfaces.get(device="mlg-leaf-0", name="Ethernet1")
iface.connected_endpoint.device
iface.connected_endpoint.name
```

Узнать имя интерфейса управления:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/interfaces/?device=mlg-leaf-0&mgmt_only=true" -H "Accept: application/json; indent=4" 
```

```
nb.dcim.interfaces.get(device="mlg-leaf-0", mgmt_only=True)
```

---

### Консольные порты

Консольные порты не являются интерфейсами, поэтому вынесены как отдельные эндпоинты.  
Порты устройства можно связать с портами консольного сервера.  
Выяснить к какому порту какого консольного сервера подключено конкретное устройство.

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/console-ports/?device=mlg-leaf-0" -H "Accept: application/json; indent=4"
```

```
nb.dcim.console_ports.get(device="mlg-leaf-0").serialize()
```

> Метод **serialize** в pynetbox позволяет преобразовать атрибуты экземпляра класса в словарь.

---

## IPAM

### VLAN и VRF

Могут быть привязаны к локации — полезно для VLAN.  
При создании VRF можно указать, допускается ли пересечение адресного пространства с другими VRF.  
Получить список всех VLAN:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/vlans/" -H "Accept: application/json; indent=4" 
```

```
nb.ipam.vlans.all()
```

Получить список всех VRF:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/vrfs/" -H "Accept: application/json; indent=4" 
```

```
nb.ipam.vrfs.all()
```

---

### IP-префиксы

Имеют иерархическую структуру. Может принадлежать какому-либо VRF (если не принадлежит — то Global).


В NetBox очень удобное визуальное представление свободных префиксов:  
![](https://fs.linkmeup.ru/images/adsm/3/available_prefixes.png) Выделить можно просто кликом на зелёную строчку.  
Может быть привязан к локации. Можно через API выделить следующий свободный под-префикс нужного размера или следующий свободный IP-адрес.  
Галочка/параметр «Is a pool» определяет, будет ли при автоматическом выделении выделяться 0-й адрес из этого префикса, или начнётся с 1-го.  
Получить список IP-префиксов сайта Малага c ролью Underlay и длиной 19:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/prefixes/?site=mlg&role=underlay&mask_length=19" -H "Accept: application/json; indent=4" 
```

```
prefix = nb.ipam.prefixes.get(site="mlg", role="underlay", mask_length="19")
```

Получить список свободных префиксов в регионе Россия c ролью Underlay:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/prefixes/40/available-prefixes/" -H "Accept: application/json; indent=4"
```

```
prefix.available_prefixes.list()
```

Выделить следующий свободный префикс длиной в 24:

```
curl -X POST "http://netbox.linkmeup.ru:45127/api/ipam/prefixes/40/available-prefixes/" \
-H "accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: TOKEN a9aae70d65c928a554f9a038b9d4703a1583594f" \
-d "{\"prefix_length\": 24}"
```

```
prefix.available_prefixes.create({"prefix_length":24})
```

> Когда внутри одного объекта нам нужно выделить какой-то дочерний, используется метод POST и нужно указать ID родительского объекта — в данном случае — **40**. Его мы выяснили вызовом из предыдущего примера.  
> В случае pynetbox мы сначала (в предыдущем примере) сохранили результат в переменную **prefix**, а далее обратились к его атрибуту **available_prefixes** и методу **create**.  
> Этот пример у вас **не сработает**, поскольку токен с правом записи уже недействителен.

---

### IP-адреса

Если есть включающий этот адрес префикс, то будут его частью. Могут быть и сами по себе.  
Могут принадлежать какому-либо VRF или быть в Global.  
Могут быть привязаны к интерфейсу, а могут висеть в воздухе.  
Можно выделить следующий свободный IP-адрес в префиксе.  
![](https://fs.linkmeup.ru/images/adsm/3/ip_addresses.png) Чтобы сделать это, просто нужно кликнуть по зелёной строчке.  
Получить список IP-адресов конкретного интерфейса:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/ip-addresses/?interface_id=8" -H "Accept: application/json; indent=4" 
```

```
nb.ipam.ip_addresses.filter(interface_id=8)
```

Или:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/ip-addresses/?device=mlg-leaf-0&interface=Ethernet1" -H "Accept: application/json; indent=4"
```

```
nb.ipam.ip_addresses.filter(device="mlg-leaf-0", interface="Ethernet1")
```

Получить список всех IP-адресов устройства:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/ip-addresses/?device=mlg-leaf-0" -H "Accept: application/json; indent=4"
```

```
nb.ipam.ip_addresses.filter(device="mlg-leaf-0")
```

Получить список доступных IP-адресов префикса:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/ipam/prefixes/28/available-ips/" -H "Accept: application/json; indent=4"
```

```
prefix = nb.ipam.prefixes.get(site="mlg", role="leaf-loopbacks")
prefix.available_ips.list()
```

> Здесь снова нужно в URL указать ID префикса, из которого выделяем адрес — на сей раз это 28.

Выделить следующий свободный IP-адрес в префиксе:

```
curl -X POST "http://netbox.linkmeup.ru:45127/api/ipam/prefixes/28/available-ips/" \
-H "accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: TOKEN a9aae70d65c928a554f9a038b9d4703a1583594f"
```

```
prefix.available_ips.create()
```

---

## Виртуализация

Мы же всё-таки боремся за звание современного ДЦ. Куда же без виртуализации.  
NetBox не выглядит и не является местом, где стоит хранить информацию о виртуальных машинах (даже о необходимости хранения в нём физических машин можно порассуждать). Однако нам это может оказаться полезным, например, можно занести информация о Route Reflector’ах, о служебных машинах, таких как NTP, Syslog, S-Flow-серверах, о машинах-управляках.  
ВМ обладает своим списком интерфейсов — они отличны от интерфейсов физических устройств и имеют свой отдельный Endpoint.  
Так можно вывести список всех виртуальных машин:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/virtualization/virtual-machines/" -H "Accept: application/json; indent=4" 
```

```
nb.virtualization.virtual_machines.all()
```

Так — всех интерфейсов всех ВМ:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/virtualization/interfaces/" -H "Accept: application/json; indent=4" 
```

```
nb.virtualization.interfaces.all()
```

Для ВМ нельзя указать конкретный гипервизор/физическую машину, на котором она запущена, но можно указать кластер. Хотя не всё так безнадёжно. Читаем дальше.

---

## Дополнительные приятные вещи

Основная функциональность NetBox закрывает большинство задач многих пользователей, но не все. Всё-таки изначально продукт написан для решения задач конкретной компании. Однако он активно развивается и новые релизы выходят довольно [часто](https://github.com/netbox-community/netbox/releases). Соответственно появляются и новые функции.  
Так, например, с моей первой установки NetBox пару лет назад в нём появились теги, config contexts, webhooks, кэширование, supervisord сменился на systemd, внешние хранилища для файлов.  
следите.  

### Custom fields

Иногда хочется к какой-либо сущности добавить поле, в которое можно было бы поместить произвольные данные.  
Например, указать номер договора поставки, по которому был приобретён коммутатор или имя физической машины, на которой запущена ВМ.  
Тут на помощь и приходит custom fields — как раз такое поле с текстовым значением, которое можно добавить почти к любой сущности в NetBox.  
Создаётся Custom fields в админской панели

Вот так это выглядит при редактировании устройства, для которого был создан custom field:  
![](https://fs.linkmeup.ru/images/adsm/3/nb_custom_field_edit.png) Запросить список устройств по значению custom_field

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/devices/?cf_contract_number=0123456789" -H "Accept: application/json; indent=4"
```

```
nb.dcim.devices.filter(cf_contract_number="0123456789")
```

---

### Config Context

Иногда хочется чего-то большего, чем неструктурированный текст. Тогда на помощь приходит [Config Context](http://netbox.linkmeup.ru:45127/extras/config-contexts/1/).  
Это возможность ввести набор структурированных данных в формате JSON, который больше некуда поместить.  
Это может быть, например, набор BGP communities или список Syslog-серверов.  
Config Context может быть локальным — настроенным для конкретного объекта — или глобальным, когда он настраивается однажды, а затем распространяется на все объекты, удовлетворяющие определённым условиям (например, расположенные на одном сайте, или запущенные на одной платформе).  
![](https://fs.linkmeup.ru/images/adsm/3/config_context.png) Config Context автоматически добавляется к результатам запроса. При этом локальные и глобальные контексты сливаются в один.  
Например, для устройства just a simple russian girl, для которого есть локальный контекст, в выводе будет ключ «config_context»:

```
curl -X GET "http://netbox.linkmeup.ru:45127/api/dcim/devices/?q=russian" -H "Accept: application/json; indent=4"
```

![](https://fs.linkmeup.ru/images/adsm/3/config_context_result.png)

---

### Теги

Про теги сложно сказать что-то новое. Они есть. Они удобны для добавления какого-либо признака. К примеру, можно пометить тегом «бяда» коммутаторы из партии, в которой сбоит память.

---

### Webhooks

Незаменимая вещь, когда нужно, чтобы об изменениях в NetBox’е узнавали другие сервисы.  
Например, при заведении нового коммутатора отправляется хука в систему автоматизации, которая запускает процесс настройки устройства и ввода в эксплуатацию.

---

# Заключение

В этой статье я не преследую цель рассмотреть все возможности NetBox, поэтому всё остальное отдаю вам на откуп. Разбирайтесь, пробуйте.  
Далее в рамках построения системы автоматизации я буду касаться только тех частей, которые нам действительно нужны.  
Итак, выше я коротко рассказал о том, что из себя представляет NetBox, и как в нём хранятся данные.  
Повторюсь, что почти все необходимые данные я туда уже внёс, и вы можете утащить себе [дамп БД](https://github.com/eucariot/ADSM/blob/master/docs/source/3_ipam/netbox_initial_db.sql).  
Всё готово к следующему этапу автоматизации: написанию системы (ахаха, просто скриптов) инициализации устройств и управления конфигурацией.

---

Но, прежде чем закончить статью я ещё скажу пару слов об установке и работе компонентов NetBox.  

# Некоторые нюансы установки NetBox

Я не буду описывать процесс инсталляции в деталях — он более чем классно описан в [официальной документации](https://netbox.readthedocs.io/en/stable/installation/).  
Посмотреть на процесс запуска docker-образа NetBox и работу в GUI можно в видео Димы Фиголя ([раз](https://www.youtube.com/watch?v=GGXgAlWm9aY&t=9655s) и [два](https://www.youtube.com/watch?v=a3yK_WAisPw)) и [Эмиля Гарипова](https://www.youtube.com/watch?v=I_Ra3PIR2Lc&feature=youtu.be).  
В целом, если следовать шагам установки/запуска неукоснительно, то всё получится.  
Но вот какие есть нюансы, про которые случайно можно забыть.

-   В файле configuration.py должен быть заполнен параметр [ALLOWED_HOSTS](https://netbox.readthedocs.io/en/stable/installation/2-netbox/#allowed_hosts):
    
    ```
    ALLOWED_HOSTS = ['netbox.linkmeup.ru', 'localhost']
    ```
    
    Тут нужно указать все возможные имена NetBox, к которым вы будете обращаться, например, может быть внешний IP-адрес или 127.0.0.1 или DNS-alias.  
    Если этого не будет сделано, сайт NetBox не откроется и будет показывать 400.
    
-   В этом же файле должен быть указан [SECRET_KEY](https://netbox.readthedocs.io/en/stable/installation/2-netbox/#secret_key), который можно выдумать самому или сгенерировать скриптом.
-   Главная страница будет показывать 502 Bad Gateway, если что-то не так с настройкой базы PostgreSQL: проверьте хост(если ставили на другую машину), порт, имя базы, имя пользователя, пароль.
-   С [некоторых пор](https://github.com/netbox-community/netbox/releases/tag/v2.6.0) NetBox по умолчанию не даёт никаких прав на чтение без авторизации.  
    Изменяется это всё в том же configuration.py:
    
    ```
    EXEMPT_VIEW_PERMISSIONS = ['*']
    ```
    
-   А ещё API запросы будут возвращать 200 и не работать, если в API URL не будет слэша в конце.
    
    ```
    curl -X GET -H "Accept: application/json; indent=4" "http://netbox.linkmeup.ru:45127/api/dcim/devices"
    ```
    

---

# Немного о PostgreSQL

Для подключения к серверу:

```
psql -U username -h hostname db_name
```

Например:

```
psql -U netbox -h localhost netbox
```

Для вывода всех таблиц:

```
/dt
```

Для выхода:

```
/q
```

Для дампа БД:

```
pg_dump -U username -h hostname db_name > netbox.sql
```

Если не хочется каждый раз вводить пароль:

```
echo *:*:*:username:password > ~/.pgpass
chmod 600 ~/.pgpass
```

Если у вас есть своя инсталляция и не хочется вносить всё руками, можно просто сделать так, взяв дамп текущей БД NetBox [тут](https://github.com/eucariot/ADSM/blob/master/docs/source/3_ipam/netbox_initial_db.sql):

```
psql -U username -h hostname db_name < netbox_initial_db.sql
```

Если предварительно нужно дропнуть все таблицы (а сделать это придётся), то можно подготовить заранее файл:

```
psql -U username -h hostname db_name
\o drop_all_tables.sql
select 'drop table ' || tablename || ' cascade;' from pg_tables;
\q
psql -U username -h hostname db_name -f drop_all_tables.sql
```












__

Keys: 

**_Status:_**  


**_Type:_**  


**_Tags:_**  

# Полезные ссылки

-   [Сам NetBox на guthub](https://github.com/netbox-community/netbox)
-   [Контейнерная версия](https://github.com/netbox-community/netbox-docker)
-   [Полная инструкция по установке и вся документация по продукту](https://netbox.readthedocs.io/en/stable/)
-   [Documenting your network infrastructure in NetBox, integrating with Ansible over REST API](http://karneliuk.com/2019/04/documenting-your-network-infrastructure-in-netbox-integrating-with-ansible-over-rest-api-and-automating-provisioning-of-cumulus-linux-arista-eos-nokia-sr-os-and-cisco-ios-xr/)
-   [IPAM NetBox and its API, Docker, Postman](https://www.youtube.com/watch?v=GGXgAlWm9aY&t=9655s)
-   [IPAM NetBox and its API, configuration templates with Python](https://www.youtube.com/watch?v=a3yK_WAisPw)
-   [SDK для работы с NetBox в Python](https://github.com/digitalocean/pynetbox)
-   [Документация по API](http://netbox.linkmeup.ru:45127/api/docs/)
-   [Mailing list](https://groups.google.com/forum/#!forum/netbox-discuss)
-   [Slack-канал NetworkToCode](https://networktocode.slack.com/)
-   [Что такое REST](https://linkmeup.ru/blog/530.html)
-   [Инсталляция NetBox для нужд АДСМ](http://netbox.linkmeup.ru:45127/)