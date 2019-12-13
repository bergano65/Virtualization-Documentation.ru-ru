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
ms.openlocfilehash: c84a6652b5918238ee8ef6e1fa7a9b2aa596aefd
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910164"
---
# <a name="docker-engine-on-windows"></a>Подсистема Docker в Windows

Подсистема DOCKER и клиент не входят в состав Windows и должны быть установлены и настроены по отдельности. Кроме того, подсистема Docker может принимать множество пользовательских конфигураций. Например, можно настроить то, как управляющая программа принимает входящие запросы, сетевые параметры по умолчанию и параметры ведения журнала и отладки. В ОС Windows эти конфигурации можно указать в файле конфигурации или с помощью диспетчера служб Windows. В этом документе описано, как установить и настроить подсистему DOCKER, а также приводятся некоторые примеры часто используемых конфигураций.

## <a name="install-docker"></a>Установка Docker

Для работы с контейнерами Windows требуется DOCKER. Docker состоит из подсистемы Docker (dockerd.exe) и клиента Docker (docker.exe). Самый простой способ установить все, что можно сделать, — это краткое руководство, которое поможет вам настроить и запустить первый контейнер.

- [Установка Docker](../quick-start/set-up-environment.md)

Сведения об установке по сценарию см. [в статье Использование сценария для установки DOCKER ee](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Прежде чем использовать DOCKER, необходимо установить образы контейнеров. Дополнительные сведения см. в разделе [Документация по основным образам контейнера](../manage-containers/container-base-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Настройка DOCKER с помощью файла конфигурации

Предпочтительным способом настройки подсистемы Docker в Windows является использование файла конфигурации. Путь к файлу конфигурации — C:\ProgramData\Docker\config\daemon.json. Если этот файл еще не существует, его можно создать.

>[!NOTE]
>Не все доступные параметры конфигурации DOCKER применяются к DOCKER в Windows. В следующем примере показаны параметры конфигурации, которые применяются. Дополнительные сведения о конфигурации подсистемы DOCKER см. в разделе [файл конфигурации управляющей программы DOCKER](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Необходимо только добавить необходимые изменения конфигурации в файл конфигурации. Например, в следующем примере подсистема DOCKER настраивается на прием входящих подключений через порт 2375. В других параметрах конфигурации будут использоваться значения по умолчанию.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Аналогично, следующий пример настраивает управляющую программу DOCKER для сохранения образов и контейнеров в альтернативном пути. Если значение не указано, по умолчанию используется `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

В следующем примере управляющая программа DOCKER настраивается на прием только безопасных подключений через порт 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Настройка DOCKER в службе DOCKER

Подсистему DOCKER также можно настроить, изменив службу DOCKER с помощью `sc config`. При использовании этого способа флаги подсистемы Docker задаются непосредственно в службе Docker. Выполните указанную ниже команду в командной строке (cmd.exe, не PowerShell).

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Вам не нужно выполнять эту команду, если файл daemon. JSON уже содержит запись `"hosts": ["tcp://0.0.0.0:2375"]`.

## <a name="common-configuration"></a>Общая конфигурация

В следующих примерах файла конфигурации представлены распространенные конфигурации Docker. Их можно объединить в один файл конфигурации.

### <a name="default-network-creation"></a>Создание сети по умолчанию

Чтобы настроить подсистему DOCKER таким образом, чтобы она не была создана сеть NAT по умолчанию, используйте следующую конфигурацию.

```json
{
    "bridge" : "none"
}
```

Дополнительные сведения см. в статье [Управление сетями Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Задание группы безопасности DOCKER

После входа в систему на узле DOCKER и запуска команд DOCKER эти команды выполняются через именованный канал. По умолчанию только члены группы "Администраторы" могут получить доступ к подсистеме Docker через именованный канал. Чтобы указать группу безопасности, имеющую такой доступ, используйте флаг `group`.

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

Дополнительные сведения см. [в разделе файл конфигурации Windows на DOCKER.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Удаление DOCKER

В этом разделе вы узнаете, как удалить DOCKER и выполнить полную очистку системных компонентов DOCKER из системы Windows 10 или Windows Server 2016.

>[!NOTE]
>Все команды необходимо выполнить в этих инструкциях из сеанса PowerShell с повышенными привилегиями.

### <a name="prepare-your-system-for-dockers-removal"></a>Подготовка системы к удалению DOCKER

Перед удалением DOCKER убедитесь, что в системе не выполняются контейнеры.

Выполните следующие командлеты, чтобы проверить работу контейнеров:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Кроме того, перед удалением DOCKER рекомендуется удалить все контейнеры, образы контейнеров, сети и тома из системы. Это можно сделать, выполнив следующий командлет:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Удаление Docker

Затем необходимо удалить DOCKER.

Удаление DOCKER в Windows 10

- Последовательно выберите **параметры** > **приложения** на компьютере с Windows 10.
- В разделе **приложения & компоненты**найдите **DOCKER для Windows**
- Последовательно выберите **Docker для Windows** > **Удалить**

Чтобы удалить DOCKER в Windows Server 2016, выполните следующие действия.

В сеансе PowerShell с повышенными привилегиями используйте командлеты **uninstall-package** и **uninstall-module** , чтобы удалить модуль docker и соответствующий поставщик Управление пакетами из системы, как показано в следующем примере:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Поставщик пакетов, который использовался для установки DOCKER, можно найти `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Очистка данных DOCKER и компонентов системы

После удаления DOCKER необходимо удалить сети DOCKER по умолчанию, чтобы их конфигурация не оставалась в системе после того, как DOCKER будет потерян. Это можно сделать, выполнив следующий командлет:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Выполните следующий командлет, чтобы удалить данные программы DOCKER из системы:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Вам также может потребоваться удалить дополнительные компоненты Windows, связанные с DOCKER или контейнерами в Windows.

Сюда входит функция "контейнеры", которая автоматически включается в любой Windows 10 или Windows Server 2016 при установке DOCKER. Это также может быть компонент «Hyper-V», который автоматически включается в Windows 10 при установке Docker, однако в Windows Server 2016 он включается вручную.

>[!IMPORTANT]
>[Компонент Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) — это общая функция виртуализации, которая позволяет гораздо больше, чем просто контейнеры. Перед отключением компонента Hyper-V убедитесь, что в системе нет других виртуализированных компонентов, требующих Hyper-V.

Чтобы удалить компоненты Windows в Windows 10, выполните следующие действия.

- Перейдите в **Панель управления** > **программы** > **программы и компоненты** > **Включение или отключение компонентов Windows**.
- Найдите имя компонента или компонентов, которые требуется отключить — в данном случае **контейнеры** и (необязательно) **Hyper-V**.
- Снимите флажок рядом с именем компонента, который необходимо отключить.
- Нажмите кнопку **"ОК"** .

Чтобы удалить компоненты Windows в Windows Server 2016, выполните следующие действия.

В сеансе PowerShell с повышенными привилегиями выполните следующие командлеты, чтобы отключить **контейнеры** и (необязательно) компоненты **Hyper-V** из системы:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Перезагрузите систему

Чтобы завершить удаление и очистку, выполните следующий командлет из сеанса PowerShell с повышенными привилегиями для перезагрузки системы:

```powershell
Restart-Computer -Force
```
