# master2025

# Базовая настройка (имена устройств)

### Задание:

a) Настройте имена устройств согласно топологии

- 1. Используйте полное доменное имя

### Вариант реализации:

### Для устройств на базе ОС "Альт Сервер":

### SW1-HQ | SW2-HQ | SW3-HQ | SRV1-HQ | SRV1-DT | SRV2-DT | SRV3-DT:

- Настраиваем имена устройств согласно топологии, используя полное доменное имя
    - в качестве доменного имени используется - **au.team**

```
hostnamectl set-hostname <ИМЯ_ВМ>.<ДОМЕННОЕ_ИМЯ>; exec bash
```

где:

- **hostnamectl** — утилита для управления именем машины;
- **set-hostname** — аргумент, позволяющий выполнить изменение хостнейма;
- **exec bash** — перезапуск оболочки bash для отображения нового хостнейма.
    - Например:
        - Для ВМ **SW1-HQ**:

```
hostnamectl set-hostname sw1-hq.au.team; exec bash
```

- - - Для ВМ **SRV1-HQ**:

```
hostnamectl set-hostname srv1-hq.au.team; exec bash
```

- - - Для ВМ **SRV1-DT**:

```
hostnamectl set-hostname srv1-dt.au.team; exec bash
```

- Проверяем:
    - Для проверки полного доменного имени используем утилиту **hostname** и добавляем ключ **-f**:

```
hostname -f
```

- - - Результат:

![](https://sysahelper.ru/pluginfile.php/812/mod_page/content/3/image.png)

Аналогично и для всех остальных устройств: SW2-HQ, SW3-HQ, SRV1-HQ, SRV1-DT, SRV2-DT, SRV3-DT.

### Для устройств на базе ОС "Альт Рабочая станция":

### CLI-HQ | ADMIN-HQ | CLI-DT | ADMIN-DT | CLI:

- Настраиваем имена устройств согласно топологии, используя полное доменное имя
    - в качестве доменного имени используется - **au.team**
    - поскольку клиент имеет графический интерфейс - воспользуемся Центром Управления Системой (**ЦУС**):


- После чего необходимо выполнить **ПЕРЕЗАГРУЗКУ** виртуальной машины
    - Результат:

Аналогично и для всех остальных устройств: ADMIN-HQ, CLI-DT, ADMIN-DT, CLI.

_P.S. на **CLI** нет необходимости задавать полное доменное имя_

### Для устройств на базе "EcoRouter":

### R-HQ | R-DT:

- Настраиваем имена устройств согласно топологии, используя полное доменное имя
    - в качестве доменного имени используется - **au.team**
    - выполняем первый вход на устройство из под пользователя по умолчанию:
        - логин: **admin**
        - пароль: **admin**

**![](https://sysahelper.ru/pluginfile.php/812/mod_page/content/3/image%20%286%29.png)**

- Переходим в привелегированный режим (**enable**) и в режим администрирования (**configure**), затем назначаем имя устройства и доменное имя:

```
ecorouter>enable
ecorouter#configure
ecorouter(config)#hostname r-hq
r-hq(config)#ip domain-name au.team
r-hq(config)#write
Building configuration...
```

Аналогично и для: R-DT

#### _Для FW-DT имя будет задано позднее, после назначения IP-адреса на локальный интерфейс и доступа к веб-интерфейсу_

# Базовая настройка (адреса устройств)


b) Сконфигурируйте адреса устройств на свое усмотрение.

- 1. Для офиса HQ выделена сеть 192.168.11.0/24
- 2. Для офиса DT выделена сеть 192.168.33.0/24
- 3. Для туннелей между офисами выделена сеть 10.10.10.0/24
    - i. Туннель должен вмещать минимально возможное количество адресов
- 4. Данные сети необходимо разделить на подсети для каждого vlan.
    - i. VLAN110 должна вмещать не более 64 адресов
    - ii. VLAN220 должна вмещать не более 16 адресов
    - iii. VLAN330 должна вмещать не более 8 адресов

### Вариант реализации:

#### Разбиение сетей на подсети:

|   |   |   |   |   |
|---|---|---|---|---|
|**Название устройства**|**NIC**|**Сеть (подсеть)**|**IP-адрес**|**Шлюз**|
|R-DT|ISP <-> R-DT|172.16.4.0/28|172.16.4.14/28|172.16.4.1|
|R-DT <-> FW-DT|192.168.33.88/30|192.168.33.89/30|-|
|Tunnel (GRE)|10.10.10.0/30|10.10.10.1/30|-|
|FW-DT|R-DT <-> FW-DT|192.168.33.88/30|192.168.33.90/30|192.168.33.89|
|SW-DT (VLAN110)|192.168.33.0/26|192.168.33.1/26|-|
|SW-DT (VLAN220)|192.168.33.64/28|192.168.33.65/28|
|SW-DT (VLAN330)|192.168.33.80/29|192.168.33.81/29|
|ADMIN-DT|SW-DT (VLAN330)|192.168.33.80/29|192.168.33.82/29|192.168.33.81|
|SRV1-DT|SW-DT (VLAN220)|192.168.33.64/28|192.168.33.66/28|192.168.33.65|
|SRV2-DT|SW-DT (VLAN220)|192.168.33.64/28|192.168.33.67/28|192.168.33.65|
|SRV3-DT|SW-DT (VLAN220)|192.168.33.64/28|192.168.33.68/28|192.168.33.65|
|WireGuard|10.6.6.0/24|10.6.6.1/24|-|
|CLI-DT|SW-DT (VLAN110)|192.168.33.0/26|DHCP|   |
|R-HQ|ISP <-> R-HQ|172.16.5.0/28|172.16.5.14/28|172.16.5.1|
|R-HQ <-> SW1-HQ (VLAN110)|192.168.11.0/26|192.168.11.1/26|-|
|R-HQ <-> SW1-HQ (VLAN220)|192.168.11.64/28|192.168.11.65/28|
|R-HQ <-> SW1-HQ (VLAN330)|192.168.11.80/29|192.168.11.81/29|
|Tunnel (GRE)|10.10.10.0/30|10.10.10.2/30|-|
|SW1-HQ|ovs-internal (VLAN330)|192.168.11.80/29|192.168.11.82/29|192.168.11.81|
|SW2-HQ|ovs-internal (VLAN330)|192.168.11.80/29|192.168.11.83/29|192.168.11.81|
|SW3-HQ|ovs-internal (VLAN330)|192.168.11.80/29|192.168.11.84/29|192.168.11.81|
|ADMIN-HQ|SW3-HQ <-> ADMIN-HQ (VLAN330)|192.168.11.80/29|192.168.11.85/29|192.168.11.81|
|SRV1-HQ|SW2-HQ <-> SRV1-HQ (VLAN220)|192.168.11.64/28|192.168.11.66/28|192.168.11.65|
|CLI-HQ|SW2-HQ <-> CLI-HQ (VLAN110)|192.168.11.0/26|DHCP|   |
|CLI|ISP <-> CLI|172.16.3.1/28|172.16.3.14/28|172.16.3.1|
|WireGuard|10.6.6.0/24|10.6.6.2/24|-|

### R-HQ:

- Смотрим наименование **порто****в** в системе (из привилегированного режима):

```
r-hq#show port brief
```

- - Результат:
        - порт **te0** - смотрит в сторону **ISP**;
        - порт **te1** - смотрит в сторону локальной сети офиса **HQ**.

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image.png)

- Также разберёмся с основными понятиями касающимися **EcoRouter**:
    - Порт (**port**) – это устройство в составе **EcoRouter**, которое работает на уровне коммутации (L2);
    - Интерфейс (**interface**) – это логический интерфейс для адресации, работает на сетевом уровне (L3);
    - **Service instance** (Сабинтерфейс, SI, Сервисный интерфейс) является логическим сабинтерфейсом, работающим между L2 и L3 уровнями:
        - Данный вид интерфейса необходим для соединения физического порта с интерфейсами L3, интерфейсами bridge, портами;
        - Используется для гибкого управления трафиком на основании наличия меток VLANов в фреймах, или их отсутствия;
        - Сквозь сервисный интерфейс проходит весь трафик, приходящий на порт.
- Таким образом, для того чтобы назначить IPv4-адрес на EcoRouter - необходимо придерживаться следующего алгоритма в общем виде:
    - Создать интерфейс с произвольным именем и назначить на него IPv4;
    - В режиме конфигурирования порта - создать service-instance с произвольным именем:
        - указать (инкапсулировать) что будет обрабатываться тегированный или не тегированный трафик;
        - указать в какой интерфейс (ранее созданный) нужно отправить обработанные кадры.

- Создаём интерфейсы (подинтерфейсы/sub-interfaces) для назначения адресов локальных подсетей офиса **HQ** для дальнейшей маршрутизации между VLAN-ами:
    - первым делом создаём интерфейсы для каждого VLAN-а:

```
r-hq#configure terminal
r-hq(config)#interface vl110
r-hq(config-if)#description "Clients"
r-hq(config-if)#ip address 192.168.11.1/26
r-hq(config-if)#exit
r-hq(config)#
r-hq(config)#interface vl220
r-hq(config-if)#description "Servers"
r-hq(config-if)#ip address 192.168.11.65/28
r-hq(config-if)#exit
r-hq(config)#
r-hq(config)#interface vl330
r-hq(config-if)#description "Administrators"
r-hq(config-if)#ip address 192.168.11.81/29
r-hq(config-if)#exit
r-hq(config)#
r-hq(config)#write
```

- - На базе физического интерфейса **te1** - для каждого VLAN-а создаём **service-instance** с инкапсуляцией соответствующих тегов (VID) и подключением необходимых интерфейсов:

```
r-hq(config)#port te1
r-hq(config-port)#service-instance te1/vl110
r-hq(config-service-instance)#encapsulation dot1q 110 exact
r-hq(config-service-instance)#rewrite pop 1
r-hq(config-service-instance)#connect ip interface vl110
r-hq(config-service-instance)#exit
r-hq(config-port)#
r-hq(config-port)#service-instance te1/vl220
r-hq(config-service-instance)#encapsulation dot1q 220 exact
r-hq(config-service-instance)#rewrite pop 1
r-hq(config-service-instance)#connect ip interface vl220
r-hq(config-service-instance)#exit
r-hq(config-port)#
r-hq(config-port)#service-instance te1/vl330
r-hq(config-service-instance)#encapsulation dot1q 330 exact
r-hq(config-service-instance)#rewrite pop 1
r-hq(config-service-instance)#connect ip interface vl330
r-hq(config-service-instance)#exit
r-hq(config-port)#
r-hq(config-port)#write
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%281%29.png)

### R-DT:

- Смотрим наименование **порто****в** в системе (из привилегированного режима):

```
r-dt#show port brief
```

- - Результат:
        - порт **te0** - смотрит в сторону **ISP**;
        - порт **te1** - смотрит в сторону локальной сети **DT** - непосредственно к **FW-DT**.

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%282%29.png)

- На текущий момент выполняем создание **интерфейса** с именем **int1** с последующим назначаем IPv4-адреса (из подсети, которая не задействована при разделение на необходимые подсети по требованиям задания) согласно таблице адресации в сторону **FW-DT**:
    - также при необходимости даём понятное описание для интерфейса (**description**);

```
r-dt#configure terminal
r-dt(config)#interface int1
r-dt(config-if)#description "Connect_FW-DT"
r-dt(config-if)#ip address 192.168.33.89/30
r-dt(config-if)#exit
r-dt(config)#
```

- - переходим в режим конфигурирования порта **te1** - создаём **service-instance** с именем **te1/int1:**
        - также указываем что будет обрабатывать не тегированный трафик (**untagged**);
        - и указываем в какой интерфейс нужно отправить обработанные кадры (**int1**);

```
r-dt(config)#port te1
r-dt(config-port)#service-instance te1/int1
r-dt(config-service-instance)#encapsulation untagged
r-dt(config-service-instance)#connect ip interface int1
r-dt(config-service-instance)#exit
r-dt(config-port)#write
r-dt(config-port)#
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%283%29.png)

### SRV1-HQ:

- В качестве сетевой подсистемы будет использоваться [Etcnet](https://www.altlinux.org/Etcnet)

- Для того, чтобы в качестве сетевой подсистемы корректно использовался **etcnet**, в основном конфигурационной файле для интерфейса **/etc/net/ifaces/<INTERFACE_NAME>/options** должны присутствовать два параметра соследующими значениями:
    - **DISABLED=no**
    - **NM_CONTROLLED=no**
        - в данном случае сетевой интерфейс имеет имя **ens19**;
        - также стоит обратить внимание, чтобы назначить статические сетевые параметры для данного интерфейса -параметр **BOOTPROTO** долже иметь значение **static**

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%284%29.png)

- Далеее назначем **IP-адрес** на интерфейс согласно таблице адресации и **IP-адрес шлюза по умолчанию**:
    - Назначаем **IP-адрес** из **vlan220** (серверная подсеть) на интерфейс **ens19** (в данном случае):

```
echo "192.168.11.66/28" > /etc/net/ifaces/ens19/ipv4address
```

- - Назначаем **IP-адрес шлюза** по умолчанию для **vlan220** (серверная подсеть):

```
echo "default via 192.168.11.65" > /etc/net/ifaces/ens19/ipv4route
```

- - Для применения внесённых изменений - необходимо перезагрузить службу **network**:

```
systemctl restart network
```

- Проверяем:
    - Наличие IPv4-адреса и адреса шлюза по умолчанию:

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%285%29.png)

_На данный момент - доступность шлюза по умолчанию не проверить, т.к. не реализована коммутация._

### SRV1-DT | SRV2-DT | SRV3-DT:

- Настраивается аналогично **SRV1-HQ**, таким образом, ожидаемый результат долже получиться следующий:
    - **SRV1-DT:**

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%286%29.png)

- - **SRV2-DT:**

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%287%29.png)

- - **SRV3-DT:**

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%288%29.png)

### ADMIN-HQ:

- Пользуясь средствами графического интерфейса - назначаем **IP-адрес** на интерфейс согласно таблице адресации и **IP-адрес шлюза по умолчанию**
    - переходим в **Альтератор** (Центр Управления Системой - ЦУС) - там же где назначали имя, задаём необходимые сетевые параметры:

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%289%29.png)

- - Результат:

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%2810%29.png)

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%2811%29.png)

### ADMIN-DT:

- Настраивается аналогично **ADMIN-HQ**
    - таким образом, ожидаемый результат должен получиться следующий:

 ![](https://sysahelper.ru/pluginfile.php/813/mod_page/content/4/image%20%2812%29.png)

#### _Для FW-DT см. в разделе [Настройка FW-DT (Ideco NGFW) назначение IP-адресов на локальные интерфейсы](https://sysahelper.ru/mod/page/view.php?id=500&forceview=1), но [после Настройка FW-DT (Ideco NGFW) для доступа в веб-интерфейс](https://sysahelper.ru/mod/page/view.php?id=499)

# Настройка FW-DT (Ideco NGFW) для доступа в веб-интерфейс

#### Настройка локального интерфейса для доступа в веб-интерфейс Ideco NGFW

- В локальной консоли Ideco NGFW выполняем следующие действия:
    1. **n** - чтобы отказаться от настройки сервера как вторую ноду кластера;
    2. **admin** - имя создаваемой учётной записи администратора (будет использоваться для доступа в веб-интерфейс);
    3. **P@ssw0rd1234** - пароль для создаваемой учётной записи администратора
        - Учитывая требования к паролю:
            - Минимальная длина пароля - 12 символов;
                
            - Содержит только строчные и заглавные латинские буквы;
                
            - Содержит цифры;
                
            - Содержит специальные символы (! # $ % & ' * + и другие).
                
    4. **P@ssw0rd1234** - подтверждаем пароль для создаваемой учётной записи администратора

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image.png)

- После создания учётной записи администратора - **нажимаем любую клавишу** для доступа в локальное меню Ideco NGFW:
    1. Вводим логин ранее созданной учётной записи администратора;
    2. Вводим пароль для учётной записи администратора

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%281%29.png)

- После доступа в локальное меню - необходимо сконфигурировать хотябы один локальный интерфейс:
    1. Выбираем сетевую карту, которая смотрит в локальную сеть офиса **DT**;
    2. **n** - для того чтобы отказаться от автоматической настройки локальной сети через DHCP;
    3. Вводим **IP-адрес/префикс** из подсети управления (vlan330);
    4. Указывает VLAN тэг (реализуя подинтерфейс, для дальнейшей маршрутизации между VLAN-ами)

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%282%29.png)

- После настройки локального интерфейса - выходим из меню Управления сервером

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%283%29.png)

- Вся последующая настройка **FW-DT** будет производиться через веб-интерфейс с **ADMIN-DT** используя для доступа:

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%284%29.png)

- Поскольку задание подразумевает разделение сети на подсети с применением VLAN-ов, то для доступа к веб-интерфейсу Ideco NGFW на физическом интерфейсе был указан тэг **330** и IP-адрес из таблицы адресации для соответствующего VLAN-а для **Администраторов**
    - для того чтобы получить доступ с **ADMIN-DT** к веб-интерфейсу **FW-DT** необходимо сконфигурировать виртуальный коммутатор, а именно сделать порт (Bridge), который подключён к **ADMIN-DT** - портом доступа (access) и назначить соответствующий VID (**330**):

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%285%29.png)

- - в результате с **ADMIN-DT** должна появиться связность до **FW-DT**:

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%286%29.png)

- - также с **ADMIN-DT** должен появиться доступ к веб-интерфейсу **FW-DT**:
        - принимаем самоподписанный сертификат

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%287%29.png)

- - - выполняем вход из под ранее созданным пользователем "**admin**" с паролем "**P@ssw0rd1234**":

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%288%29.png)

- После доступа к веб-интерфейсу сразу же меняем имя устройству:

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%289%29.png)

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%2810%29.png)

- - Результат:

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%2811%29.png)

#### Опционально:

- Добавление сертификата Ideco NGFW в доверенные, для ускорение дальнейшей работы с сетевыми интерфейсами:
    - Скачиваем корневой сертификат Ideco NGFW из веб-интерфейса

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%2812%29.png)

- - Результат:

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%2813%29.png)

- - На **ADMIN-DT** добавляем скаченный сертификат в доверенные:

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%2814%29.png)

- - Результат:

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%2815%29.png)

![](https://sysahelper.ru/pluginfile.php/850/mod_page/content/5/image%20%2816%29.png)

# Настройка FW-DT (Ideco NGFW) назначение IP-адресов на локальные интерфейсы

### Назначение IP-адресов на локальные интерфейсы

- С **ADMIN-DT** переходим в веб-интерфейс **FW-DT** для дальнейшей настройки локальных интерфейсов:
    - В модуле **Сервисы** переходим в раздел **Сетевые интерфейсы** и первым делом редактируем уже существующее подключение:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%285%29.png)

- - Даём понятное имя для локального интерфейса, также помещаем интерфейс в зону безопасности, которую необходимо сначала создать:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%286%29.png)

- - В результате получаем следующую конфигурацию для существующего локального интерфейса для подсети Администраторов
        - проверяем и нажимаем **Сохранить**:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%287%29.png)

- - - Результат:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%288%29.png)

- - Добавляем **Локальный Ethernet** для Серверной подсети (vlan220):

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%289%29.png)

- - Выбираем уже использующийцся физический интерфейс для реализации на его основе подинтерфейса:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2810%29.png)

- - Заполняем необходимые поля в соответствие с таблицей адресации и нажимаем **Добавить**:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2811%29.png)

- - - Результат:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2812%29.png)

- - Аналогичным образом добавляем подинтерфейс для Клиентской подсети (vlan110)
        - Результат:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2813%29.png)

- Также реализуем подключение к маршрутизатору **R-DT**, используя второе физическое подключение
    - Добавляем **Локальный Ethernet**:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2814%29.png)

- - Выбираем ранее не использованный физический интерфейс:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2815%29.png)

- - Заполняем все необходимые поля, за исключением IP-адреса шлюза по умолчанию
        - по условиям задания: _FW-DT должен получать маршрут по умолчанию и другие необходимые маршруты от R-DT через OSPF_

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2816%29.png)

- - - Результат:

![](https://sysahelper.ru/pluginfile.php/855/mod_page/content/5/image%20%2817%29.png)

# Настройте подключения маршрутизаторов к провайдеру (подключения R-HQ)

b) Для подключения R-HQ к провайдеру необходимо должен использовать последний адрес из сети 172.16.5.0/28.

c) Провайдер использует первый адрес из каждой сети

### Вариант реализации:

### R-HQ:

- Выполняем создание **интерфейса** с именем **isp** и назначаем IPv4-адрес согласно таблице адресации в сторону **ISP**:

```
r-hq#configure terminal
r-hq(config)#interface isp
r-hq(config-if)#ip address 172.16.5.14/28
r-hq(config-if)#exit
r-hq(config)#
```

- Переходим в режим конфигурирования порта **te0** - создаём **service-instance** с именем **te0/isp:**
    - также указываем что будет обрабатывать не тегированный трафик (**untagged**);
    - и указываем в какой интерфейс нужно отправить обработанные кадры (**isp**);

```
r-hq(config)#port te0
r-hq(config-port)#service-instance te0/isp
r-hq(config-service-instance)#encapsulation untagged
r-hq(config-service-instance)#connect ip interface isp
r-hq(config-service-instance)#exit
r-hq(config-port)#exit
r-hq(config)#
```

- Задаём IP-адрес шлюза по умолчанию:

```
r-hq(config)#ip route 0.0.0.0/0 172.16.5.1
r-hq(config)#write
r-hq(config)#
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/819/mod_page/content/4/image.png)

# Настройте подключения маршрутизаторов к провайдеру (подключения R-DT)

a) Для подключения R-DT к провайдеру необходимо использовать последний адрес из сети 172.16.4.0/28.

c) Провайдер использует первый адрес из каждой сети

### Вариант реализации:

### R-DT:

- Выполняем создание **интерфейса** с именем **isp** и назначаем IPv4-адрес согласно таблице адресации в сторону **ISP**:

```
r-dt#configure terminal
r-dt(config)#interface isp
r-dt(config-if)#ip address 172.16.4.14/28
r-dt(config-if)#exit
r-dt(config)#
```

- Переходим в режим конфигурирования порта **te0** - создаём **service-instance** с именем **te0/isp:**
    - также указываем что будет обрабатывать не тегированный трафик (**untagged**);
    - и указываем в какой интерфейс нужно отправить обработанные кадры (**isp**);

```
r-dt(config)#port te0
r-dt(config-port)#service-instance te0/isp
r-dt(config-service-instance)#encapsulation untagged
r-dt(config-service-instance)#connect ip interface isp
r-dt(config-service-instance)#exit
r-dt(config-port)#exit
r-dt(config)#
```

- Задаём IP-адрес шлюза по умолчанию:

```
r-dt(config)#ip route 0.0.0.0/0 172.16.4.1
r-dt(config)#write
r-dt(config)#
```

- - Проверяем:

![](

# Настройка динамической трансляции адресов (HQ)

a) Настройте на маршрутизаторах динамическую трансляцию адресов.

b) Все устройства во всех офисах должны иметь доступ к сети Интернет

### Вариант реализации:

### R-HQ:

- С точки зрения **EcoRouter** - реализуем конфигурацию **static source PAT:**
    - Интерсейс в сторону **ISP** с именем **isp** - назначаем как **nat outside**:

```
r-hq#configure terminal
r-hq(config)#interface isp
r-hq(config-if)#ip nat outside
r-hq(config-if)#exit
r-hq(config)#
```

- - Подинтерфейсы  **vl110, vl220, vl330,** которые смотрят в сторону **SW1-HQ** - назначаем как **nat inside**:

```
r-hq(config)#interface vl110
r-hq(config-if)#ip nat inside
r-hq(config-if)#exit
r-hq(config)#
r-hq(config)#interface vl220
r-hq(config-if)#ip nat inside
r-hq(config-if)#exit
r-hq(config)#
r-hq(config)#interface vl330
r-hq(config-if)#ip nat inside
r-hq(config-if)#exit
r-hq(config)#
```

- - создаём пула адресов с именем **LOCAL-HQ** для входящего трафика - указываем диапазон адресов из выделенной подсети:

```
r-hq(config)#ip nat pool LOCAL-HQ 192.168.11.1-192.168.11.254
```

- - задаём правила для трансляции адресов:

```
r-hq(config)#ip nat source dynamic inside pool LOCAL-HQ overload 172.16.5.14
r-hq(config)#write
r-hq(config)#
```

#### Для проверки:

- Проверяем:
    - т.к. на данном этапе ещё не настройена коммутация на **SW1-HQ** - проверить работоспособность **NAT** можно назначив средствами **iproute2** временно на интерфейс **SW1-HQ** на интерфейс,смотрящий в сторону **R-HQ** - тегированный подинтерфейс с IP-адресом из подсети для **vlan330:**

```
ip link add link ens19 name ens19.330 type vlan id 330
ip link set dev ens19.330 up
ip addr add 192.168.11.82/29 dev ens19.330
ip route add 0.0.0.0/0 via 192.168.11.81
```

- - затем проверяем доступ в сеть Интернет:

![](https://sysahelper.ru/pluginfile.php/821/mod_page/content/2/image.png)

- - после чего можно проверить таблицу трансляции адресов на **R-HQ**:

![](https://sysahelper.ru/pluginfile.php/821/mod_page/content/2/image%20%281%29.png)

# Настройка динамической трансляции адресов (DT)
a) Настройте на маршрутизаторах динамическую трансляцию адресов.

b) Все устройства во всех офисах должны иметь доступ к сети Интернет

### Вариант реализации:

### R-DT:

- С точки зрения **EcoRouter** - реализуем конфигурацию **static source PAT:**
    - интерсейс в сторону **ISP** с именем **int** - назначаем как **nat outside**:

```
r-dt#configure terminal
r-dt(config)#interface isp
r-dt(config-if)#ip nat outside
r-dt(config-if)#exit
r-dt(config)#
```

- - Интерфейс **int1** в сторону **FW-DT** - назначаем как **nat inside**:

```
r-dt(config)#interface int1
r-dt(config-if)#ip nat inside
r-dt(config-if)#exit
r-dt(config)#
```

- - Создаём пула адресов с именем **LOCAL-DT** для входящего трафика - указываем диапазон адресов из выделенной подсети:

```
r-dt(config)#ip nat pool LOCAL-DT 192.168.33.1-192.168.33.254
```

- - Задаём правила для трансляции адресов:

```
r-dt(config)#ip nat source dynamic inside pool LOCAL-DT overload 172.16.4.14
r-dt(config)#write
r-dt(config)#
```

# Настройка коммутации (SW1-HQ, SW2-HQ, SW3-HQ VLAN-ы)

a) Настройте коммутаторы SW1-HQ, SW2-HQ, SW3-HQ.

- 1. Используйте Open vSwitch
- 2. Имя коммутатора должно совпадать с коротким именем устройства
    - i. Используйте заглавные буквы

3. Передайте все физические порты коммутатору.

4. Обеспечьте включение портов, если это необходимо

c) Для каждого офиса устройства должны находиться в соответствующих VLAN

- 1. Клиенты - vlan110,
- 2. Сервера – в vlan220,
- 3. Администраторы – в vlan330.

### Вариант реализации:

**SW1-HQ:**

- Поскольку на данном этапе ещё не настройена коммутация на **SW1-HQ**, а установить пакет **openvswitch** необходимо, то можно назначив средствами **iproute2** временно на интерфейс,смотрящий в сторону **R-HQ** - тегированный подинтерфейс с IP-адресом из подсети для **vlan330:**

```
ip link add link ens19 name ens19.330 type vlan id 330
ip link set dev ens19.330 up
ip addr add 192.168.11.82/29 dev ens19.330
ip route add 0.0.0.0/0 via 192.168.11.81
echo "nameserver 77.88.8.8" > /etc/resolv.conf
```

- Обновляем список пакетов и устанавливаем **openvswitch**:

```
apt-get update && apt-get install -y openvswitch
```

- Включаем и добавляем в автозагрузку **openvswitch**:

```
systemctl enable --now openvswitch
```

- Перезагрузить сервер будет быстрее чем удалять параметры заданые в ручную через пакет **iproute2**:

```
reboot
```

- Проверяем интерфейсы и определяемся какой к кому направлен:
    - таким образом, имеем:
        - **ens19** - интерфейс в сторону **R-HQ**;
        - **ens20** - интерфейс в сторону **SW2-HQ**;
        - **ens21** - интерфейс в сторону **SW3-HQ**.

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image.png)

- Обеспечим включение портов **ens20** и **ens21**:
    - Создадим для них одноимённые директории по пути **/etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА>**:

```
mkdir /etc/net/ifaces/ens2{0,1}
```

- - Для каждого интерфейса необходимо создать конфигурационный файл **options**:
        - данный файл должен включать в себя минимально необходимое содержимое, а именно **два** параметра: **TYPE** и **BOOTPROTO**
        - создаём данный файл для интерфейса **ens20**:

```
cat <<EOF > /etc/net/ifaces/ens20/options
  TYPE=eth
  BOOTPROTO=static
EOF
```

- - - **P.S.** или же открываем через текстовы редактор, например: **vim**
    - Файл **options** для интерфейса **ens21** будет аналогичен как и для **ens20** - поэтому его можно просто скопировать:

```
cp /etc/net/ifaces/ens20/options /etc/net/ifaces/ens21/
```

- - Также важно, чтобы и для **ens19** в файле **options** параметр **BOOTPROTO** имел значение **static**:

```
sed -i "s/BOOTPROTO=dhcp/BOOTPROTO=static/g" /etc/net/ifaces/ens19/options
```

- - Перезагружаем службу **network** для применения изменений:

```
systemctl restart network
```

- - Проверяем что все интерфейсы перешли в состояние **UP**:

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%281%29.png)

- Создадим коммутатор имя которого должно совпадать с коротким именем устройства и с использованием заглавных букв - **SW1-HQ**:

```
ovs-vsctl add-br SW1-HQ
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%282%29.png)

- Сетевая подсистема **etcnet** будет взаимодействовать с **openvswitch**, для того чтобы корректно можно было назначить  IP-адрес на  интерфейс управления  
    - Создаём каталог для management интерфейса с именем **MGMT**:

```
mkdir /etc/net/ifaces/MGMT
```

- - Описываем файл **options** для создания management интерфейса с именем **mgmt**:

```
vim /etc/net/ifaces/MGMT/options
```

- - Содержимое, где:
        - **TYPE** - тип интерфейса (**internal**);
        - **BOOTPROTO** - определяет как будут назначаться сетевые параметры (статически);
        - **CONFIG_IPV4** - определяет использовать конфигурацию протокола IPv4 или нет;
        - **BRIDGE** - определяет к какому мосту необходимо добавить данный интерфейс;
        - **VID** - определяет принадлежность интерфейса к VLAN;

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%283%29.png)

- - Назначаем IP-адрес и шлюз на созданный интерфейс **MGMT** согласно таблице адресации для Администраторской подсети:

```
echo "192.168.11.82/29" > /etc/net/ifaces/MGMT/ipv4address
```

```
echo "default via 192.168.11.81" > /etc/net/ifaces/MGMT/ipv4route
```

- - Правим основной файл **options** в котором по умолчанию сказано удалять настройки заданые через **ovs-vsctl,** т.к. через **etcnet** будет выполнено только создание **bridge** и интерфейса типа **internal** с назначением необходимого IP-адреса, а настройка функционала будет выполнена средствами **openvswitch**:

```
sed -i "s/OVS_REMOVE=yes/OVS_REMOVE=no/g" /etc/net/ifaces/default/options
```

- Перезапускаем службу **network**:

```
systemctl restart network
```

- Проверяем:
    - На текущей момент создан интерфейс управления и назначем IP-адрес из соответствующей подсети
    - Данный интерфейс управления помечен тегом (VID) 330 и добавлен в bridge SW1-HQ

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%284%29.png)

- Средствами **openvswitch** настраиваем следующий функционал:
    - Порт в сторону маршрутизатора и в сторону остальных коммутаторов должны быть магистральными и пропускать использующиеся VLAN-ы:

```
ovs-vsctl add-port SW1-HQ ens19 trunk=110,220,330
```

```
ovs-vsctl add-port SW1-HQ ens20 trunk=110,220,330
```

```
ovs-vsctl add-port SW1-HQ ens21 trunk=110,220,330
```

- - Включаем модуль ядра **8021q:**

```
modprobe 8021q
```

- - При необходимости добавляем и на постоянной основе:

```
echo "8021q" | tee -a /etc/modules
```

- Проверяем:
    - наличие портов в коммутаторе
    - включённый модуль
    - доступ в сеть Интернет

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%285%29.png)

**SW2-HQ:**

- Поскольку на данном этапе ещё не настройена коммутация на **SW2-HQ**, а установить пакет **openvswitch** необходимо, то можно назначив средствами **iproute2** временно на интерфейс смотрящий в сторону **SW1-HQ** тегированный подинтерфейс с IP-адресом из подсети для **vlan330:**

```
ip link add link ens19 name ens19.330 type vlan id 330
ip link set dev ens19.330 up
ip addr add 192.168.11.83/29 dev ens19.330
ip route add 0.0.0.0/0 via 192.168.11.81
echo "nameserver 77.88.8.8" > /etc/resolv.conf
```

- Обновляем список пакетов и устанавливаем **openvswitch**:

```
apt-get update && apt-get install -y openvswitch
```

- Включаем и добавляем в автозагрузку **openvswitch**:

```
systemctl enable --now openvswitch
```

Перезагрузить сервер будет быстрее чем удалять параметры заданые в ручную через пакет **iproute2**:

```
reboot
```

- Проверяем интерфейсы и определяемся какой к кому направлен:
    - Таким образом, имеем:
        - **ens19** - интерфейс в сторону **SW1-HQ**
        - **ens20** - интерфейс в сторону **SW3-HQ**
        - **ens21** - интерфейс в сторону **SRV1-HQ**
        - **ens22** - интерфейс в сторону **CLI-HQ**

**![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%287%29.png)**

- Обеспечим включение портов **ens20,** **ens21** и **ens22**:
    - Создадим для них одноимённые директории по пути **/etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА>**:

```
mkdir /etc/net/ifaces/ens2{0..2}
```

- - Для каждого интерфейса необходимо создать конфигурационный файл **options**:
        - Данный файл должен включать в себя минимально необходимое содержимое, а именно **два** параметра: **TYPE** и **BOOTPROTO**
        - Создаём данный файл для интерфейса **ens20**:

```
cat <<EOF > /etc/net/ifaces/ens20/options
  TYPE=eth
  BOOTPROTO=static
EOF
```

- - - **P.S.** или же открываем через текстовы редактор, например: **vim**
    - Файл **options** для интерфейса **ens21** и **ens22** будет аналогичен как и для **ens20**, поэтому его можно просто скопировать:

```
cp /etc/net/ifaces/ens20/options /etc/net/ifaces/ens21/
```

```
cp /etc/net/ifaces/ens20/options /etc/net/ifaces/ens22/
```

- - Также важно, чтобы для **ens19** в файле **options** параметр **BOOTPROTO** имел значение **static**:

```
sed -i "s/BOOTPROTO=dhcp/BOOTPROTO=static/g" /etc/net/ifaces/ens19/options
```

- - Перезагружаем службу **network** для применения изменений:

```
systemctl restart network
```

- - Проверяем что все интерфейсы перешли в состояние **UP**:

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%288%29.png)

- Создадим коммутатор имя которого должно совпадать с коротким именем устройства с использованием заглавных букв - **SW2-HQ**:

```
ovs-vsctl add-br SW2-HQ
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%289%29.png)

- Сетевая подсистема **etcnet** будет взаимодействовать с **openvswitch**, для того чтобы корректно можно было назначить  IP-адрес на  интерфейс управления  
    - Создаём каталог для management интерфейса с именем **MGMT**:

```
mkdir /etc/net/ifaces/MGMT
```

- - Описываем файл **options** для создания management интерфейса с именем **mgmt**:

```
vim /etc/net/ifaces/MGMT/options
```

- - Содержимое, где:
        - **TYPE** - тип интерфейса (**internal**);
        - **BOOTPROTO** - определяет как будут назначаться сетевые параметры (статически);
        - **CONFIG_IPV4** - определяет использовать конфигурацию протокола IPv4 или нет;
        - **BRIDGE** - определяет к какому мосту необходимо добавить данный интерфейс;
        - **VID** - определяет принадлежность интерфейса к VLAN;

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2810%29.png)

- - Назначаем IP-адрес и шлюз на созданный интерфейс **MGMT** согласно таблеце адресации для Администраторской подсети:

```
echo "192.168.11.83/29" > /etc/net/ifaces/MGMT/ipv4address
```

```
echo "default via 192.168.11.81" > /etc/net/ifaces/MGMT/ipv4route
```

- - Правим основной файл **options** в котором по умолчанию сказано удалять настройки заданые через **ovs-vsctl,** т.к. через **etcnet** будет выполнено только создание **bridge** и интерфейса типа **internal** с назначением необходимого IP-адреса, а настройка функционала будет выполнена средствами **openvswitch**:

```
sed -i "s/OVS_REMOVE=yes/OVS_REMOVE=no/g" /etc/net/ifaces/default/options
```

- Перезапускаем службу **network**:

```
systemctl restart network
```

- Проверяем:
    - На текущей момент создан интерфейс управления и назначем IP-адрес из соответствующей подсети
    - Данный интерфейс управления помечен тегом (VID) 330 и добавлен в bridge SW1-HQ

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2811%29.png)

- Средствами **openvswitch** настраиваем следующий функционал:
    - Порты в сторону коммутаторов (**ens19, ens20**) должны быть магистральными и пропускать использующиеся VLAN-ы:

```
ovs-vsctl add-port SW2-HQ ens19 trunk=110,220,330
```

```
ovs-vsctl add-port SW2-HQ ens20 trunk=110,220,330
```

- - Порт в сторону **SRV1-HQ (ens21)** должен быть портом доступа и принадлежать VLAN **220** (Сервера):

```
ovs-vsctl add-port SW2-HQ ens21 tag=220
```

- - Порт в сторону **CLI-HQ (ens22)** должен быть портом доступа и принадлежать VLAN **110** (Клиенты):

```
ovs-vsctl add-port SW2-HQ ens22 tag=110
```

- - Включаем модуль ядра **8021q:**

```
modprobe 8021q
```

- - При необходимости добавляем и на постоянной основе:

```
echo "8021q" | tee -a /etc/modules
```

- Проверяем:
    - наличие портов в коммутаторе
    - включённый модуль
    - доступ в сеть Интернет

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2812%29.png)

**SW3-HQ:**

- Поскольку на данном этапе ещё не настройена коммутация на **SW3-HQ**, а установить пакет **openvswitch** необходимо, то можно назначив средствами **iproute2** временно на интерфейс,смотрящий в сторону **SW1-HQ** тегированный подинтерфейс с IP-адресом из подсети для **vlan330:**

```
ip link add link ens19 name ens19.330 type vlan id 330
ip link set dev ens19.330 up
ip addr add 192.168.11.84/29 dev ens19.330
ip route add 0.0.0.0/0 via 192.168.11.81
echo "nameserver 77.88.8.8" > /etc/resolv.conf
```

- Обновляем список пакетов и устанавливаем **openvswitch**:

```
apt-get update && apt-get install -y openvswitch
```

- Включаем и добавляем в автозагрузку **openvswitch**:

```
systemctl enable --now openvswitch
```

- Перезагрузить сервер будет быстрее чем удалять параметры заданые в ручную через пакет **iproute2**:

```
reboot
```

- Проверяем интерфейсы и определяемся какой к кому направлен:
    - Таким образом, имеем:
        - **ens19** - интерфейс в сторону **SW1-HQ**
        - **ens20** - интерфейс в сторону **SW2-HQ**
        - **ens21** - интерфейс в сторону **ADMIN-HQ**

**![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2814%29.png)  
**

- Обеспечим включение портов **ens20** и **ens21**:
    - Создадим для них одноимённые директории по пути **/etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА>**:

```
mkdir /etc/net/ifaces/ens2{0,1}
```

- - Для каждого интерфейса необходимо создать конфигурационный файл **options**:
        - Данный файл должен включать в себя минимально необходимое содержимое, а именно **два** параметра: **TYPE** и **BOOTPROTO**
        - Создаём данный файл для интерфейса **ens20**:

```
cat <<EOF > /etc/net/ifaces/ens20/options
  TYPE=eth
  BOOTPROTO=static
EOF
```

- - - **P.S.** или же открываем через текстовы редактор, например: **vim**
    - Файл **options** для интерфейса **ens21** будет аналогичен как и для **ens20** - поэтому его можно просто скопировать:

```
cp /etc/net/ifaces/ens20/options /etc/net/ifaces/ens21/
```

- - Также важно, чтобы и для **ens19** в файле **options** параметр **BOOTPROTO** имел значение **static**:

```
sed -i "s/BOOTPROTO=dhcp/BOOTPROTO=static/g" /etc/net/ifaces/ens19/options
```

- - Перезагружаем службу **network** для применения изменений:

```
systemctl restart network
```

- - Проверяем что все интерфейсы перешли в состояние **UP**:

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2815%29.png)

- Создадим коммутатор имя которого должно совпадать с коротким именем устройства с использованием заглавных букв - **SW3-HQ**:

```
ovs-vsctl add-br SW3-HQ
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2816%29.png)

- Сетевая подсистема **etcnet** будет взаимодействовать с **openvswitch**, для того чтобы корректно можно было назначить  IP-адрес на  интерфейс управления  
    - Создаём каталог для management интерфейса с именем **MGMT**:

```
mkdir /etc/net/ifaces/MGMT
```

- - Описываем файл **options** для создания management интерфейса с именем **mgmt**:

```
vim /etc/net/ifaces/MGMT/options
```

- - Содержимое, где:
        - **TYPE** - тип интерфейса (**internal**);
        - **BOOTPROTO** - определяет как будут назначаться сетевые параметры (статически);
        - **CONFIG_IPV4** - определяет использовать конфигурацию протокола IPv4 или нет;
        - **BRIDGE** - определяет к какому мосту необходимо добавить данный интерфейс;
        - **VID** - определяет принадлежность интерфейса к VLAN;

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2817%29.png)

- - Назначаем IP-адрес и шлюз на созданный интерфейс **MGMT** согласно таблице адресации для Администраторской подсети:

```
echo "192.168.11.84/29" > /etc/net/ifaces/MGMT/ipv4address
```

```
echo "default via 192.168.11.81" > /etc/net/ifaces/MGMT/ipv4route
```

- - Правим основной файл **options** в котором по умолчанию сказано удалять настройки заданые через **ovs-vsctl,** т.к. через **etcnet** будет выполнено только создание **bridge** и интерфейса типа **internal** с назначением необходимого IP-адреса, а настройка функционала будет выполнена средствами **openvswitch**:

```
sed -i "s/OVS_REMOVE=yes/OVS_REMOVE=no/g" /etc/net/ifaces/default/options
```

- Перезапускаем службу **network**:

```
systemctl restart network
```

- Проверяем:
    - На текущей момент создан интерфейс управления и назначем IP-адрес из соответствующей подсети
    - Данный интерфейс управления помечен тегом (VID) 330 и добавлен в bridge SW1-HQ

![](https://sysahelper.ru/pluginfile.php/815/mod_page/content/4/image%20%2818%29.png)

- Средствами **openvswitch** настраиваем следующий функционал:
    - Порты в сторону коммутаторов (**ens19, ens20**) должны быть магистральными и пропускать использующиеся VLAN-ы:

```
ovs-vsctl add-port SW3-HQ ens19 trunk=110,220,330
```

```
ovs-vsctl add-port SW3-HQ ens20 trunk=110,220,330
```

- - Порт в сторону **ADMIN-HQ (ens21)** должен быть портом доступа и принадлежать VLAN - **330** (Администраторы):

```
ovs-vsctl add-port SW3-HQ ens21 tag=330
```

- - Включаем модуль ядра **8021q:**

```
modprobe 8021q
```

- - При необходимости добавляем и на постоянной основе:

```
echo "8021q" | tee -a /etc/modules
```

# Настройка коммутации (протокол основного дерева

6. Настройте протокол основного дерева

- i. Корнем дерева должен выступать SW1-HQ

### Вариант реализации:

**SW1-HQ:**

Настроим Bridge **SW1-HQ** на участие в дереве 802.1D: 

```
ovs-vsctl set bridge SW1-HQ stp_enable=true
```

- - Задаём приоритет для Bridge **SW1-HQ** в **16384**, т.к. по условиям задания он должен быть корневым:

```
ovs−vsctl set bridge SW1-HQ other_config:stp-priority=16384
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/816/mod_page/content/2/image.png)

**SW2-HQ:**

- Настроим Bridge **SW2-HQ** на участие в дереве 802.1D: 

```
ovs-vsctl set bridge SW2-HQ stp_enable=true
```

- - Задаём приоритет для Bridge **SW2-HQ** в **24576**, т.к. по условиям задания **SW1-HQ** должен быть корневым:

```
ovs−vsctl set bridge SW2-HQ other_config:stp-priority=24576
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/816/mod_page/content/2/image%20%281%29.png)

**SW3-HQ:**

- Настроим Bridge **SW3-HQ** на участие в дереве 802.1D: 

```
ovs-vsctl set bridge SW3-HQ stp_enable=true
```

- - Задаём приоритет для Bridge **SW3-HQ** в **28672**, т.к. по условиям задания **SW1-HQ** должен быть корневым:

```
ovs−vsctl set bridge SW3-HQ other_config:stp-priority=28672
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/816/mod_page/content/2/image%20%282%29.png)
# Настройка коммутации (коммутатор SW-DT)
b) Настройте коммутатор SW-DT

- 1. В качестве коммутатора используйте соответствующий виртуальный коммутатор.

c) Для каждого офиса устройства должны находиться в соответствующих VLAN

- 1. Клиенты - vlan110,
- 2. Сервера – в vlan220,
- 3. Администраторы – в vlan330.

### Вариант реализации:

- В качестве коммутатора используется виртуальный коммутатор на уровне Альт Виртуализаци PVE (на котором развёрнут стенд) **vmbr112**:  
    - подразумевается что для **vmbr112** в текущем случае выставлен чек-бокс:

![](https://sysahelper.ru/pluginfile.php/817/mod_page/content/2/image.png)

- Реализуем необходимые порты "доступа (access)":
    - **SRV1-DT** (vlan 220 - Сервера):

![](https://sysahelper.ru/pluginfile.php/817/mod_page/content/2/image%20%281%29.png)

- - **SRV2-DT** (vlan 220 - Сервера):

![](https://sysahelper.ru/pluginfile.php/817/mod_page/content/2/image%20%282%29.png)

- - **SRV3-DT** (vlan 220 - Сервера):

![](https://sysahelper.ru/pluginfile.php/817/mod_page/content/2/image%20%283%29.png)

- - **CLI-DT** (vlan 110 - Клиента):

![](https://sysahelper.ru/pluginfile.php/817/mod_page/content/2/image%20%284%29.png)

_P.S. ADMIN-DT был настроен ранее на этапе: [Настройка FW-DT (Ideco NGFW) для доступа в веб-интерфейс](https://sysahelper.ru/mod/page/view.php?id=499)_

# Настройка протокола динамической конфигурации хостов (HQ)

### Задание:

a) На R-HQ настройте протокол динамической конфигурации хостов для клиентов (CLI-HQ)

- 1. Адрес сети – согласно топологии
    - i. Исключите адрес шлюза по умолчанию из диапазона выдаваемых адресов
- 2. Адрес шлюза по умолчанию – в соответствии с топологией
    - i. Шлюзом для сети HQ является маршрутизатор R-HQ
- 3. DNS-суффикс – au.team
- 4. Настройте клиентов на получение динамических адресов.

### Вариант реализации:

### R-HQ:

- Задаём POOL адресов с именем **CLI-HQ**, затем задаём диапазон IP-адресов, который будет раздаваться DHCP сервером:
    - в данном случае раздаваться будет вся клиентская подсеть заисключением IP-адреса маршрутизатора R-HQ

```
r-hq#configure terminal
r-hq(config)#ip pool CLI-HQ 192.168.11.2-192.168.11.62
r-hq(config)#
```

- Для настройки DHCP-сервера необходимо в режиме конфигурации ввести команду **dhcp-server <NUMBER>**
    - где **NUMBER** – номер сервера в системе маршрутизатора:

```
r-hq(config)#dhcp-server 1
r-hq(config-dhcp-server)#
```

- Привязываем ранее созданный POOL раздаваемых адресов с именем **CLI-HQ**, а также указанием номера сервера в системе маршрутизатора **1**:

```
r-hq(config-dhcp-server)#pool CLI-HQ 1
r-hq(config-dhcp-server-pool)#
```

- Задаём основные параметры для раздачи DHCP сервером:

```
r-hq(config-dhcp-server-pool)#mask 26
r-hq(config-dhcp-server-pool)#gateway 192.168.11.1
r-hq(config-dhcp-server-pool)#dns 192.168.11.66,192.168.33.66
r-hq(config-dhcp-server-pool)#domain-name au.team
r-hq(config-dhcp-server-pool)#exit
r-hq(config-dhcp-server)#exit
r-hq(config)#
```

- После настройки сервера необходимо указать, на каком интерфейсе маршрутизатор будет принимать пакеты DHCP Discover и отвечать на них предложением с IP-настройками:
    - в данном случае подинтерфейс с именем **vl110** смотрит в сторону клиентской подсети (vlan110)

```
r-hq(config)#interface vl110
r-hq(config-if)#dhcp-server 1
r-hq(config-if)#exit
r-hq(config)#write
r-hq(config)#
```

### CLI-HQ:

- Настраиваем клиента на получение динамических адресов:

![](https://sysahelper.ru/pluginfile.php/823/mod_page/content/3/image.png)

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/823/mod_page/content/3/image%20%281%29.png)

# Настройка протокола динамической конфигурации хостов (DT)

### Задание:

a) На R-DT настройте протокол динамической конфигурации хостов для клиентов (CLI-DT)

- 1. Адрес сети – согласно топологии
    - i. Исключите адрес шлюза по умолчанию из диапазона выдаваемых адресов
- 2. Адрес шлюза по умолчанию – в соответствии с топологией
    - i. Шлюзом для сети DT является межсетевой экран FW-DT
- 3. DNS-суффикс – au.team
- 4. Настройте клиентов на получение динамических адресов.

### Вариант реализации:

### R-DT:

Аналогично R-HQ, подробный процесс настройки DHCP - сервера рассмотрет в [Настройка протокола динамической конфигурации хостов (HQ)](https://sysahelper.ru/mod/page/view.php?id=472)

- Таким образом настройка DHCP - сервера выглядит следующим образом:

```
r-dt#configure terminal
r-dt(config)#ip pool CLI-DT 192.168.33.2-192.168.33.62
r-dt(config)#dhcp-server 1
r-dt(config-dhcp-server)#pool CLI-DT 1
r-dt(config-dhcp-server-pool)#mask 26
r-dt(config-dhcp-server-pool)#gateway 192.168.33.1
r-dt(config-dhcp-server-pool)#dns 192.168.33.66,192.168.11.66
r-dt(config-dhcp-server-pool)#domain-name au.team
r-dt(config-dhcp-server-pool)#exit
r-dt(config-dhcp-server)#exit
r-dt(config)#interface int1
r-dt(config-if)#dhcp-server 1
r-dt(config-if)#exit
r-dt(config)#write
r-dt(config)#
```

### FW-DT:

- Настраиваем **DHCP Relay**:
    - В веб-интерфейсе **FW-DT** с **ADMIN-DT** переходим в модуле **Сервисы** в раздел **DHCP-сервер** и нажимаем **Добавить**:

![](https://sysahelper.ru/pluginfile.php/822/mod_page/content/3/image.png)

- - Выбираем интерфейс, который будет участвовать в раздаче IP-адресов, затем введим IP-адрес DHCP-сервера:

![](https://sysahelper.ru/pluginfile.php/822/mod_page/content/3/image%20%281%29.png)

- - Активируем службу для работы DHCP-Relay:

![](https://sysahelper.ru/pluginfile.php/822/mod_page/content/3/image%20%282%29.png)

- - Результат:

![](https://sysahelper.ru/pluginfile.php/822/mod_page/content/3/image%20%283%29.png)

### CLI-DT:

- Настраиваем клиента на получение динамических адресов:

![](https://sysahelper.ru/pluginfile.php/822/mod_page/content/3/image%20%284%29.png)

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/822/mod_page/content/3/image%20%285%29.png)

P.S. на текущий момент доступа в сеть Интернет из офиса DT быть не должно:

1. Не реализована авторизация на FW-DT;
2. FW-DT не имеет IP-адреса шлюза по умолчанию, т.к. не было настройки OSPF ещё.

# Между офисами DT и HQ необходимо сконфигурировать ip туннель

6) Между офисами DT и HQ необходимо сконфигурировать ip туннель

- a) Используйте GRE

### Вариант реализации:

### R-DT:

- Настраиваем **GRE** туннель в сторону **R-HQ**,
    - где: i**nterface tunnel.<номер>** - номер это произвольное число

```
r-dt#configure terminal
r-dt(config)#  interface tunnel.0
r-dt(config-if-tunnel)#description "Connect_HQ-R"
r-dt(config-if-tunnel)#ip add 10.10.10.1/30
r-dt(config-if-tunnel)#ip mtu 1476
r-dt(config-if-tunnel)#ip tunnel 172.16.4.14 172.16.5.14 mode gre
r-dt(config-if-tunnel)#end
r-dt#write
r-dt#
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/824/mod_page/content/2/image.png)

### R-HQ:

- Настраиваем **GRE** - туннель в сторону **R-DT**,
    - где: **interface tunnel.<номер>** - номер это произвольное число

```
r-hq#configure terminal
r-hq(config)#  interface tunnel.0
r-hq(config-if-tunnel)#description "Connect_DT-R"
r-hq(config-if-tunnel)#ip add 10.10.10.2/30
r-hq(config-if-tunnel)#ip mtu 1476
r-hq(config-if-tunnel)#ip tunnel 172.16.5.14 172.16.4.14 mode gre
r-hq(config-if-tunnel)#end
r-hq#write
r-hq#
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/824/mod_page/content/2/image%20%281%29.png)

# Настройте динамическую маршрутизацию OSPF (DT и HQ)

a) Между офисами DT и HQ

- 1. Маршрутизаторы должны быть защищены от вброса маршрутов с любых интерфейсов, кроме тех, на которых обмен маршрутами явно требуется.
- 2. Обеспечьте защиту протокола маршрутизации посредством парольной защиты
    - i. Используйте пароль P@ssw0rd

### Вариант реализации:

### R-HQ:

- Настраиваем OSPFv2:
    - Задаём router-id;
    - Переводим все интерфейсы в пассивный режим;
    - Объявляем сети;
    - На туннельном интерфейсе отключаем пассивный режим, чтобы можно было установить соседство:

```
r-hq#configure terminal
r-hq(config)#router ospf 1
r-hq(config-router)#ospf router-id 10.10.10.2
r-hq(config-router)#passive-interface default
r-hq(config-router)#network 10.10.10.0  0.0.0.3 area 0
r-hq(config-router)#network 192.168.11.0 0.0.0.63 area 0
r-hq(config-router)#network 192.168.11.64 0.0.0.15 area 0
r-hq(config-router)#network 192.168.11.80 0.0.0.7 area 0
r-hq(config-router)#no passive-interface tunnel.0
r-hq(config-router)#exit
r-hq(config)#exit
r-hq#write
r-hq#
```

- Обеспечиваем защиту протокола маршрутизации посредством парольной защиты:

```
r-hq#configure terminal
r-hq(config)#interface tunnel.0
r-hq(config-if-tunnel)#ip ospf authentication message-digest
r-hq(config-if-tunnel)#ip ospf message-digest-key 1 md5 P@ssw0rd
r-hq(config-if-tunnel)#exit
r-hq(config)#write
r-hq(config)#
```

### R-DT:

- Настраиваем OSPFv2:
    - Задаём router-id
    - Переводим все интерфейсы в пассивный режим, т.к. сказано в задании
    - Объявляем сети
    - На туннельном и в сторону FW-DT интерфейсах отключаем пассивный режим, чтобы можно было установить соседство

```
r-dt#configure terminal
r-dt(config)#router ospf 1
r-dt(config-router)#ospf router-id 10.10.10.1
r-dt(config-router)#passive-interface default
r-dt(config-router)#network 10.10.10.0  0.0.0.3 area 0
r-dt(config-router)#network 192.168.33.88 0.0.0.3 area 0
r-dt(config-router)#no passive-interface tunnel.0
r-dt(config-router)#no passive-interface int1
r-dt(config-router)#exit
r-dt(config)#exit
r-dt#write
r-dt#
```

- Обеспечиваем защиту протокола маршрутизации посредством парольной защиты:

```
r-dt#configure terminal
r-dt(config)#interface tunnel.0
r-dt(config-if-tunnel)#ip ospf authentication message-digest
r-dt(config-if-tunnel)#ip ospf message-digest-key 1 md5 P@ssw0rd
r-dt(config-if-tunnel)#exit
r-dt(config)#write
r-dt(config)#
```

- Проверяем:

- - Утсановленное соседство и таблицы маршрутизации:
        - **R-HQ:**

**![](https://sysahelper.ru/pluginfile.php/825/mod_page/content/3/image.png)**

- - - **R-DT:**

![](https://sysahelper.ru/pluginfile.php/825/mod_page/content/3/image%20%281%29.png)

# Настройте динамическую маршрутизацию OSPF (R-DT и FW-DT)
b) Между R-DT и FW-DT

- 1. R-DT должен узнавать о сетях, подключенных к FW-DT по OSPF.
- 2. FW-DT должен получать маршрут по умолчанию и другие необходимые маршруты от R-DT через OSPF.
- 3. R-DT должен быть защищен от вброса маршрутов с любых интерфейсов, кроме тех, на которых обмен маршрутами явно требуется.

### Вариант реализации:

### R-DT:

- FW-DT должен получать маршрут по умолчанию и другие необходимые маршруты от R-DT через OSPF:

```
r-dt#configure terminal
r-dt(config)#router ospf 1
r-dt(config-router)#default-information originate 
r-dt(config-router)#exit
r-dt(config)#  write
r-dt(config)#
```

### FW-DT:

- В веб-интерфейсе **FW-DT** с **ADMIN-DT** переходим в модуль **Сервисы** в раздел **OSPF** на вкладку **ДОПОЛНИТЕЛЬНО** и снимаем чек-боксы с функционала, который не требуется по заданию:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image.png)

- Для настройки OSPF переходим на вкладку **ОСНОВНЫЕ** и нажимаем **Добавить**:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%281%29.png)

- Настраиваем OSPF на локальном интерфейсе:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%282%29.png)

- Аналогично и для всех остальных интерфейсов, в результате должны получить следующее:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%283%29.png)

- Включаем OSPF:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%284%29.png)

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%285%29.png)

P.S. но если приглядеться на маршрут по умолчанию, который получен по OSPF, то можно увидеть **inactive**

- Также результат попытки выхода в сеть Интернет получается следующий:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%287%29.png)

- Полная таблица маршрутизации на FW-DT выглядит следующим образом:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%288%29.png)

- А в результате вывода команды **show runnin-config**:
    - присутствует информация, которая не была целенаправлена задана через веб-интерфейс FW-DT

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%289%29.png)

- Тогда переходим в конфигурационный файл **/etc/frr/frr.conf** (например используя текстовый редактор **vi**) и удаляем данный блок
- После чего необходимо перезагрузить службу **frr:**

```
systemctl restart frr
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/826/mod_page/content/3/image%20%2811%29.png)

# Настройка авторизации на FW-DT (Ideco NGFW) для доступа в сеть Интернет из офиса DT

### Настройка авторизации на FW-DT (Ideco NGFW) для доступа в сеть Интернет из офиса DT

Авторизация - необходимое условие для доступа пользователя в интернет. Для работы в пределах локальной сети авторизация не требуется.

- Реализуем авторизацию на основе **Авторизация по подсетям**
    - Чтобы не регистрировать каждое устройство в виде отдельного пользователя NGFW и не фиксировать для него факторы авторизации, можно создать правило авторизации на вкладке Авторизация по подсетям.
        
    - Эта функция позволяет пользователю NGFW авторизоваться автоматически из требуемой подсети без привязки к конкретному IP/MAC-адресу.
        
    - Правила авторизации по подсетям полезны, когда требуется автоматически авторизовать большое количество устройств. Трафик по всей подсети фиксируется на одного пользователя.
        

- Создадим пользователя для дальнейшей реализации **Авторизации по подсетям** на его основе
    - В модуле **Пользователи** переходим в раздел **Учётные записи** и нажимаем **Добавить пользователя**:

![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image.png)

- - Задаём имя и логин (произвольные) и нажимаем **Добавить** (внизу):

![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%281%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%282%29.png)

- Создаём **Авторизацию по подсети**
    - В модуле **Пользователи** переходим в раздел **Авторизация** на вкладку **АВТОРИЗАЦИЯ ПО ПОДСЕТЯМ** и нажимаем **Добавить**:

![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%283%29.png)

- - Выбираем ранее созданного пользователя и указываем подсеть выделенную для офиса DT:

![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%284%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%285%29.png)

- Проверяем:
    - Пиктограмма пользователя должна стать зелёного цвета (В данный момент пользователь прошел процедуру авторизации, и ему был предоставлен доступ в интернет):

![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%286%29.png)

- - На всех устройствах офиса DT - должен появится доступ в сеть Интернет:
        - **SRV1-DT:**

**![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%2810%29.png)**

- - - **SRV2-DT:**

**![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%2811%29.png)**

- - - **SRV3-DT:**

**![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%2812%29.png)**

- - - **ADMIN-DT:**

**![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%288%29.png)**

- - - **CLI-DT:**

**![](https://sysahelper.ru/pluginfile.php/859/mod_page/content/2/image%20%289%29.png)**

# Базовая настройка (пользователь sshuser)

c) На всех устройства (кроме FW-DT) создайте пользователя sshuser с паролем P@ssw0rd

- 1. Пользователь sshuser должен иметь возможность запуска утилиты sudo без дополнительной аутентификации.
- 2. На маршрутизаторах пользователь sshuser должен обладать максимальными привилегиями.

### Вариант реализации:

### SRV1-DT | SRV2-DT | SRV3-DT | SW1-HQ | SW2-HQ | SW3-HQ    | SRV1-HQ:

- Для создания пользователя **sshuser** используем утилиту [useradd](https://www.opennet.ru/man.shtml?topic=useradd&category=8&russian=0):
    - где:
        - **useradd** - утилита для создания пользователя;
        - **sshuser** - имя пользователя;
        - **-m** - если домашнего каталога пользователя не существует, то он будет создан;
        - **-U** - создаётся одноимённая группа и пользователь автоматически в неё добавляется;
        - **-s /bin/bash** - задаётся командный интерпретатор для пользователя:

```
useradd sshuser -m -U -s /bin/bash
```

- Проверяем созданного пользователя с необходимыми параметрами:
    - для этого необходимо открыть содержимое файла **/etc/passwd** с помощью утилиты **cat** или же текстовым редактором, например: **vim**;
    - или же использовать утилиту [grep](https://ru.wikipedia.org/wiki/Grep) и передать ей в качестве значения имя пользователя:

```
grep sshuser /etc/passwd
```

- - Результат на примере для **SRV1-HQ****:**

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image.png)

_Аналогично для всех остальных устройств_

- Для назначения пользователя **sshuser** пароля **P@ssw0rd** используем утилиту [passwd](https://www.opennet.ru/man.shtml?topic=passwd&category=1):
    - во время запуска - утилита в интерактивном режиме попросит ввести пароль для пользователя и затем подтвердить его:

```
passwd sshuser
```

- - Результат запуска утилиты:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%281%29.png)

- Проверяем:
    - на **SRV-HQ1** - выполняем вход из под пользователя **sshuser** с паролем **P@ssw0rd**:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%282%29.png)

Аналогично для всез остальных устройств_:_ SRV1-DT, SRV2-DT, SRV3-DT, SW1-HQ, SW2-HQ, SW3-HQ.

- Реализуем возможность запуска утилиты sudo пользователю sshuser без ввода пароля:
- [Рекумендуется прочитать перед выполнением](https://www.altlinux.org/Sudo)

- Добавляем пользователя **sshuser** в группу **wheel** для этого используем утилиту [usermod](https://www.opennet.ru/man.shtml?topic=usermod&category=8&russian=0)
    - поскольку штатное состояние политики: **wheelonly** (Означает что пользователь из группы wheel имеет право запускать саму команду sudo, но не означает, что он через sudo может выполнить какую-то команду с правами root)
    - где:
        - **usermod** - утилита для изменения и работы с параметрами пользователя;
        - **-aG** - параметр чтобы добавить пользователя в дополнительную группу(ы). Использовать только вместе с параметром **-G;**
        - **wheel** - имя группы;
        - **sshuser** - имя пользователя:

```
usermod -aG wheel sshuser
```

- Добавляем следующую строку в файл в [/etc/sudoers](https://www.opennet.ru/man.shtml?topic=sudoers&category=5&russian=0) чтобы была возможность запуска **sudo** без дополнительной аутентификации:

```
echo "sshuser ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers
```

- - **P.S.** или же открываем через текстовы редактор, например: **vim**
- Проверяем:
    - на **SRV-HQ1** выполняем вход из под пользователя **sshuser** и пытаемся повысить привелегии:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%283%29.png)

Аналогично для всез остальных устройств_:_ SRV1-DT, SRV2-DT, SRV3-DT, SW1-HQ, SW2-HQ, SW3-HQ.

### R-DT | R-HQ:

- Создаём пользователя **sshuser** на маршрутизаторах с паролем **P@ssw0rd** и с максимальными привилегиями:
    - максимальным привилегиям в EcoRouter - соответствуем роль **admin**:

```
r-hq#configure terminal
r-hq(config)#username sshuser
r-hq(config-user)#password P@ssw0rd
r-hq(config-user)#role admin 
r-hq(config-user)#exit
r-hq(config)#write
r-hq(config)#
```

- Проверяем:
    - на **R-DT** выполняем вход из под пользователя **sshuser**:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%284%29.png)

- - на **R-HQ** выполняем вход из под пользователя **sshuser**:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%285%29.png)

### CLI | ADMIN-DT | CLI-DT | ADMIN-HQ | CLI-HQ:

- Поскольку клиенты имеют графический интерфейс - воспользуемся Центром Управления Системой (**ЦУС**):

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%286%29.png)

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%287%29.png)

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%288%29.png)

- - Создаём пользователя **sshuser**:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%2810%29.png)

- Задаём пользователю **sshuser** пароль **P@ssw0rd**:
- - - Добавляем его в группу **wheel** - выставив чек-бокс **Входит в группу администраторов**

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%2811%29.png)

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%2812%29.png)

- Устанавливаем пакет **sudo**:
    - для доступа в сеть Интернет можно временно использовать публичный DNS (**echo 'nameserver 77.88.8.8' > /etc/resolv.conf**)

```
apt-get update && apt-get install -y sudo
```

- Добавляем следующую строку в файл в [/etc/sudoers](https://www.opennet.ru/man.shtml?topic=sudoers&category=5&russian=0) чтобы была возможность запуска **sudo** без дополнительной аутентификации:

```
echo "sshuser ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/814/mod_page/content/2/image%20%2813%29.png)

Аналогично для всез остальных устройств: ADMIN-DT, CLI-DT, ADMIN-HQ, CLI-HQ.

# Настройка DNS для SRV1-HQ и SRV1-DT (основной DNS сервер)

a) Реализуйте основной DNS сервер компании на SRV1-HQ

- 1. Для всех устройств обоих офисов необходимо создать записи A и PTR.
- 2. Для всех сервисов предприятия необходимо создать записи CNAME.
- 3. Загрузка записей с SRV1-HQ должна быть разрешена только для SRV1-DT

d) В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер

### Вариант реализации:

Записи типа A, PTR и CNAME - будут создаваться после установки контроллера домена, средствами samba-tool.

Данный процесс рассмотрет в [Добавление всех необходимые записей типа A, PTR и CNAME средствами samba-tool](https://sysahelper.ru/mod/page/view.php?id=502)

### SRV1-HQ:

- Для установки необходимых пакетов, временно установим в качестве DNS публичный адрес:

```
echo "nameserver 77.88.8.8" > /etc/resolv.conf
```

- Установим необходимые пакеты:

```
apt-get update && apt-get install -y bind bind-utils
```

- Правим конфигурационный файл **/etc/bind/options.conf**:

```
vim /etc/bind/options.conf
```

- - вносим следующие изменения:
        - **listen-on** - Позволяет указать сетевые интерфейсы, которые будет прослушивать служба;
        - **listen-on-v6** - Раз IPv6 не используется, тогда не используем;
        - **forwarders** - DNS-сервер, на который будут перенаправляться запросы клиентов;
        - **allow-query** - IP-адреса и подсети от которых будут обрабатываться запросы;
        - **allow-transfer** - Устанавливает возможность передачи зон для slave-серверов.

![](https://sysahelper.ru/pluginfile.php/827/mod_page/content/4/image.png)

- Включаем и добавляем в автозагрузку службу **bind**:

```
systemctl enable --now bind
```

- Проверяем:
    - Настраиваем SRV1-HQ на использование в качестве DNS-сервера самого себя:

```
cat <<EOF > /etc/net/ifaces/ens19/resolv.conf
  search au.team
  nameserver 192.168.11.66
EOF
```

- - Перезагружаем службы **network** и **bind**:

```
systemctl restart network
```

```
systemctl restart bind
```

- - Проверяем доступ в сеть Интернет:

![](https://sysahelper.ru/pluginfile.php/827/mod_page/content/4/image%20%281%29.png)

# Настройка DNS для SRV1-HQ и SRV1-DT (резервный DNS сервер)

b) Сконфигурируйте SRV1-DT, как резервный DNS сервер

d) В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер

### Вариант реализации:

### SRV1-DT:

- Для установки необходимых пакетов, временно установим в качестве DNS публичный адрес:

```
echo "nameserver 77.88.8.8" > /etc/resolv.conf
```

- Установим необходимые пакеты:

```
apt-get update && apt-get install -y bind bind-utils
```

- Правим конфигурационный файл **/etc/bind/options.conf**:

```
vim /etc/bind/options.conf
```

- - Вносим следующие изменения:
        - Основные параметры описаны в [Настройка DNS для SRV1-HQ и SRV1-DT (основной DNS сервер)](https://sysahelper.ru/mod/page/view.php?id=476)

![](https://sysahelper.ru/pluginfile.php/828/mod_page/content/3/image.png)

- Чтобы bind работал в режиме SLAVE, нужно настроить control:

```
control bind-slave enabled
```

- Задаём настройки для внутреннего DNS:

```
cat <<EOF > /etc/net/ifaces/ens19/resolv.conf
  search au.team
  nameserver 192.168.33.66
  nameserver 192.168.11.66
EOF
```

- Перезагружаем службу **network**:

```
systemctl restart network
```

- Включаем и добавляем в автозагрузку службу **bind**:

```
systemctl enable --now bind
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/828/mod_page/content/3/image%20%281%29.png)

# Настройка DNS для SRV1-HQ и SRV1-DT (устройства должны быть настроены)

c) Все устройства должны быть настроены на использование обоих внутренних DNS серверов.

- 1. Для офиса HQ основным DNS сервером является SRV1-HQ
- 2. Для офиса DT основным DNS сервером является SRV1-DT

### Вариант реализации:

### SRV2-DT | SRV3-DT | ADMIN-DT:

- Задаём настройки DNS:

```
cat <<EOF > /etc/net/ifaces/ens19/resolv.conf
  search au.team
  nameserver 192.168.33.66
  nameserver 192.168.11.66
EOF
```

- Перезагружаем службу **network**:

```
systemctl restart network
```

### SW1-HQ | SW2-HQ | SW3-HQ:

- Задаём настройки DNS:

```
cat <<EOF > /etc/net/ifaces/MGMT/resolv.conf
  search au.team
  nameserver 192.168.11.66
  nameserver 192.168.33.66
EOF
```

- Перезагружаем службу **network**:

```
systemctl restart network
```

### ADMIN-HQ:

- Задаём настройки DNS:

```
cat <<EOF > /etc/net/ifaces/ens19/resolv.conf
  search au.team
  nameserver 192.168.11.66
  nameserver 192.168.33.66
EOF
```

- Перезагружаем службу **network**:

```
systemctl restart network
```

### R-HQ:

- Задаём настройки DNS:

```
r-hq#configure terminal
r-hq(config)#ip name-server 192.168.11.66 192.168.33.66
r-hq(config)#ip domain-name au.team
r-hq(config)#write
r-hq(config)#
```

### R-DT:

- Задаём настройки DNS:

```
r-dt#configure terminal
r-dt(config)#ip name-server 192.168.33.66 192.168.11.66
r-dt(config)#ip domain-name au.team
r-dt(config)#write
r-dt(config)#
```
# Настройте синхронизацию времени между сетевыми устройствами по протоколу NTP (SRV1-HQ)

a) В качестве сервера должен выступать SRV1-HQ

- 1. Используйте стратум 5
- 2. Используйте ntp2.vniiftri.ru в качестве внешнего сервера синхронизации времени

c) Используйте на всех устройствах московский часовой пояс.

### Вариант реализации:

### SRV1-HQ:

- Редактируем конфигурационный файл  **chrony**:

```
vim /etc/chrony.conf
```

- - Приводим его к следующему виду:

![](https://sysahelper.ru/pluginfile.php/830/mod_page/content/5/image%20%282%29.png)

- Перезагружаем службу **chronyd** для применения изменений:

```
systemctl restart chronyd
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/830/mod_page/content/5/image%20%281%29.png)

# Настройте синхронизацию времени между сетевыми устройствами по протоколу NTP (устройства должны синхронизировать)

b) Все устройства должны синхронизировать своё время с SRV1-HQ.

- 1. Используйте chrony, где это возможно

c) Используйте на всех устройствах московский часовой пояс.

### Вариант реализации:

### SW1-HQ | SW2-HQ | SW3-HQ | SRV1-DT | SRV2-DT | SRV3-DT:

- Редактируем конфигурационный файл  **chrony**:

```
vim /etc/chrony.conf
```

- - Приводим его к следующему виду:

![](https://sysahelper.ru/pluginfile.php/831/mod_page/content/6/image.png)

- Перезагружаем службу **chronyd** для применения изменений:

```
systemctl restart chronyd
```

- Проверяем наличие клиентов на **SRV1-HQ**:

![](https://sysahelper.ru/pluginfile.php/831/mod_page/content/6/image%20%281%29.png)

### ADMIN-HQ | ADMIN-DT | CLI-HQ | CLI-DT:

![](https://sysahelper.ru/pluginfile.php/831/mod_page/content/6/image%20%282%29.png)

- Проверяем наличие клиентов на **SRV1-HQ**:

![](https://sysahelper.ru/pluginfile.php/831/mod_page/content/6/image%20%283%29.png)

### R-HQ | R-DT:

```
r-hq#configure terminal
r-hq(config)#ntp server 192.168.11.66
r-hq(config)#ntp timezone UTC+3
r-hq(config)#write
r-hq(config)#
```

### FW-DT:

![](https://sysahelper.ru/pluginfile.php/831/mod_page/content/6/image%20%284%29.png)

![](https://sysahelper.ru/pluginfile.php/831/mod_page/content/6/image%20%285%29.png)

- Проверяем наличие клиентов на **SRV1-HQ**:

![](https://sysahelper.ru/pluginfile.php/831/mod_page/content/6/image%20%286%29.png)

# Реализация доменной инфраструктуры SAMBA AD (основной доменный контроллер)

### Задание:

a) Сконфигурируйте основной доменный контроллер на SRV1-HQ

- 1. Используйте модуль BIND9_DLZ

### Вариант реализации:

### SRV1-HQ:

- Установим необходимый пакет:

```
apt-get install task-samba-dc -y
```

- ⁠ Настройка BIND9 для работы с Samba AD:  
    - Отключаем chroot:

```
control bind-chroot disabled
```

- - Отключаем KRB5RCACHETYPE:

```
grep -q KRB5RCACHETYPE /etc/sysconfig/bind || echo 'KRB5RCACHETYPE="none"' >> /etc/sysconfig/bind
```

- - Подключаем плагин BIND_DLZ:

```
grep -q 'bind-dns' /etc/bind/named.conf || echo 'include "/var/lib/samba/bind-dns/named.conf";' >> /etc/bind/named.conf
```

- - Отредактируем файл **/etc/bind/options.conf**:

```
vim /etc/bind/options.conf
```

- - - В раздел **options** необходимо добавить строки:

![](https://sysahelper.ru/pluginfile.php/832/mod_page/content/2/image.png)

- - - В раздел **logging** необходимо добавить строку:

![](https://sysahelper.ru/pluginfile.php/832/mod_page/content/2/image%20%281%29.png)

- Выполняем остановку службы **bind**:

```
systemctl stop bind
```

- Необходимо очистить базы и конфигурацию Samba:

```
rm -f /etc/samba/smb.conf
rm -rf /var/lib/samba
rm -rf /var/cache/samba
mkdir -p /var/lib/samba/sysvol
```

- Запускаем интерактивную установку контроллера домена:

```
samba-tool domain provision
```

- - Результат:
        - Все параметры за исключением **DNS backend** должны подставляться автоматически корректными

![](https://sysahelper.ru/pluginfile.php/832/mod_page/content/2/image%20%282%29.png)

- - Результат:

![](https://sysahelper.ru/pluginfile.php/832/mod_page/content/2/image%20%283%29.png)

- Запускаем службы **samba** и **bind**:

```
systemctl enable --now samba
```

```
systemctl start bind
```

- Настраиваем Kerberos:

```
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/832/mod_page/content/2/image%20%284%29.png)

# Добавление всех необходимые записей типа A, PTR и CNAME средствами samba-tool

### Добавление всех необходимые записей типа A, PTR и CNAME средствами samba-tool

### SRV1-HQ:

- Добавляем все необходимые записи типа **A, PTR** и **CNAME** средствами **samba-tool**
    - Добавляем записи типа **А**:

```
samba-tool dns add 127.0.0.1 au.team r-dt A 192.168.33.89
samba-tool dns add 127.0.0.1 au.team fw-dt A 192.168.33.90
samba-tool dns add 127.0.0.1 au.team fw-dt A 192.168.33.1
samba-tool dns add 127.0.0.1 au.team fw-dt A 192.168.33.65
samba-tool dns add 127.0.0.1 au.team fw-dt A 192.168.33.81
samba-tool dns add 127.0.0.1 au.team admin-dt A 192.168.33.82
samba-tool dns add 127.0.0.1 au.team srv1-dt A 192.168.33.66
samba-tool dns add 127.0.0.1 au.team srv2-dt A 192.168.33.67
samba-tool dns add 127.0.0.1 au.team srv3-dt A 192.168.33.68
samba-tool dns add 127.0.0.1 au.team cli-dt A 192.168.33.2

samba-tool dns add 127.0.0.1 au.team r-hq A 192.168.11.1
samba-tool dns add 127.0.0.1 au.team r-hq A 192.168.11.65
samba-tool dns add 127.0.0.1 au.team r-hq A 192.168.11.81
samba-tool dns add 127.0.0.1 au.team sw1-hq A 192.168.11.82
samba-tool dns add 127.0.0.1 au.team sw2-hq A 192.168.11.83
samba-tool dns add 127.0.0.1 au.team sw3-hq A 192.168.11.84
samba-tool dns add 127.0.0.1 au.team admin-hq A 192.168.11.85
samba-tool dns add 127.0.0.1 au.team cli-hq A 192.168.11.2
```

- Проверяем:

```
samba-tool dns query 127.0.0.1 au.team @ A
```

- - Результат:

![](https://sysahelper.ru/pluginfile.php/860/mod_page/content/3/image.png)

- Создаём зоны обратного просмотра для добавления PTR-записей:
    - Для сети офиса **HQ**:

```
samba-tool dns zonecreate 127.0.0.1 11.168.192.in-addr.arpa
```

- - Для сети офиса **DT**:

```
samba-tool dns zonecreate 127.0.0.1 33.168.192.in-addr.arpa
```

- - Проверяем:

```
samba-tool dns zonelist 127.0.0.1
```

- - - Результат:

![](https://sysahelper.ru/pluginfile.php/860/mod_page/content/3/image%20%281%29.png)

- Добавляем все необходимые записи типа **A, PTR** и **CNAME** средствами **samba-tool**
    - Добавляем записи типа **PTR**:

```
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 89 PTR r-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 90 PTR fw-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 1 PTR fw-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 65 PTR fw-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 81 PTR fw-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 82 PTR admin-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 66 PTR srv1-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 67 PTR srv2-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 68 PTR srv3-dt.au.team
samba-tool dns add 127.0.0.1 33.168.192.in-addr.arpa 2 PTR cli-dt.au.team

samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 1 PTR r-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 65 PTR r-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 66 PTR srv1-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 81 PTR r-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 82 PTR sw1-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 83 PTR sw2-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 84 PTR sw3-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 85 PTR admin-hq.au.team
samba-tool dns add 127.0.0.1 11.168.192.in-addr.arpa 2 PTR cli-hq.au.team
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/860/mod_page/content/3/image%20%282%29.png)

- Добавляем записи типа **CNAME** - для необходимых сервисов:

```
samba-tool dns add 127.0.0.1 au.team www CNAME srv1-dt.au.team -U administrator
```

```
samba-tool dns add 127.0.0.1 au.team zabbix CNAME srv1-dt.au.team -U administrator
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/860/mod_page/content/3/image%20%284%29.png)

# Реализация доменной инфраструктуры SAMBA AD (пользователи, группы, подразделения)

### Задание:

2. Создайте 30 пользователей user1-user30 с паролем P@ssw0rd.

3. Пользователи user1-user10 должны входить в состав группы group1.

4. Пользователи user11-user20 должны входить в состав группы group2.

5. Пользователи user21-user30 должны входить в состав группы group3.

6. Создайте подразделения CLI и ADMIN

- i. Поместите клиентов в подразделения в зависимости от их роли.

### Вариант реализации:

### SRV1-HQ:

- Создаём группы **group1, group2** и **group3**:

```
samba-tool group add group1
```

```
samba-tool group add group2
```

```
samba-tool group add group3
```

- - Проверяем:

![](https://sysahelper.ru/pluginfile.php/833/mod_page/content/3/image.png)

- Создаём пользователей **user1-user30** с паролем **P@ssw0rd**:
    - Создаём пользователей **user1-user10** - и добавляем в группу **group1**:

```
for i in {1..10}; do
  samba-tool user add user$i P@ssw0rd;
  samba-tool user setexpiry user$i --noexpiry;
  samba-tool group addmembers "group1" user$i;
done
```

- - Создаём пользователей **user11-user20** - и добавляем в группу **group2**:

```
for i in {11..20}; do
  samba-tool user add user$i P@ssw0rd;
  samba-tool user setexpiry user$i --noexpiry;
  samba-tool group addmembers "group2" user$i;
done
```

- - Создаём пользователей **user21-user30** - и добавляем в группу **group3**:

```
for i in {21..30}; do
  samba-tool user add user$i P@ssw0rd;
  samba-tool user setexpiry user$i --noexpiry;
  samba-tool group addmembers "group3" user$i;
done
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/833/mod_page/content/3/image%20%281%29.png)

- Создим подразделения **CLI** и **ADMIN**:

```
samba-tool ou add 'OU=CLI'
samba-tool ou add 'OU=ADMIN'
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/833/mod_page/content/3/image%20%282%29.png)

# Реализация доменной инфраструктуры SAMBA AD (резервный контроллер домена)

### Задание:

f) В качестве резервного контроллера домена используйте SRV1-DT.

1. Используйте модуль BIND9_DLZ

### Вариант реализации:

### SRV1-DT:

- Реализуем резервный контроллер домена с модулем **BIND9_DLZ**:
    - Установим необходимый пакет:

```
apt-get install task-samba-dc -y
```

- - Останавливаем конфликтующие службы **krb5kdc** и **slapd**, а также **bind**:

```
for service in smb nmb krb5kdc slapd bind; do 
  systemctl disable $service; 
  systemctl stop $service; 
done
```

- - Отключаем KRB5RCACHETYPE:

```
grep -q KRB5RCACHETYPE /etc/sysconfig/bind || echo 'KRB5RCACHETYPE="none"' >> /etc/sysconfig/bind
```

- - Подключаем плагин BIND_DLZ:

```
grep -q 'bind-dns' /etc/bind/named.conf || echo 'include "/var/lib/samba/bind-dns/named.conf";' >> /etc/bind/named.conf
```

- - Отредактируем файл **/etc/bind/options.conf**:

```
vim /etc/bind/options.conf
```

- - - В раздел **options** необходимо добавить строки:

![](https://sysahelper.ru/pluginfile.php/834/mod_page/content/4/image.png)

- - - В раздел **logging** необходимо добавить строку:

![](https://sysahelper.ru/pluginfile.php/834/mod_page/content/4/image%20%281%29.png)

- - Установим следующие параметры в файле конфигурации клиента Kerberos:

```
vim /etc/krb5.conf
```

- - - Содержимое:

![](https://sysahelper.ru/pluginfile.php/834/mod_page/content/4/image%20%282%29.png)

- - Запросим билет Kerberos администратора домена: 

```
kinit administrator@AU.TEAM
```

- - - Проверяем:

![](https://sysahelper.ru/pluginfile.php/834/mod_page/content/4/image%20%283%29.png)

_P.S. предварительно на FW-DT должен быть отключён перехват пользовательских DNS запросов_

- Необходимо очистить базы и конфигурацию Samba:

```
rm -f /etc/samba/smb.conf
rm -rf /var/lib/samba
rm -rf /var/cache/samba
mkdir -p /var/lib/samba/sysvol
```

- Вводим **SRV1-DT** в домен **au.team** в качестве контроллера домена: 

```
samba-tool domain join au.team DC -Uadministrator --realm=au.team --dns-backend=BIND9_DLZ
```

- Включаем и добавляем в автозагрузку службы **samba** и **bind**:

```
systemctl enable --now samba
```

```
systemctl enable --now bind
```

- Выполняем процедуру репликации - с первого контроллера домена на второй:

```
samba-tool drs replicate srv1-dt.au.team srv1-hq.au.team dc=au,dc=team -Uadministrator
```

- Выполняем процедуру репликации на первый контроллер домена со второго:

```
samba-tool drs replicate srv1-hq.au.team srv1-dt.au.team dc=au,dc=team -Uadministrator
```

- - Результат:

![](https://sysahelper.ru/pluginfile.php/834/mod_page/content/4/image%20%284%29.png)

- Проверяем репликацию:

![](https://sysahelper.ru/pluginfile.php/834/mod_page/content/4/image%20%285%29.png)

# Реализация доменной инфраструктуры SAMBA AD (Ввод клиентов в домен)

7. Клиентами домена являются ADMIN-DT, CLI-DT, ADMIN-HQ, CLI-HQ.

### Вариант реализации:

### ADMIN-HQ:

- Вводим в домен:

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%283%29.png)

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%284%29.png)

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%285%29.png)

#### 8 - Перезагрузить виртуальную машину

### CLI-HQ:

- Вводим в домен аналогично **ADMIN-HQ**:

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%286%29.png)

### ADMIN-DT:

- Вводим в домен аналогично **ADMIN-HQ**:

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%287%29.png)

### CLI-DT:

- Вводим в домен аналогично **ADMIN-HQ**:

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%288%29.png)

### SRV1-HQ:

- Проверяем наличие клиентов в домене:

```
samba-tool computer list
```

- - Результат:

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%289%29.png)

- Перемещаем клинетов в подразредения:
    - **ADMIN-DT** в подразделение **ADMIN**:

```
samba-tool computer move ADMIN-DT 'OU=ADMIN,DC=au,DC=team'
```

- - **ADMIN-HQ** в подразделение **ADMIN**:

```
samba-tool computer move ADMIN-HQ 'OU=ADMIN,DC=au,DC=team'
```

- - **CLI-DT** в подразделение **CLI**:

```
samba-tool computer move CLI-DT 'OU=CLI,DC=au,DC=team'
```

- - **CLI-HQ** в подразделение **CLI**:

```
samba-tool computer move CLI-HQ 'OU=CLI,DC=au,DC=team'
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/861/mod_page/content/5/image%20%2810%29.png)

# Реализация доменной инфраструктуры SAMBA AD (общая папка)

### Задание:

h) Реализуйте общую папку на SRV1-HQ

- 1. Используйте название SAMBA
- 2. Используйте расположение /opt/data

### Вариант реализации:

### SRV1-HQ:

- Создаём директорию для общей папки:

```
mkdir /opt/data
```

- Задаём права на директорию:

```
chmod 777 /opt/data
```

- Реализуем общую папку, вносим следующую информацию в конфигурационный файл **/etc/samba/smb.conf**:

![](https://sysahelper.ru/pluginfile.php/835/mod_page/content/5/image.png)

- Перезагружаем службу **samba**:

```
systemctl restart samba
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/835/mod_page/content/5/image%20%281%29.png)

# Реализация бекапа общей папки на сервере SRV1-HQ с использованием systemctl (юнит типа service)

a) Бекап должен архивировать все данные в формат tar.gz и хранить в директории /var/bac/.

- 1. Архивация должна производиться благодаря юниту типа service с названием backup.
- 2. Сервис должен включатся автоматический при загрузке.

### Вариант реализации:

### SRV1-HQ:

- Создаём директорию для хранения бэкапа общей папки:

```
mkdir /var/bac/
```

- Создаём юнит типа **service** с названием **backup**:

```
vim /etc/systemd/system/backup.service
```

- - Помещаем в данный файл - следующее содержимое, где:
        - **Description** - описание юнита;
        - **Type** - тип юнита (очень важный параметр, **oneshot** — если подразумевается разовый запуск утилиты или скрипта, то подойдет этот тип)
        - **ExecStart** - команда, которая запускает службу. Именно в этом параметре нужно указать главный исполняемый файл (утилиту или скрипт), ради которого мы создаём службу
        - **WantedBy** - если мы включим автозагрузку этой службы (с помощью команды **systemctl enable <имя службы>**), то она должна запуститься при загрузке мультипользовательского режима (**multi-user.target**)

![](https://sysahelper.ru/pluginfile.php/784/mod_page/content/2/%D0%B8%D0%B7%D0%BE%D0%B1%D1%80%D0%B0%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5.png)

- Выполним команду которая запустит перезагрузку и перечитывание всех конфигурационных файлов для юнитов:

```
systemctl daemon-reload
```

- Включаем и добавляем в автозагрузку созданные юнит **backup.service**:

```
systemctl enable --now backup.service
```

- После запуска и добавления в автозигрузку службы **backup** должен создаться архив, а служба перейти в состояние **inactive,** но добавлена в автозагрузку, чтобы при перезагрузки автоматически создавалась резервная копия:

![](https://sysahelper.ru/pluginfile.php/839/mod_page/content/3/image.png)

# Реализация бекапа общей папки на сервере SRV1-HQ с использованием systemctl (юнит типа timer)

b) Время выполнение бекапа каждый день в 8 часов вечера.

- 1. Используйте юнит типа timer для выполнения.
- 2. Если устройство будет выключено, то архивация производится сразу после запуска.

### Вариант реализации:

### SRV1-HQ:

- Создаём юнит типа **service** с названием **timer**:

```
vim /etc/systemd/system/backup.timer
```

- - Помещаем в данный файл - следующее содержимое, где:
        - **Description** - описание юнита;
        - **OnCalendar** - представления события календаря, в данном случае подразумевается каждый день (*) каждого месяца (*) каждого года (*) в 20 часов 00 минут 00 секунд
        - **Persistent** - указывает запускать таймер немедленно, если был пропущен предыдущий запуск
        - **Unit** - указывае какой юнит следует запускать

![](https://sysahelper.ru/pluginfile.php/840/mod_page/content/3/image.png)

- Выполним команду которая запустит перезагрузку и перечитывание всех конфигурационных файлов для юнитов:

```
systemctl daemon-reload
```

- Включаем и добавляем в автозагрузку созданные юнит **backup.timer**:

```
systemctl enable --now backup.timer
```

- Проверяем статус юнита:

![](https://sysahelper.ru/pluginfile.php/840/mod_page/content/3/image%20%281%29.png)

# Управление доменом с помощью ADMC (изменения рабочего стола)

a) Управление доменом с помощью ADMC осуществляться с ADMIN-HQ

b) Для подразделения CLI настройте политику изменения рабочего стола на картинку компании, а также запретите использование пользователям изменение сетевых настроек и изменение графических параметров рабочего стола.

### Вариант реализации:

### ADMIN-HQ | ADMIN-DT | CLI-HQ | CLI-DT:

- Необходимо установить пакет **gpupdate**:

```
apt-get update && apt-get install -y gpupdate
```

- Включить модуль групповых политик:

```
gpupdate-setup enable
```

### ADMIN-HQ:

- Установим пакет **admc**:

```
apt-get update && apt-get install -y admc
```

- Проверяем:
    - Получаем билет Kerberos (из под обычного пользователя):

```
kinit administrator@AU.TEAM
```

- - Запускаем оснастку ADMC:

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image.png)

- Установим пакет **gpui**, для редактирования настроек клиентской конфигурации:

```
apt-get install -y gpui
```

- В оснастке ADMC - переходим в раздел **Объекты групповой политики** - выбираем подразделение **CLI** и нажимаем **Создать политику и связать с этом подразделением**:

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%281%29.png)

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%282%29.png)

- Выбираем созданную групповую политику и нажимаем **Изменить**:

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%283%29.png)

- - В соответствие с требованиями задания - реализуем необходимый функционал:
        - Задаём картинку компании для рабочего стола и запрещаем её менять

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%284%29.png)

- - - Запрещаем изменение сетевых настроек

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%285%29.png)

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%286%29.png)

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%288%29.png)

- - опционально пройтись по всему списку и выставить **Включено**, с вариантом ограничений **No**:

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%289%29.png)

- Проверяем:
    - перезагружаем **CLI-HQ** и **CLI-DT**:
        - картинка фона рабочего стола должна быть установлена:

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%2810%29.png)

- - При попытке отключить сетевой интерфейс:

![](https://sysahelper.ru/pluginfile.php/836/mod_page/content/2/image%20%2811%29.png)

# Управление доменом с помощью ADMC (подключение общей папки)

a) Управление доменом с помощью ADMC осуществляться с ADMIN-HQ

c) Для подразделения ADMIN реализуйте подключение общей папки SAMBA с использованием доменных политик.

### Вариант реализации:

### ADMIN-HQ:

- В оснастке ADMC - переходим в раздел **Объекты групповой политики** - выбираем подразделение **ADMIN** и нажимаем **Создать политику и связать с этом подразделением**:

![](https://sysahelper.ru/pluginfile.php/837/mod_page/content/4/image.png)

- Выбираем созданную групповую политику и нажимаем **Изменить**:

![](https://sysahelper.ru/pluginfile.php/837/mod_page/content/4/image%20%281%29.png)

- - В соответствие с требованиями задания - реализуем необходимый функционал:

![](https://sysahelper.ru/pluginfile.php/837/mod_page/content/4/image%20%285%29.png)

- - - Результат:

![](https://sysahelper.ru/pluginfile.php/837/mod_page/content/4/image%20%286%29.png)

- Проверяем:
    - перезагружаем **ADMIN-DT** и **ADMIN-HQ** - должен подключиться автоматически сетевой диск:

![](https://sysahelper.ru/pluginfile.php/837/mod_page/content/4/image%20%287%29.png)

![](https://sysahelper.ru/pluginfile.php/837/mod_page/content/4/image%20%288%29.png)

# Развертывание приложений в Docker на SRV2-DT (локальный Docker Registry)

### Задание:

a) Создайте локальный Docker Registry.

### Вариант реализации:

### SRV2-DT:

- Установим пакет для работы с **Docker**:

```
apt-get update && apt-get install -y docker-engine
```

- Запускаем и добавляем в автозагрузку службу **docker**:

```
systemctl enable --now docker.service
```

- Создаём и запускаем локальный **Docker Registry**:
    - Поднимает **контейнер Docker** с именем **DockerRegistry** из образа **registry:2** 
    - Контейнер будет слушать сетевые запросы **на порту 5000**
    - Параметр **--restart=always** позволит автоматически запускаться контейнеру после перезагрузки сервера.

```
docker run -d -p 5000:5000 --restart=always --name DockerRegistry registry:2
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/841/mod_page/content/2/image.png)

# Развертывание приложений в Docker на SRV2-DT (Dockerfile для приложения web)

b) Напишите Dockerfile для приложения web.

- 1. В качестве базового образа используйте nginx:alpine
- 2. Содержание index.html

```
<html>
    <body>
        <center><h1><b>WEB</b></h1></center>
    </body>
</html>
```

- 3. Соберите образ приложения web и загрузите его в ваш Registry.
    - i. Используйте номер версии 1.0 для вашего приложения
    - ii. Образ должен быть доступен для скачивания и дальнейшего запуска на локальной машине

### Вариант реализации:

### SRV2-DT:

- Напишим **Dockerfile** для приложения **web**:

```
vim Dockerfile
```

- - Содержимое, где:
        - **FROM** - задаёт базовый образ;
        - **COPY** - копирует с локального хоста в контейнер:

![](https://sysahelper.ru/pluginfile.php/842/mod_page/content/3/image.png)

- Создаём файл **index.html**:

```
vim index.html
```

- - Содержимое по требованию задания:

![](https://sysahelper.ru/pluginfile.php/842/mod_page/content/3/image%20%281%29.png)

- Выполняем сборку docker-образа:
    - **-t** - позволяет присвоить имя собираемому образу;
    - "**.**" - говорит о том что **Dockerfile** находится в текущей директории откуда выполняется данная команда и имеет имя именно **Dockerfile:**

```
docker build -t localhost:5000/web:1.0 .
```

- Результат:

![](https://sysahelper.ru/pluginfile.php/842/mod_page/content/3/image%20%282%29.png)

- Проверяем:
    - Наличие собранного образа

![](https://sysahelper.ru/pluginfile.php/842/mod_page/content/3/image%20%283%29.png)

- Загружаем образ собранный из **Dockerfile** в локальной **DockerRegistry**

```
docker push localhost:5000/web:1.0
```

- - Результат:

![](https://sysahelper.ru/pluginfile.php/842/mod_page/content/3/image%20%284%29.png)

- Проверяем:
    - Возможность загрузки из локального Docker Registry:  
        - Сперва удаляем образ **localhost:5000/web:1.0**

![](https://sysahelper.ru/pluginfile.php/842/mod_page/content/3/image%20%285%29.png)

- - - Загружаем образ приложения **web** из локального Docker Registry:

![](https://sysahelper.ru/pluginfile.php/842/mod_page/content/3/image%20%286%29.png)

# Развертывание приложений в Docker на SRV2-DT (Docker контейнер)

### Задание:

c) Разверните Docker контейнер используя образ из локального Registry.

- 1. Имя контейнера web
- 2. Контейнер должно работать на порту 80
- 3. Обеспечьте запуск контейнера после перезагрузки компьютера

### Вариант реализации:

### SRV2-DT:

- Запускаем **docker**-контейнер:
    - С именем **web** из образа **localhost:5000/web:01**
    - Контейнер будет слушать сетевые запросы на порту **80**, а параметр **--restart=always** позволит автоматически запускаться контейнеру после перезагрузки сервера

```
docker run -d -p 80:80 --restart=always --name web localhost:5000/web:1.0
```

- Проверяем:
    - Запущенный docker-контейнер:

![](https://sysahelper.ru/pluginfile.php/843/mod_page/content/4/image.png)

- - Доступ до веб-сервера и приложения

![](https://sysahelper.ru/pluginfile.php/843/mod_page/content/4/image%20%281%29.png)

- - Или:

![](https://sysahelper.ru/pluginfile.php/843/mod_page/content/4/image%20%282%29.png)

# Настройка системы централизованного мониторинга (используйте Zabbix)

### Задание:

a) В качестве сервера системы централизованного мониторинга используйте SRV3-DT

b) В качестве системы централизованного мониторинга используйте Zabbix

- 1. В качестве сервера баз данных используйте PostgreSQL
    - i. Имя базы данных: zabbix
    - ii. Пользователь базы данных: zabbix
    - iii. Пароль пользователя базы данных: zabbixpwd
- 2. В качестве веб-сервера используйте Apache

c) Система централизованного мониторинга должна быть доступна для внутренних пользователей по адресу http://<IP адрес SRV3-DT>/zabbix

- 1. Администратором системы мониторинга должен быть пользователь Admin с паролем P@ssw0rd
- 2. Часовой пояс по умолчанию должен быть Europe/Moscow

### Вариант реализации:

### SRV3-DT:

- Устанавливаем **необходимые** пакеты:

```
apt-get update && apt-get install -y postgresql16-server zabbix-server-pgsql
```

- Создаём системные базы данных для корректной работы PostgreSQL:

```
/etc/init.d/postgresql initdb
```

- Включаем и добавляем в автозагрузку службу **postgresql**:

```
systemctl enable --now postgresql
```

- Создаём пользоавтеля **zabbix** в базе данных **PostgreSQL**:

```
su - postgres -s /bin/sh -c 'createuser --no-superuser --no-createdb --no-createrole --encrypted --pwprompt zabbix'
```

- - После запуска данной команды - задаём в качестве пароля для пользователя **zabbix** - пароль **zabbixpwd** и подтверждаем его:

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%281%29.png)

- Создаём базу данных с именем **zabbix**:

```
su - postgres -s /bin/sh -c 'createdb -O zabbix zabbix'
```

- Выполняем перезагрузку службы **postgresql**:

```
systemctl restart postgresql
```

- Добавляем в базу данные для веб-интерфейса:

```
su - postgres -s /bin/sh -c 'psql -U zabbix -f /usr/share/doc/zabbix-common-database-pgsql-*/schema.sql zabbix'
```

```
su - postgres -s /bin/sh -c 'psql -U zabbix -f /usr/share/doc/zabbix-common-database-pgsql-*/images.sql zabbix'
```

```
su - postgres -s /bin/sh -c 'psql -U zabbix -f /usr/share/doc/zabbix-common-database-pgsql-*/data.sql zabbix'
```

- Устанавливаем пакеты для веб-сервера **apache2**:

```
apt-get install -y apache2 apache2-mod_php8.2
```

- Включаем и добавляем в автозагрузку службу отвечающую за веб-сервер **apache2**:

```
systemctl enable --now httpd2
```

- Установим **PHP** и необходимые модули для корректной работы:

```
apt-get install -y php8.2 php8.2-{mbstring,sockets,gd,xmlreader,pgsql,ldap,openssl}
```

- Меняем некоторые опции **php** в файле **/etc/php/8.2/apache2-mod_php/php.ini**: 

```
vim /etc/php/8.2/apache2-mod_php/php.ini
```

- - Находим следующие параметры и приводим их к следующему виду:

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%282%29.png)

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%283%29.png)

- Перезапускаем службу отвечающую за веб-сервер **apache2**:

```
systemctl restrart httpd2
```

- Вносим изменения в конфигурационный файл **/etc/zabbix/zabbix_server.conf:**

```
vim /etc/zabbix/zabbix_server.conf
```

- - Добавляем следующие изменения:

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%284%29.png)

- Добавим **Zabbix-сервер** в автозапуск и запустить его:

```
systemctl enable --now zabbix_pgsql
```

- Установим пакет с веб-интерфейсом Zabbix:

```
apt-get install zabbix-phpfrontend-{apache2,php8.2} -y
```

- Включаем аддоны в **apache2**: 

```
ln -s /etc/httpd2/conf/addon.d/A.zabbix.conf /etc/httpd2/conf/extra-enabled/
```

- Изменяем права доступа к конфигурационному каталогу веб-интерфейса, чтобы веб-установщик мог записать конфигурационный файл: 

```
chown apache2:apache2 /var/www/webapps/zabbix/ui/conf
```

- Перезапускаем службу отвечающую за веб-сервер **apache2**:

```
systemctl restrart httpd2
```

- Далее установка производится средствами веб-интерфейса:
    - Например с **ADMIN-DT** в браузере перейти на страницу установки Zabbix сервера **http://<IP адрес SRV3-DT>/zabbix**: 

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%285%29.png)

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%286%29.png)

1. В качестве базы данных выбираем - **PostgreSQL**;
2. В качестве сервера базы данных указываем **IP** - адрес или имя **localhost;**
3. Указываем имя созданной базы данных **zabbix;**
4. Указываем имя созданного пользователя **zabbix;**
5. Указываем пароль для пользователя zabbix  **zabbixpwd**;
6. Нажимаем **Next step**

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%287%29.png)

- При необходимости задаём имя серверу и нажимаем **Next step**

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%288%29.png)

- Проверяем введённые ранее параметры и нажимаем **Next step**

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%289%29.png)

- Нажимаем **Finish**

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%2810%29.png)

- Выполняем вход из под пользователя по умолчанию: **Admin** с паролем: **zabbix**

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%2812%29.png)

- В качестве пароля для пользователя **Admin -** необходимо установить **P@ssw0rd:**
    - переходим в настройки аутентификации и снимаем галочку, которая запрещает использование слабых паролей

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%2813%29.png)

- Задаём новый пароль **P@ssw0rd** - для пользователя **Admin**

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%2814%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/844/mod_page/content/2/image%20%2815%29.png)

# Настройка системы централизованного мониторинга (узел системы централизованного мониторинга)

### Задание:

d) Настройте узел системы централизованного мониторинга

- 1. В качестве узлов сети используйте устройства SRV1-DT, SRV2-DT, SRV3-DT, SRV1-HQ.
- 2. Имя узла сети должно соответствовать полному имени устройства

### Вариант реализации:

### SRV1-DT | SRV2-DT | SRV3-DT | SRV1-HQ:

- Устанавливаем необходимый пакет **zabbix-agent**:

```
apt-get install zabbix-agent -y
```

- Редактируем конфигурационный файл **/etc/zabbix/zabbix_agentd.conf:**

```
vim /etc/zabbix/zabbix_agentd.conf
```

- - Вносим следующие изменения:
        -  Узказывая IP-адрес **SRV3-DT**:

![](https://sysahelper.ru/pluginfile.php/845/mod_page/content/3/image.png)

-  Включаем и добавляем в автозагрузку службу **zabbix_agentd**:

```
systemctl enable --now zabbix_agentd.service
```

_Аналогично для всех остальных устройств:  SRV1-DT, SRV2-DT, SRV3-DT_

- Каждый хост необходимо зарегистрировать на сервере Zabbix, сделать это можно, используя веб-интерфейс
    - Переходим **Monitoring -> Hosts -> Create host**:

![](https://sysahelper.ru/pluginfile.php/845/mod_page/content/3/image%20%281%29.png)

- Заполняем поля для добавления нового хоста:

![](https://sysahelper.ru/pluginfile.php/845/mod_page/content/3/image%20%282%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/845/mod_page/content/3/image%20%283%29.png)

_Аналогично для всех остальных устройств:  SRV1-DT, SRV2-DT, SRV3-DT_

- Результат:

![](https://sysahelper.ru/pluginfile.php/845/mod_page/content/3/image%20%284%29.png)

# Настройте веб-сервер nginx как обратный прокси-сервер на SRV1-DT

a) При обращении по доменному имени www.au.team, клиента должно перенаправлять на SRV2-DT на контейнер web

b) При обращении по доменному имени zabbix.au.team клиента должно перенаправлять на SRV3-DT на сервис Zabbix

c) Если необходимо, настройте сетевое оборудование для обеспечения работы требуемых сервисов.

### Вариант реализации:

### SRV1-DT:

- Установим пакет **nginx**:

```
apt-get update && apt-get install -y nginx
```

- Создадим конфигурационный файл, в котором опишем необходимые параметры для настройки обратного прокси-сервера:

```
vim /etc/nginx/sites-available.d/proxy.conf
```

- - Содержимое должно быть следующее:

![](https://sysahelper.ru/pluginfile.php/846/mod_page/content/3/image%20%283%29.png)

- Добавляем символьную ссылку на созданный файл:

```
ln -s /etc/nginx/sites-available.d/proxy.conf /etc/nginx/sites-enabled.d/
```

- Проверяем корректность написания конфигурационного файла для обратного прокси-сервера:

![](https://sysahelper.ru/pluginfile.php/846/mod_page/content/3/image%20%281%29.png)

- Включаем и добавляем в автозагрузку службу **nginx**:

```
systemctl enable --now nginx
```

- Проверяем:
    - Доступ по **[http://www.au.team](http://www.au.team/)**:

![](https://sysahelper.ru/pluginfile.php/846/mod_page/content/3/image%20%282%29.png)

- - Доступ по **[http://zabbix.au.team](http://zabbix.au.team/)**:

![](https://sysahelper.ru/pluginfile.php/846/mod_page/content/3/image%20%284%29.png)

# Настройка узла управления Ansible (Инвентарь)

### Задание:

a) Настройте узел управления на базе ADMIN-DT

- 1. Используйте стандартную пакетную версию ansible.

b) Сконфигурируйте инвентарь

- 1. Инвентарь должен располагаться по пути /etc/ansible/inventory.
    - i. Настройте запуск данного инвентаря по умолчанию
- 2. Инвентарь должен содержать три группы устройств:
    - i. Networking (R-DT, R-HQ)
    - ii. Servers (SRV1-HQ, SRV1-DT, SRV2-DT, SRV3-DT)
    - iii. Clients (ADMIN-HQ, ADMIN-DT, CLI-HQ, CLI-DT)

### Вариант реализации:

### ADMIN-DT:

- Установим пакет **ansible**:

```
apt-get update && apt-get install -y ansible sshpass
```

- Назначем необходимые права на директорию **/etc/ansible**:

```
chown -R root:user /etc/ansible
```

```
chmod -R 774 /etc/ansible
```

- Из  под обычного пользователя **user** пероходим для дальнейшей работы в директорию **/etc/ansible**:

```
cd /etc/ansible
```

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/847/mod_page/content/2/image.png)

- Создаём инвентарный файл:

```
vim inventory
```

- - Помещаем в него следующее содержимое:

![](https://sysahelper.ru/pluginfile.php/847/mod_page/content/2/image%20%281%29.png)

- Редактируем конфигурационный файл **ansible.cfg**:

```
vim ansible.cfg
```

- - Вносим следующие изменения:
        - Инвентарь должен располагаться по пути **/etc/ansible/inventory**
        - Отключите проверку **SSH–ключа на хосте**

![](https://sysahelper.ru/pluginfile.php/847/mod_page/content/2/image%20%282%29.png)

# Настройка узла управления Ansible (доступ ко всем устройствам)

3. Реализуйте доступ ко всем устройствам с учетом настроек SSH

- i. Подключение осуществляется по пользователю sshuser
- ii. Используйте корректный интерпретатор Python
- iii. Отключите проверку SSH–ключа на хосте

c) Выполните тестовую команду “ping” средствами ansible

- 1. Убедитесь, что все устройства отвечают “pong” без предупреждающих сообщений
- 2. Убедитесь, что команды ansible выполняются от пользователя user без использования sudo

### Вариант реализации:

### ADMIN-DT:

- Из  под обычного пользователя **user** пероходим для дальнейшей работы в директорию **/etc/ansible**:

```
cd /etc/ansible
```

- Создадим директорию для описания групповых переменных, которые будут использоваться для подключения по SSH:

```
mkdir group_vars
```

- Опишем переменные для каждой группы в соответствующих файлах:
    - Должны быть следующие файлы, со следующими значениями

![](https://sysahelper.ru/pluginfile.php/848/mod_page/content/3/image%20%281%29.png)

- Проверяем:

![](https://sysahelper.ru/pluginfile.php/848/mod_page/content/3/image.png)

# Настройка резервного копирования (установка сервера управления)

1. На ADMIN-HQ развернуть Кибер Бекап 17 версии
    

### Вариант реализации:

### ADMIN-HQ:

- Обновляем систему до актуального состояния и перезагружаем устройство:

```
apt-get update && apt-get dist-upgrade -y && update-kernel -y && apt-get clean && reboot
```

- Должны быть установлены следующие пакеты:
    - где <**x.x**> – версия ядра

```
apt-get install kernel-source-<x.x>
```

```
apt-get install kernel-headers-modules-std-def gcc make -y
```

- Задаём разрешение на исполнение установочному файлу с дистрибутивом Кибер Бэкап:
    - _для этого дистрибутив должен быть заранее скачен и помещён на виртуальную машину_

```
chmod +x CyberBackup_17_64-bit.x86_64
```

- Из под суперпользователя запускаем файл установки:

```
./CyberBackup_17_64-bit.x86_64
```

- Результат:

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image.png)

- Выполняем установку сервера управления для Кибер Бэкап:

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%281%29.png)

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%282%29.png)

- Выбираем необходимые компоненты для установки:

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%283%29.png)

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%284%29.png)

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%285%29.png)

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%286%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%287%29.png)

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%289%29.png)

- Открываем веб-браузер и переходим в веб-интерфейс управления и выполняем вход из под суперпользователя:

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%2810%29.png)

- Активируем пробную версию на 30 дней:

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%2811%29.png)

![](https://sysahelper.ru/pluginfile.php/849/mod_page/content/4/image%20%2812%29.png)

# Настройка резервного копирования (Настройка организации и пользователя)

### Задание:

1. Настроить организацию wsr
    
2. Настроить пользователя с правами администратора на сервере Кибер Бекап wsradmin с паролем P@ssw0rd
    

### Вариант реализации:

### ADMIN-HQ:

- Создаём пользователя **wsradmin** с паролем **P@ssw0rd** на уровне ОС:

![](https://sysahelper.ru/pluginfile.php/862/mod_page/content/6/image%20%2813%29.png)

- Создаём отмел **wsr**:

![](https://sysahelper.ru/pluginfile.php/862/mod_page/content/6/image%20%2814%29.png)

- Добавляем пользователя **wsradmin** в отдел **wsr** с правами **администратора**:

![](https://sysahelper.ru/pluginfile.php/862/mod_page/content/6/image%20%2815%29.png)

![](https://sysahelper.ru/pluginfile.php/862/mod_page/content/6/image%20%2816%29.png)

![](https://sysahelper.ru/pluginfile.php/862/mod_page/content/6/image%20%2817%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/862/mod_page/content/6/image%20%2818%29.png)

- Реализуем возможность для доступа в веб-интерфейс управления из под данного пользователя:

![](https://sysahelper.ru/pluginfile.php/862/mod_page/content/6/image%20%2819%29.png)

# Настройка резервного копирования (установка агента)

### Задание:

d) Установить на CLI-HQ агент Кибер Бекап с функциями узла хранилища и подключить его при помощи токена

### Вариант реализации:

### CLI-HQ:

- Обновляем систему до актуального состояния и перезагружаем устройство:

```
apt-get update && apt-get dist-upgrade -y && update-kernel -y && apt-get clean && reboot
```

- Должны быть установлены следующие пакеты:
    - где <**x.x**> – версия ядра

```
apt-get install kernel-source-<x.x>
```

```
apt-get install kernel-headers-modules-std-def gcc make -y
```

- Задаём разрешение на исполнение установочному файлу с дистрибутивом Кибер Бэкап:
    - _для этого дистрибутив должен быть заранее скачен и помещён на виртуальную машину_

```
chmod +x CyberBackup_17_64-bit.x86_64
```

- Из под суперпользователя запускаем файл установки:

```
./CyberBackup_17_64-bit.x86_64
```

- Результат:

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2819%29.png)

- Выполняем установку агента Кибер Бекап с функциями узла хранилища:

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2820%29.png)

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2821%29.png)

- Выбираем необходимые компоненты для установки:

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2822%29.png)

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2823%29.png)

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2824%29.png)

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2825%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/863/mod_page/content/6/image%20%2826%29.png)

# Настройка резервного копирования (подключить в качестве устройства хранения)

e) Подключить в качестве устройства хранения блочное устройство sdb в формате xfs (устройство должно быть примонтировано в папку /backups)

f) Создать план полного резервного копирования для сервера ADMIN-HQ

g) Выполнить полное резервное копирование ADMIN-HQ на узел хранения

### Вариант реализации:

### CLI-HQ:

- Средствами графического интерфейса создаём на диске **sdb** таблицу разделов, раздел и соответствующую файловую систему:

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2827%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2828%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2829%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2830%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2831%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2832%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2833%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2834%29.png)

- Устройство должно быть примонтировано в папку /backups
    - Создаём необходимую директорию, выдаём соответствующие права:

```
mkdir /backups && chmod 777 /backups
```

- - Реализуем монтирофание:

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2835%29.png)

### ADMIN-HQ:

- В веб-интерфейсе управления реализуем план полного резервного копирования
    - **_в данном случае будет реализован план для CLI-HQ, т.к. на ADMIN-HQ не установлен агент из-за нехватки дискового пространства_**

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2836%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2837%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2839%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2840%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2841%29.png)

- Выполняем полное резервное копирование ADMIN-HQ на узел хранения
    - _**В данном случае выполняется резервное копирование CLI-HQ т.к. на ADMIN-HQ не хватило места на диске для установки агента**_

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2842%29.png)

- Результат:

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2843%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2844%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2845%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2846%29.png)

![](https://sysahelper.ru/pluginfile.php/864/mod_page/content/7/image%20%2847%29.png)

# Настройка межсетевого экрана

1. Сервера и Администраторы офиса DT должны иметь доступ ко всем устройствам

2. Клиенты офиса DT должны иметь доступ только к серверам

3. Разрешите ICMP-запросы администраторами офиса DT на внутренние интерфейсы межсетевого экрана

### Вариант реализации:

### ADMIN-DT:

- Поскольку всё что не запрещено явно, то разрешено, для выполнения данного блока достаточно следующего набора правил для **FORWARD**:
    - **Сервера и Администраторы офиса DT должны иметь доступ ко всем устройствам** - имеют;
    - **Клиенты офиса DT должны иметь доступ только к серверам** - имеют, всё остальное запрещено;
    - **Разрешите ICMP-запросы администраторами офиса DT на внутренние интерфейсы межсетевого экрана** - имеют

![](https://sysahelper.ru/pluginfile.php/838/mod_page/content/2/image.png)

- Или же реализация принципа "всё что не разрешено, то запрещено":
    - **FORWARD:**

![](https://sysahelper.ru/pluginfile.php/838/mod_page/content/2/image%20%281%29.png)

- - **INPUT:**

![](https://sysahelper.ru/pluginfile.php/838/mod_page/content/2/image%20%282%29.png)