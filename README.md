## Запуск сервера

Сервер работает только на Windows, чтобы запустить его введите в cmd:
```
> cd <расположение файла>
```

Для запуска сервера используйте команду:

```
> smarthome.exe -s
```

Дополнительно при запуске в режиме сервера можно указать следующие опции:
* `-p PORT` - номер порта, на котором сервер будет ожидать подключения. По умолчанию используется порт 9998.
* `-0 TIME` - начальное модельное время для симуляции в формате `YYYY-MM-DDThh:mm:ss`, например, `1984-04-01T13:00:00`. По умолчанию - текущее астрономическое время, округленное до секунд.
* `-1 DUR` - продолжительность моделирования, по умолчанию 1 час.
* `-S SCENARIO` - запустить сценарий с указанным номером. Сценарии описаны далее.
* `-V` - выводить принятые и отправленные пакеты в JSON-формате.

Если запуск был успешен, демо-сервер будет ожидать подключения на указанном порту. Проверить это можно, например, с помощью `curl`

```
curl -H POST -d "" localhost:9998
```

Если не сработает команжа выше, попробуйте:

```
Invoke-RestMethod -uri localhost:9998 -method POST -body
```

В ответ вы должны получить закодированную в base64 строку с сообщением с начальным модельным временем. Ответ сервера может выглядеть примерно так:

```
DbMG_38BBgaI0Kv6kzGK
```

## Сценарий работы

В сети находится таймер `TIMER01`, лампа `LAMP02` и выключатель `SWITCH03`, коммутированный с лампой. Выключатель изначально находится в состоянии "выключен" и периодически меняет свое состояние. Таймер шлет сообщения о текущем времени каждые 100мс модельного времени.

## Запуск программы

Для запуска программы откройте терминал, укажите путь к файлу с кодом и введите команду<br>
```
go run hub.go http://localhost:9998 ef0`<br>
```
, где http://localhost:9998 - адрес, который вы присвоили серверу( по умолчанию он http://localhost:9998), ef0 - адрес хаба закодированного в 16-ричной системе счисления


## Кодирование и декодирование пакетов

Демо-сервер может использоваться как утилита для кодирования и декодирования пакетов.

Обратите внимание, что JSON-формат используется только для отладочных целей. По сети передаются пакеты в бинарном формате, закодированные в Base64.

Опция `-B` выполняет декодирование пакета из base64 в JSON-форму, например, если выполнить из командной строки Unix команду

```
echo DbMG_38BBgaI0Kv6kzGK | smarthome.exe -B
```

на стандартный поток вывода будет напечатано

```
[
    {
        "length": 13,
        "payload": {
            "src": 819,
            "dst": 16383,
            "serial": 1,
            "dev_type": 6,
            "cmd": 6,
            "cmd_body": {
                "timestamp": 1688984021000
            }
        },
        "crc8": 138
    }
]
```

Опция `-J` выполняет преобразование из JSON-представления в base64-представления для отправки. Например, если запустить из командной строки команду

```
smarthome.exe -J
```

 и на стандартном потоке ввода ввести

```
[
    {
        "length": 13,
        "payload": {
            "src": 819,
            "dst": 16383,
            "serial": 1,
            "dev_type": 6,
            "cmd": 6,
            "cmd_body": {
                "timestamp": 1688984021000
            }
        },
        "crc8": 138
    }
]
```

на стандартный поток вывода будет напечатано

```
DbMG_38BBgaI0Kv6kzGK
```

Опция `-K` выполняет преобразование из JSON-представления в бинарное представление. Так можно посмотреть, как представляется пакет в бинарной форме до его кодирования в Base64.
Например, если запустить из командной строки команду

```
smarthome.exe -K | hexdump -C
```

 и на стандартном потоке ввода ввести

```
[
    {
        "length": 13,
        "payload": {
            "src": 819,
            "dst": 16383,
            "serial": 1,
            "dev_type": 6,
            "cmd": 6,
            "cmd_body": {
                "timestamp": 1688984021000
            }
        },
        "crc8": 138
    }
]
```

на стандартный поток вывода будет напечатано

```
00000000  0d b3 06 ff 7f 01 06 06  88 d0 ab fa 93 31 8a     |.............1.|
```

## Примеры пакетов

Ниже приведены примеры пакетов в base64 для всех возможных пар из типа устройства и команды.

* SmartHub, WHOISHERE (1, 1): `DAH_fwEBAQVIVUIwMeE`
* SmartHub, IAMHERE (1, 2): `DAH_fwIBAgVIVUIwMak`
* EnvSensor, WHOISHERE (2, 1): `OAL_fwMCAQhTRU5TT1IwMQ8EDGQGT1RIRVIxD7AJBk9USEVSMgCsjQYGT1RIRVIzCAAGT1RIRVI03Q`
* EnvSensor, IAMHERE (2, 2): `OAL_fwQCAghTRU5TT1IwMQ8EDGQGT1RIRVIxD7AJBk9USEVSMgCsjQYGT1RIRVIzCAAGT1RIRVI09w`
* EnvSensor, GETSTATUS (2, 3): `BQECBQIDew`
* EnvSensor, STATUS (2, 4): `EQIBBgIEBKUB4AfUjgaMjfILrw`
* Switch, WHOISHERE (3, 1): `IgP_fwcDAQhTV0lUQ0gwMQMFREVWMDEFREVWMDIFREVWMDO1`
* Switch, IAMHERE (3, 2): `IgP_fwgDAghTV0lUQ0gwMQMFREVWMDEFREVWMDIFREVWMDMo`
* Switch, GETSTATUS (3, 3): `BQEDCQMDoA`
* Switch, STATUS (3, 4): `BgMBCgMEAac`
* Lamp, WHOISHERE (4, 1): `DQT_fwsEAQZMQU1QMDG8`
* Lamp, IAMHERE (4, 2): `DQT_fwwEAgZMQU1QMDGU`
* Lamp, GETSTATUS (4, 3): `BQEEDQQDqw`
* Lamp, STATUS (4, 4): `BgQBDgQEAaw`
* Lamp, SETSTATUS (4, 5): `BgEEDwQFAeE`
* Socket, WHOISHERE (5, 1): `DwX_fxAFAQhTT0NLRVQwMQ4`
* Socket, IAMHERE (5, 2): `DwX_fxEFAghTT0NLRVQwMc0`
* Socket, GETSTATUS (5, 3): `BQEFEgUD5A`
* Socket, STATUS (5, 4): `BgUBEwUEAQ8`
* Socket, SETSTATUS (5, 5): `BgEFFAUFAQc`
* Clock, IAMHERE (6, 2): `Dgb_fxUGAgdDTE9DSzAxsw`
* Clock, TICK (6, 6): `DAb_fxgGBpabldu2NNM`
