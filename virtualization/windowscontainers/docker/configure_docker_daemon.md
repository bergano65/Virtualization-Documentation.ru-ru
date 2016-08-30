---
title: "Настройка Docker в Windows"
description: "Настройка Docker в Windows"
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: 4dded90462c5438a6836ec32a165a9cc1019d6ec
ms.openlocfilehash: 6ae49d82a89b2f30198de05aa4915726853172f5

---

# Подсистема Docker в Windows

Подсистема и клиент Docker не входят в состав Windows, потому их нужно установить и настроить отдельно. Кроме того, подсистема Docker может принимать множество пользовательских конфигураций. Например, можно настроить то, как управляющая программа принимает входящие запросы, сетевые параметры по умолчанию и параметры ведения журнала и отладки. В ОС Windows эти конфигурации можно указать в файле конфигурации или с помощью диспетчера служб Windows. В этом документе описано, как установить и настроить подсистему Docker; также представлены примеры некоторых часто используемых конфигураций.

## Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента.

Скачайте подсистему Docker.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Разархивируйте ZIP-архив в Program Files.

```
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Добавьте каталог Docker в системный путь. По завершении перезапустите сеанс PowerShell, чтобы был распознан измененный путь.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Чтобы установить Docker в качестве службы Windows, выполните следующую команду.

```none
dockerd --register-service
```

После установки эту службу можно запустить.

```none
Start-Service Docker
```

Прежде чем можно будет использовать Docker, нужно установить образы контейнеров. Дополнительные сведения см. в статье [Управление образами контейнеров](../management/manage_images.md).

## Файл конфигурации Docker

Предпочтительным способом настройки подсистемы Docker в Windows является файл конфигурации. Путь к файлу конфигурации — C:\ProgramData\docker\config\daemon.json. Если этот файл еще не существует, его можно создать.

Примечание. Не все доступные параметры конфигурации Docker применимы к Docker в Windows. В примере ниже показаны применимые. Полную документацию по настройке подсистемы Docker (включая инструкции для Linux) см. в статье [Docker Daemon]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/) (Управляющая программа Docker).

```none
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "graph": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

В файл конфигурации следует добавлять только требуемые изменения в конфигурации. Например, в этом случае подсистема Docker настраивается на прием входящих подключений через порт 2375. В других параметрах конфигурации будут использоваться значения по умолчанию.

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

А в этом примере управляющая программа Docker настраивается на прием только защищенных подключений через порт 2376.

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## Диспетчер управления службами

Подсистему Docker можно также настроить, изменив службу Docker командой `sc config`. При использовании этого способа флаги подсистемы Docker задаются непосредственно в службе Docker.


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## Распространенные конфигурации

В следующих примерах файла конфигурации представлены распространенные конфигурации Docker. Их можно объединить в один файл конфигурации.

### Создание сети по умолчанию 

Чтобы указать подсистеме Docker, что сеть NAT по умолчанию создаваться не должна, используйте следующий код. Дополнительные сведения см. в статье [Управление сетями Docker](../management/container_networking.md).

```none
{
    "bridge" : "none"
}
```

### Задание группы безопасности для Docker

При входе на компьютер с Docker и выполнении команд Docker локально команды выполняются через именованный канал. По умолчанию только члены группы "Администраторы" могут получить доступ к подсистеме Docker через именованный канал. Чтобы указать группу безопасности, имеющую такой доступ, используйте флаг `group`.

```none
{
    "group" : "docker"
}
```

## Настройка прокси-сервера

Чтобы задать данные о прокси-сервере для `docker search` и `docker pull`, создайте переменную среды Windows с именем `HTTP_PROXY` или `HTTPS_PROXY` и значением, содержащим данные о прокси-сервере. Это можно сделать в PowerShell, используя команду следующего вида:

```none
[Environment]::SetEnvironmentVariable("HTTP_PROXY”, “http://username:password@proxy:port/”, [EnvironmentVariableTarget]::Machine)
```

После задания переменной перезапустите службу Docker.

```none
restart-service docker
```

Дополнительные сведения см. в статье [Daemon Socket Options](https://docs.docker.com/v1.10/engine/reference/commandline/daemon/#daemon-socket-option) (Параметры сокетов управляющей программы) на Docker.com.

## Сбор журналов
Подсистема Docker записывает сообщения в журнал событий приложений Windows, а не в файл журнала. Эти журналы можно легко прочитать, отсортировать и отфильтровать с помощью Windows PowerShell.

Например, следующая команда выведет записи журнала подсистемы Docker за последние 5 минут, начиная с самой ранней.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Эти записи легко перенаправить в CSV-файл, чтобы открыть их в другой программе или редакторе электронных таблиц.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.csv ```



<!--HONumber=Aug16_HO4-->


