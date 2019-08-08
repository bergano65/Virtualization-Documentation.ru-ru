---
title: Диспетчер очереди печати в контейнерах Windows
description: Описание текущего рабочего поведения службы очереди печати в контейнерах Windows
keywords: Dock, контейнеры, принтер, диспетчер очереди
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999101"
---
# <a name="print-spooler-in-windows-containers"></a>Диспетчер очереди печати в контейнерах Windows

Приложения с зависимостью от служб печати могут быть успешно включены в контейнеры Windows. Для успешного включения функций печати необходимо соблюдать особые требования. В этом руководстве объясняется, как правильно настроить развертывание.

> [!IMPORTANT]
> При успешном доступе к службам печати в контейнерах функциональные возможности ограничены в форме; Некоторые действия, связанные с печатью, могут не работать. Например, приложения, которые имеют зависимость от установки драйверов принтеров, не могут быть контейнерными, так как **Установка драйверов из контейнера не поддерживается**. Если вы нашли неподдерживаемую функцию печати, которую вы хотите поддерживать в контейнерах, пожалуйста, откройте отзыв ниже.

## <a name="setup"></a>Настройка

* Это должен быть Windows Server 2019 или Windows 10 Pro/2018 Enterprise с обновлением и более новыми версиями.
* Изображение [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) должно быть целевым базовым изображением. Другие базовые образы контейнера Windows (например, Nano Server и Windows Server Core) не разносят роль сервера печати.

### <a name="hyper-v-isolation"></a>Изоляция Hyper-V

Рекомендуем запустить контейнер с изоляцией Hyper-V. При запуске в этом режиме вы можете использовать любое количество контейнеров, доступ к которым будет выполняться с помощью служб печати. Вам не нужно изменять службу очереди печати на узле.

Для проверки функциональных возможностей можно использовать следующий запрос PowerShell:

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

Из-за общего характера ядра изолированных контейнеров, текущее поведение ограничивает возможности пользователей запускать только **один экземпляр** службы диспетчера очереди печати для всего узла и его дочерних элементов. Если на узле запущен диспетчер очереди печати, необходимо остановить службу на узле, прежде чем аттемпинг запустить службу печати в гостевой системе.

> [!TIP]
> Если вы запустили контейнер и запрос для службы очереди печати одновременно в контейнере и на узле, оба будут сообщать о своем состоянии состояние "работает". Но не децеивед, контейнер не сможет запросить список доступных принтеров. Служба диспетчера очереди обслуживания не должна выполняться. 

Чтобы проверить, работает ли на узле служба печати, воспользуйтесь запросом в PowerShell.

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Чтобы остановить службу очереди печати на узле, выполните указанные ниже команды в PowerShell.

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