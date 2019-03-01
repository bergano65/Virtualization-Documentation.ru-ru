---
title: Настройка Docker в Windows
description: Настройка Docker в Windows
keywords: docker, контейнеры
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: bbc405fc2a490cfe5082be112fde724707e24785
ms.sourcegitcommit: 21d93e5febd9b1b47ae1aa59d08086e6ec1691e0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/28/2019
ms.locfileid: "9121056"
---
# <a name="docker-engine-on-windows"></a>Подсистема Docker в Windows

Подсистема и клиент Docker не входят с Windows, необходимо установить и настроить по отдельности. Кроме того, подсистема Docker может принимать множество пользовательских конфигураций. Например, можно настроить то, как управляющая программа принимает входящие запросы, сетевые параметры по умолчанию и параметры ведения журнала и отладки. В ОС Windows эти конфигурации можно указать в файле конфигурации или с помощью диспетчера служб Windows. В этом документе подробно описано, как установить и настроить подсистему Docker и также предоставляет примеры некоторых часто используемых конфигураций.


## <a name="install-docker"></a>Установка Docker
Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker (dockerd.exe) и клиента Docker (docker.exe). Самый простой способ установки всех компонентов указан в кратких руководствах по началу работы. Они помогут все Настройка и запуск первого контейнера. 

* [Контейнеры Windows в Windows Server 2019 г.](../quick-start/quick-start-windows-server.md)
* [Контейнеры Windows в Windows10](../quick-start/quick-start-windows-10.md)

Сведения об установке с помощью сценария см. в разделе [Использование сценария для установки Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Прежде чем можно будет использовать Docker образы контейнеров, нужно установить. Дополнительные сведения см. в [кратком руководстве по началу работы с образами](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-configuration-file"></a>Настройка Docker с помощью файла конфигурации

Предпочтительным способом настройки подсистемы Docker в Windows является использование файла конфигурации. Путь к файлу конфигурации— C:\ProgramData\Docker\config\daemon.json. Если этот файл еще не существует, его можно создать.

Примечание. Не все доступные параметры конфигурации Docker применимы к Docker в Windows. В примере ниже показаны применимые. Полную документацию по настройке подсистемы Docker (включая инструкции для Linux) см. в статье [Docker daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file) (Файл конфигурации управляющей программы Docker).

```
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
    "data-root": "",
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

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Аналогично в этом примере настраивается хранение образов и контейнеров по альтернативному пути в управляющей программе Docker. Если значение не указано, по умолчанию используется "c:\programdata\docker".

```
{    
    "data-root": "d:\\docker"
}
```

А в этом примере управляющая программа Docker настраивается на прием только защищенных подключений через порт 2376.

```
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


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Примечание. Не нужно выполнять эту команду в том случае, если файл daemon.json уже содержит запись `"hosts": ["tcp://0.0.0.0:2375"]`.

## <a name="common-configuration"></a>Распространенные конфигурации

В следующих примерах файла конфигурации представлены распространенные конфигурации Docker. Их можно объединить в один файл конфигурации.

### <a name="default-network-creation"></a>Создание сети по умолчанию 

Чтобы указать подсистеме Docker, что сеть NAT по умолчанию создаваться не должна, используйте следующий код. Дополнительные сведения см. в статье [Управление сетями Docker](../container-networking/network-drivers-topologies.md).

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>Задание группы безопасности для Docker

При входе на компьютер с Docker и выполнении команд Docker локально команды выполняются через именованный канал. По умолчанию только члены группы "Администраторы" могут получить доступ к подсистеме Docker через именованный канал. Чтобы указать группу безопасности, имеющую такой доступ, используйте флаг `group`.

```
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

## <a name="uninstall-docker"></a>Удаление Docker
*Выполните действия, описанные в этом разделе, чтобы удалить Docker и выполнить полную очистку компонентов системы Docker в Windows 10 или Windows Server 2016.*

> Примечание. Все описанные ниже команды необходимо выполнять в сеансе PowerShell **с повышенными привилегиями**.

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>ШАГ 1. Подготовка системы к удалению Docker 
Перед удалением Docker рекомендуется убедиться, что в вашей системе не выполняются контейнеры (если вы этого еще не сделали). Вот несколько полезных команды.
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
Кроме того, рекомендуется удалить все контейнеры, образы контейнеров, сети и тома из системы перед удалением Docker.
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>ШАГ 2. Удаление Docker 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>Процедура удаления Docker в Windows 1010:***
- На компьютере с Windows 10 перейдите в раздел **Параметры > Приложения**.
- В разделе **Приложения и компоненты** найдите пункт **Docker для Windows**
- Щелкните **Docker для Windows > Удалить**

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>Процедура удаления Docker в Windows Server 201616:***
В сеансе PowerShell с повышенными привилегиями выполните командлеты `Uninstall-Package` и `Uninstall-Module` для удаления модуля Docker и соответствующего поставщик управления пакетами из системы. 
> Совет. Вы можете найти поставщик пакетов, который использовался для установки Docker с помощью команды: `PS C:\> Get-PackageProvider -Name *Docker*`

*Например*:
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>ШАГ 3. Очистка данных и системных компонентов Docker
Удалите *сети Docker по умолчанию,*, чтобы их конфигурация не сохранилась в системе после удаления Docker:
```
Get-HNSNetwork | Remove-HNSNetwork
```
Удалите *программные данные* Docker из системы:
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
Можно также удалить *необязательные компоненты Windows*, связанные с Docker и контейнерами в Windows. 

Как минимум, к ним относится компонент «Контейнеры», который автоматически включается в любом экземпляре Windows 10 или Windows Server 2016 при установке Docker. Это также может быть компонент «Hyper-V», который автоматически включается в Windows 10 при установке Docker, однако в Windows Server 2016 он включается вручную.

> **ВАЖНОЕ ПРИМЕЧАНИЕ ОБ ОТКЛЮЧЕНИИ HYPER-V.** [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/) — это общий компонент виртуализации, который предоставляет очень много возможностей, а не только контейнеры! Прежде чем отключить Hyper-V, убедитесь, что в системе нет других виртуальных компонентов, которые зависят от Hyper-V.

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>Процедура удаления компонентов Windows в Windows 1010:***
- На компьютере с Windows 10 перейдите в раздел **Панель управления > Программы Hyper-V > Программы и компоненты > Включение или отключение компонентов Windows**.
- Найдите имя компонентов, которые требуется отключить — в этом случае это **Контейнеры** и (необязательно) **Hyper-V**
- **Снимите** флажок рядом с именем компонента, который вы хотите отключить
- Нажмите кнопку **ОК**.

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>Процедура удаления компонентов Windows в Windows Server 201616:***
В сеансе PowerShell с повышенными привилегиями выполните следующие команды, чтобы отключить компоненты **Контейнеры** и (необязательно) **Hyper-V**.
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>ШАГ 4. Перезагрузка компьютера
Чтобы завершить процедуру удаления и очистки, в сеансе PowerShell с повышенными привилегиями выполните следующую команду:
```
Restart-Computer -Force
```
