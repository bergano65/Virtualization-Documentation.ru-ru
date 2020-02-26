---
title: Диспетчер очереди печати в контейнерах Windows
description: Описание текущего рабочего поведения службы диспетчера очереди печати в контейнерах Windows
keywords: DOCKER, контейнеры, принтер, диспетчер очереди печати
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439541"
---
# <a name="print-spooler-in-windows-containers"></a>Диспетчер очереди печати в контейнерах Windows

Приложения с зависимостью от служб печати могут быть успешно включены в контейнеры Windows. Для успешного включения функций службы печати необходимо соблюдать особые требования. В этом руководство объясняется, как правильно настроить развертывание.

> [!IMPORTANT]
> При успешном получении доступа к службам печати в контейнерах функциональные возможности ограничены в форме. Некоторые действия, связанные с печатью, могут не работать. Например, приложения, зависящие от установки драйверов принтера в узле, не могут быть контейнерными, так как **Установка драйвера из контейнера не поддерживается**. Если вы нашли неподдерживаемую функцию печати, которую вы хотите поддерживать в контейнерах, откройте отзыв.

## <a name="setup"></a>Установка

* Узел должен иметь версию Windows Server 2019 или Windows 10 Pro/Enterprise Октябрь 2018 или более поздней версии.
* Образ [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) должен быть целевым базовым образом. Другие базовые образы контейнеров Windows (например, Nano Server и Windows Server Core) не имеют роли сервера печати.

### <a name="hyper-v-isolation"></a>Изоляция Hyper-V

Рекомендуется запускать контейнер с изоляцией Hyper-V. При запуске в этом режиме можно использовать столько контейнеров, сколько требуется для выполнения с доступом к службам печати. Не нужно изменять службу диспетчера очереди печати на узле.

Проверить функциональность можно с помощью следующего запроса PowerShell:

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>Изоляция процессов

Из-за общего характера ядра контейнеров, изолированных от процессов, текущее поведение ограничивает пользователя запуском только **одного экземпляра** службы диспетчера очереди печати на узле и всех его дочерних контейнерах. Если на узле запущен диспетчер очереди печати, необходимо закрыть службу на узле, прежде чем попытке запустить службу принтера в гостевой системе.

> [!TIP]
> Если запустить контейнер и запросить службу очереди печати одновременно в контейнере и на узле, то оба будут сообщать о своем состоянии "выполняется". Но не децеивед--контейнер не сможет запросить список доступных принтеров. Служба диспетчера очереди узла не должна работать. 

Чтобы проверить, запущена ли на узле служба принтера, используйте запрос в PowerShell ниже:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Чтобы отключить службу диспетчера очереди на узле, используйте следующие команды в PowerShell:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Запустите контейнер и проверьте доступ к принтерам.

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```