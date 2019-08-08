---
title: Настройка Docker в Windows
description: Настройка Docker в Windows
keywords: docker, контейнеры
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: 953dfaf71170de656f4e6ba5e91d524708d5a12a
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998221"
---
# <a name="docker-engine-on-windows"></a>Подсистема Docker в Windows

Подсистема стыковки и клиент не входят в состав Windows и нуждаются в установке и настройке отдельно. Кроме того, подсистема Docker может принимать множество пользовательских конфигураций. Например, можно настроить то, как управляющая программа принимает входящие запросы, сетевые параметры по умолчанию и параметры ведения журнала и отладки. В ОС Windows эти конфигурации можно указать в файле конфигурации или с помощью диспетчера служб Windows. В этом документе содержатся сведения о том, как установить и настроить подсистему Dock, а также приводятся некоторые примеры часто используемых конфигураций.

## <a name="install-docker"></a>Установка Docker

Для работы с контейнерами Windows требуется закрепление. Docker состоит из подсистемы Docker (dockerd.exe) и клиента Docker (docker.exe). Самый простой способ получить все, что вы установили, — это краткое руководство, которое поможет вам настроить и запустить первый контейнер.

- [Контейнеры Windows в Windows Server 2019](../quick-start/quick-start-windows-server.md)
- [Контейнеры Windows в Windows 10](../quick-start/quick-start-windows-10.md)

Для установки программы-дока в сценариях установки ознакомьтесь с разделом [Использование сценария](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Прежде чем использовать Dock, необходимо установить изображения контейнера. Дополнительные сведения можно найти [в руководстве краткое руководство по работе с изображениями](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Настройка Dock с помощью файла конфигурации

Предпочтительным способом настройки подсистемы Docker в Windows является использование файла конфигурации. Путь к файлу конфигурации— C:\ProgramData\Docker\config\daemon.json. Вы можете создать этот файл, если он еще не существует.

>[!NOTE]
>Не все доступные параметры конфигурации Dock применяются к Docking в Windows. В следующем примере показаны параметры конфигурации, которые применяются. Дополнительные сведения о конфигурации подсистемы Dock можно найти в разделе [файл конфигурации демона Dock](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```json
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

Вам нужно только добавить в файл конфигурации нужные изменения конфигурации. Например, в следующем примере настраивается механизм стыковки для приема входящих подключений на порту 2375. В других параметрах конфигурации будут использоваться значения по умолчанию.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Аналогичным образом, в следующем примере выполняется настройка демона Dock для хранения изображений и контейнеров в альтернативном пути. Если не указано, используется `c:\programdata\docker`значение по умолчанию.

```json
{    
    "data-root": "d:\\docker"
}
```

В следующем примере производится настройка демона Dock для приема только защищенных подключений через порт 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Настройка Dock в службе Dock

Подсистему Dock также можно настроить, изменив службу Dock `sc config`. При использовании этого способа флаги подсистемы Docker задаются непосредственно в службе Docker. Выполните указанную ниже команду в командной строке (cmd.exe, не PowerShell).

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Вам не нужно выполнять эту команду, если в файле Daemon. JSON уже есть `"hosts": ["tcp://0.0.0.0:2375"]` запись.

## <a name="common-configuration"></a>Общая конфигурация

В следующих примерах файла конфигурации представлены распространенные конфигурации Docker. Их можно объединить в один файл конфигурации.

### <a name="default-network-creation"></a>Создание по сети по умолчанию

Чтобы настроить подписку так, чтобы она не выстроила сеть NAT по умолчанию, используйте следующую конфигурацию.

```json
{
    "bridge" : "none"
}
```

Дополнительные сведения см. в статье [Управление сетями Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Настройка группы безопасности Dock

После входа в систему стыковочного узла и при локальном запуске команд Dock эти команды выполняются с помощью именованного канала. По умолчанию только члены группы "Администраторы" могут получить доступ к подсистеме Docker через именованный канал. Чтобы указать группу безопасности, имеющую такой доступ, используйте флаг `group`.

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Конфигурация прокси-сервера

Чтобы задать данные о прокси-сервере для `docker search` и `docker pull`, создайте переменную среды Windows с именем `HTTP_PROXY` или `HTTPS_PROXY` и значением, содержащим данные о прокси-сервере. Это можно сделать в PowerShell, используя команду следующего вида:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

После задания переменной перезапустите службу Docker.

```powershell
Restart-Service docker
```

Дополнительные сведения можно найти [в разделе файл конфигурации Windows на DOCKER.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Удаление стыковочного узла

В этом разделе вы узнаете, как удалить стыковочный модуль и выполнить полную очистку системных компонентов DOCKER из системы Windows 10 или Windows Server 2016.

>[!NOTE]
>Вы должны выполнить все команды в этих инструкциях из сеанса PowerShell с повышенными привилегиями.

### <a name="prepare-your-system-for-dockers-removal"></a>Подготовка системы к удалению стыковочного узла

Перед удалением Dock убедитесь в том, что на вашем компьютере не работают никакие контейнеры.

Выполните следующие командлеты, чтобы проверить работу контейнеров:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Кроме того, перед удалением дока рекомендуется удалить из системы все контейнеры, изображения-контейнеры, сети и тома. Это можно сделать, выполнив следующий командлет:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Удаление Docker

После этого вам потребуется удалить стыковочный узел.

Удаление стыковочного узла в Windows 10

- Откройте**приложение** " **Параметры** > " на компьютере с Windows 10.
- В разделе **приложения & функции**найдите **Dock для Windows**
- Переход к **Dock для** > **удаления** Windows

Чтобы удалить Dock на сервере Windows Server 2016, выполните указанные ниже действия.

В сеансе PowerShell с повышенными привилегиями используйте командлеты **uninstall-package** и **uninstall-module** для удаления из системы модуля Dock и соответствующего поставщика управления пакетами, как показано в следующем примере:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Вы можете найти поставщика пакетов, который использовался для установки Dock с `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Очистка данных и компонентов системы DOCKER

После удаления стыковочного узла вам потребуется удалить используемые по умолчанию сети стыковочного устройства, чтобы их конфигурация не оставалась на вашем компьютере после выхода из стыковочного узла. Это можно сделать, выполнив следующий командлет:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Запустите следующий командлет, чтобы удалить данные программы DOCKER из системы:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Можно также удалить необязательные компоненты Windows, связанные с Docker и контейнерами в Windows.

Сюда входит функция Containers, которая автоматически включается в любой Windows 10 или Windows Server 2016 при установленном Dock. Это также может быть компонент «Hyper-V», который автоматически включается в Windows 10 при установке Docker, однако в Windows Server 2016 он включается вручную.

>[!IMPORTANT]
>[Функция Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) — это общая функция виртуализации, которая позволяет значительно больше, чем просто контейнеры. Перед отключением компонента Hyper-V убедитесь, что в системе нет других виртуализированных компонентов, требующих Hyper-V.

Чтобы удалить компоненты Windows в Windows 10, выполните указанные ниже действия.

- **** На >  **панели** > управления выберите программы**и компоненты** > , чтобы**включить или выключить компоненты Windows**.
- Найдите имя компонента или функций, которые вы хотите отключить — в этом случае — контейнеры **** и (необязательно) **Hyper-V**.
- Снимите флажок рядом с названием функции, которую вы хотите отключить.
- Нажмите кнопку **"ОК"**

Чтобы удалить компоненты Windows в Windows Server 2016, выполните указанные ниже действия.

В сеансе PowerShell с повышенными привилегиями выполните следующие командлеты, **** чтобы отключить контейнеры и (необязательно) компоненты **Hyper-V** из системы.

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Перезагрузка системы

Чтобы завершить удаление и очистку, выполните следующий командлет из сеанса PowerShell с повышенными привилегиями для перезагрузки системы:

```powershell
Restart-Computer -Force
```
