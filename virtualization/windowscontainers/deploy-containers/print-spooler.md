---
title: Очередь печати принтера в контейнерах Windows
description: Описание текущего поведения работы службы печати принтера в контейнерах Windows
keywords: docker, контейнеры, принтер, принтера
author: cwilhit
ms.openlocfilehash: 45176e651ee2ef9b6daea9919004601734084083
ms.sourcegitcommit: 04c372c87c832f73a1aa120b0ff6c2c2b9c8c1b1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2019
ms.locfileid: "9257989"
---
# <a name="print-spooler-in-windows-containers"></a>Очередь печати принтера в контейнерах Windows

Приложения с зависимостью от служб печати можно контейнерных успешно с контейнерами Windows. Не может быть контейнерных приложений, имеющих зависимости для установки драйверов принтера в узле; Установка драйвера из внутри контейнера не поддерживается, так как утечку состояние контейнера на узле. Существуют специальные требования, которые должны быть выполнены для успешного включения функции службы принтера. В этом руководстве объясняется, как правильно настроить развертывания.

> [!IMPORTANT]
> При получении доступа к печати обслуживает успешно контейнеры работает, функциональность будет ограничена в форме; Некоторые действия печати, могут не работать. Если это так, откройте ниже отзыв.

## <a name="setup"></a>Настройка

* Узел должен быть Windows Server 2019 или Windows 10 Pro или Корпоративная октября 2018 г. или более поздней версии.
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) изображения должны быть целевых базового образа. Роль сервера печати не подходят для других Windows базовыми образами контейнера (например, Nano Server и Windows Server Core).

### <a name="hyper-v-isolation"></a>Изоляция Hyper-V

Мы рекомендуем запуск контейнера с изоляцией Hyper-V. При запуске в этом режиме, может быть любое количество контейнеров, как вы хотите работать с доступом для служб печати. Вам не нужно Изменение принтера службы на узле.

Вы можете проверить функциональность со следующим запросом PowerShell:

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

### <a name="process-isolation"></a>Изоляции процессов

Из-за особенностей общее ядро изолированного процесса контейнеры текущего поведения ограничивает пользователя запущен только **один экземпляр** службу очереди печати принтера между основным приложением и все дочерние объекты контейнера. Если на узле есть под управлением диспетчер очереди печати, необходимо остановить службу на узле перед attemping запустить службу принтера на гостевой виртуальной машине.

> [!TIP]
> Если после запуска контейнера, одновременно запрашивать службу принтера в контейнер и узлом оба сообщит об их состояние как «запущено». Но не deceived — контейнер не сможет запрашивать список доступных принтеров. Служба диспетчера основного приложения не должны выполняться. 

Чтобы проверить, узел того, выполняется ли служба принтер, используйте запрос в PowerShell ниже:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Чтобы остановить службу принтера на узле, используйте следующие команды в PowerShell ниже:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Запустите контейнер и проверки доступа к принтеры.

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