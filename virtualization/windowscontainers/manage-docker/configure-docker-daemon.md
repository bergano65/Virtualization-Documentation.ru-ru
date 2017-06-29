---
title: "Настройка Docker в Windows"
description: "Настройка Docker в Windows"
keywords: "docker, контейнеры"
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: 9aa9e4a0415a89762438a8e8a85a901ca5360c6f
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: ru-RU
---
# <a name="docker-engine-on-windows"></a>Подсистема Docker в Windows

Подсистема и клиент Docker не входят в состав Windows, потому их нужно установить и настроить отдельно. Кроме того, подсистема Docker может принимать множество пользовательских конфигураций. Например, можно настроить то, как управляющая программа принимает входящие запросы, сетевые параметры по умолчанию и параметры ведения журнала и отладки. В ОС Windows эти конфигурации можно указать в файле конфигурации или с помощью диспетчера служб Windows. В этом документе описано, как установить и настроить подсистему Docker; также представлены примеры некоторых часто используемых конфигураций.


## <a name="install-docker"></a>Установка Docker
Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker (dockerd.exe) и клиента Docker (docker.exe). Самый простой способ установки всех компонентов указан в кратких руководствах по началу работы. С их помощью вы сможете все настроить и запустить первый контейнер. 

* [Контейнеры Windows в Windows Server2016](../quick-start/quick-start-windows-server.md)
* [Контейнеры Windows в Windows10](../quick-start/quick-start-windows-10.md)


### <a name="manual-installation"></a>установка вручную;
Если вы хотите использовать версии подсистемы и клиента Docker, находящиеся в разработке, выполните следующие действия. Будут установлены подсистема и клиент Docker. Если вы являетесь разработчиком и выполняете тестирование новых возможностей или используете сборку программы предварительной оценки Windows, вам может потребоваться использовать версию Docker, находящуюся в разработке. В противном случае следуйте инструкциям в разделе "Установка Docker" выше, чтобы получить последние выпущенные версии.

> Если у вас установлена версия Docker для Windows, удалите ее перед тем, как выполнить действия по ручной установке. 

Скачивание подсистемы Docker

Последнюю версию всегда можно найти здесь: https://master.dockerproject.org. В этом примере используется последняя версия из основной ветви (Master Branch). 

```powershell
$version = (Invoke-WebRequest -UseBasicParsing https://raw.githubusercontent.com/docker/docker/master/VERSION).Content.Trim()
Invoke-WebRequest "https://master.dockerproject.org/windows/amd64/docker-$($version).zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Распакуйте ZIP-архив в папку Program Files.

```powershell
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Добавьте каталог Docker в системный путь. По завершении перезапустите сеанс PowerShell, чтобы был распознан измененный путь.

```powershell
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Чтобы установить Docker в качестве службы Windows, выполните следующую команду.

```none
dockerd --register-service
```

После установки эту службу можно запустить.

```powershell
Start-Service Docker
```

Прежде чем можно будет использовать Docker, нужно установить образы контейнеров. Дополнительные сведения см. в [кратком руководстве по началу работы с образами](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-configuration-file"></a>Настройка Docker с помощью файла конфигурации

Предпочтительным способом настройки подсистемы Docker в Windows является использование файла конфигурации. Путь к файлу конфигурации— C:\ProgramData\docker\config\daemon.json. Если этот файл еще не существует, его можно создать.

Примечание. Не все доступные параметры конфигурации Docker применимы к Docker в Windows. В примере ниже показаны применимые. Полную документацию по настройке подсистемы Docker (включая инструкции для Linux) см. в статье [Docker daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file) (Файл конфигурации управляющей программы Docker).

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

Аналогично в этом примере настраивается хранение образов и контейнеров по альтернативному пути в управляющей программе Docker. Если значение не указано, по умолчанию используется "c:\programdata\docker".

```none
{    
    "graph": "d:\\docker"
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

## <a name="configure-docker-on-the-docker-service"></a>Настройка Docker в службе Docker

Подсистему Docker можно также настроить, изменив службу Docker командой `sc config`. При использовании этого способа флаги подсистемы Docker задаются непосредственно в службе Docker. Выполните указанную ниже команду в командной строке (cmd.exe, не PowerShell).


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Примечание. Не нужно выполнять эту команду в том случае, если файл daemon.json уже содержит запись `"hosts": ["tcp://0.0.0.0:2375"]`.

## <a name="common-configuration"></a>Распространенные конфигурации

В следующих примерах файла конфигурации представлены распространенные конфигурации Docker. Их можно объединить в один файл конфигурации.

### <a name="default-network-creation"></a>Создание сети по умолчанию 

Чтобы указать подсистеме Docker, что сеть NAT по умолчанию создаваться не должна, используйте следующий код. Дополнительные сведения см. в статье [Управление сетями Docker](../manage-containers/container-networking.md).

```none
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>Задание группы безопасности для Docker

При входе на компьютер с Docker и выполнении команд Docker локально команды выполняются через именованный канал. По умолчанию только члены группы "Администраторы" могут получить доступ к подсистеме Docker через именованный канал. Чтобы указать группу безопасности, имеющую такой доступ, используйте флаг `group`.

```none
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Настройка прокси-сервера

Чтобы задать данные о прокси-сервере для `docker search` и `docker pull`, создайте переменную среды Windows с именем `HTTP_PROXY` или `HTTPS_PROXY` и значением, содержащим данные о прокси-сервере. Это можно сделать в PowerShell, используя команду следующего вида:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

После задания переменной перезапустите службу Docker.

```powershell
Restart-Service docker
```

Дополнительные сведения см. в разделе [Windows Configuration File](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file) (Файл конфигурации Windows) на сайте Docker.com.

