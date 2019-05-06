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
ms.openlocfilehash: 354469199f3c7e886760e8a391edccde067986af
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610294"
---
# <a name="docker-engine-on-windows"></a>Подсистема Docker в Windows

Подсистема и клиент Docker не включаются в Windows и необходимо установить и настроить по отдельности. Кроме того, подсистема Docker может принимать множество пользовательских конфигураций. Например, можно настроить то, как управляющая программа принимает входящие запросы, сетевые параметры по умолчанию и параметры ведения журнала и отладки. В ОС Windows эти конфигурации можно указать в файле конфигурации или с помощью диспетчера служб Windows. В этом документе подробно описано, как установить и настроить подсистему Docker и также предоставляет примеры некоторых часто используемых конфигураций.

## <a name="install-docker"></a>Установка Docker

Для работы с контейнерами Windows необходимо Docker. Docker состоит из подсистемы Docker (dockerd.exe) и клиента Docker (docker.exe). Самый простой способ получить все приложения, установленные находится в Краткое руководство, руководства, которые помогут вам получить все настройки и запуск первого контейнера.

- [Контейнеры Windows в Windows Server 2019](../quick-start/quick-start-windows-server.md)
- [Контейнеры Windows в Windows 10](../quick-start/quick-start-windows-10.md)

Об установке см. в разделе [Использование сценария для установки Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Прежде чем можно будет использовать Docker, вам потребуется установить образы контейнеров. Дополнительные сведения см. [в руководстве по краткое руководство по использованию изображения](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Настройка Docker с помощью файла конфигурации

Предпочтительным способом настройки подсистемы Docker в Windows является использование файла конфигурации. Путь к файлу конфигурации— C:\ProgramData\Docker\config\daemon.json. Можно создать этот файл, если он еще не существует.

>[!NOTE]
>Не все доступные параметры конфигурации Docker относится к Docker в Windows. В следующем примере показано параметры конфигурации, которые применяются. Дополнительные сведения о настройке подсистемы см. в разделе [файл конфигурации управляющей программы Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Необходимо только добавить требуемые изменения в файл конфигурации. Например следующий пример настраивает подсистему Docker на прием входящих подключений через порт 2375. В других параметрах конфигурации будут использоваться значения по умолчанию.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Аналогичным образом указанного ниже примера управляющая программа Docker для сохранения образов и контейнеров по альтернативному пути. Если параметр не задан, по умолчанию используется `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

Следующий пример управляющая программа Docker настраивается на прием только защищенных подключений через порт 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Настройка Docker в службе Docker

Подсистема Docker можно также настроить, изменив службу Docker с помощью `sc config`. При использовании этого способа флаги подсистемы Docker задаются непосредственно в службе Docker. Выполните указанную ниже команду в командной строке (cmd.exe, не PowerShell).

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Вам не нужно выполнять эту команду Если файл daemon.json уже содержит `"hosts": ["tcp://0.0.0.0:2375"]` входа.

## <a name="common-configuration"></a>Распространенные конфигурации

В следующих примерах файла конфигурации представлены распространенные конфигурации Docker. Их можно объединить в один файл конфигурации.

### <a name="default-network-creation"></a>Создание сети по умолчанию

Чтобы настроить подсистему Docker, таким образом, чтобы она не создает сеть NAT по умолчанию, используйте следующую конфигурацию.

```json
{
    "bridge" : "none"
}
```

Дополнительные сведения см. в статье [Управление сетями Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Задание группы безопасности для Docker

При входа к узлу Docker и команд Docker локально выполняются эти команды выполняются через именованный канал. По умолчанию только члены группы "Администраторы" могут получить доступ к подсистеме Docker через именованный канал. Чтобы указать группу безопасности, имеющую такой доступ, используйте флаг `group`.

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

Дополнительные сведения см. в разделе [Файл конфигурации Windows на сайте Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Удаление Docker

В этом разделе можно узнать, как удалить Docker и выполнить полную очистку компонентов системы Docker в Windows 10 или Windows Server 2016.

>[!NOTE]
>В этих инструкциях из сеансов PowerShell высшего необходимо выполнить все команды.

### <a name="prepare-your-system-for-dockers-removal"></a>Подготовка системы к удалению Docker

Перед удалением Docker, убедитесь, что не выполняются контейнеры в вашей системе.

Выполните приведенные ниже командлеты для проверки наличия контейнерами:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Это также рекомендуется удалить все контейнеры, образы контейнеров, сети и тома из системы перед удалением Docker. Это можно сделать, выполнив следующий командлет:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Удаление Docker

Затем вам потребуется фактически удалять Docker.

Для удаления Docker в Windows 10

- Перейдите к **параметрам** > **приложений** на компьютере с Windows 10
- В разделе **& функции приложения**найдите **Docker для Windows**
- Последовательно выберите пункты **Docker для Windows** > **удаления**

Для удаления Docker в Windows Server 2016:

Из сеансов PowerShell высшего используйте командлеты **Удаление пакета** и **Удаления модуля** для удаления модуля Docker и соответствующего поставщик управления пакетами из системы, как показано в следующем примере:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Можно найти поставщик пакетов, который использовался для установки Docker с помощью `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Очистка данных и системных компонентов Docker

После удаления Docker, необходимо удалить сети Docker по умолчанию, чтобы их конфигурации не остаются в системе, когда исчезнет Docker. Это можно сделать, выполнив следующий командлет:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Выполните следующий командлет, чтобы удалить данные программы Docker из системы:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Можно также удалить необязательные компоненты Windows, связанные с Docker и контейнерами в Windows.

Сюда входят компонент «Контейнеры», который автоматически включается в любом экземпляре Windows 10 или Windows Server 2016 при установке Docker. Это также может быть компонент «Hyper-V», который автоматически включается в Windows 10 при установке Docker, однако в Windows Server 2016 он включается вручную.

>[!IMPORTANT]
>[Функция Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/) — это общий компонент виртуализации, позволяющий гораздо больше, чем просто контейнеров. Прежде чем отключить Hyper-V, убедитесь, что нет других виртуальных компонентов в системе, требующих Hyper-V.

Удаление компонентов Windows в Windows 10.

- Перейдите в меню **Панель управления** > **программы** > **программы и компоненты** > **Включение или отключение компонентов**.
- Найдите имя включив нужные компоненты, вы захотите отключить — в данном случае **контейнеров** и (необязательно) **Hyper-V**.
- Снимите флажок рядом с именем компонента, который вы хотите отключить.
- Нажмите кнопку **«ОК»**

Удаление компонентов Windows в Windows Server 2016.

Из сеансов PowerShell высшего, выполните приведенные ниже командлеты, чтобы отключить **контейнеров** и (необязательно) компоненты **Hyper-V** из системы:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Перезагрузка компьютера

Для завершения удаления и очистки, запустите следующий командлет из сеансов PowerShell высшего перезагрузки системы:

```powershell
Restart-Computer -Force
```
