---
title: Контейнеры для Windows и Linux в Windows 10
description: Краткое руководство по развертыванию контейнеров
keywords: Dock, Containers, ЛКОВ
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129389"
---
# <a name="get-started-configure-your-environment-for-containers"></a>Начало работы: Настройка среды для контейнеров

В этом кратком руководстве показано, как выполнять указанные ниже действия.

> [!div class="checklist"]
> * Настройка среды для контейнеров
> * Выполнение первого изображения контейнера
> * Контаинеризе простого приложения .NET Core

## <a name="prerequisites"></a>Что вам понадобится

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Убедитесь, что вы отвечаете на следующие требования:

- Одна компьютерная система (физическая или виртуальная) под управлением Windows Server 2016 или более поздней версии.

> [!NOTE]
> Если вы используете Windows Server 2019 Insider Preview, пожалуйста, обновите [ознакомительную версию до Window server 2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ).

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 профессиональная и Корпоративная](#tab/Windows-10-Client)

Убедитесь, что вы отвечаете на следующие требования:

- Одна физическая компьютерная система под управлением Windows 10 профессиональная или Корпоративная с обновлением юбилея (версия 1607) или более поздней.
- [Технология Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) должна быть включена.

> [!NOTE]
>  Начиная с Windows 10, Октябрь обновление 2018, мы больше не запрещают пользователям запускать контейнер Windows в режиме изоляции процессов в Windows 10 Корпоративная или профессиональная для разработки и тестирования. Дополнительные сведения можно найти в разделе " [часто задаваемые вопросы](../about/faq.md) ". 
> 
> Контейнеры Windows Server по умолчанию используют изоляцию Hyper-V в Windows 10 для предоставления разработчикам одной и той же версии ядра и конфигурации, которая будет использоваться в производстве. Дополнительные сведения об изоляции Hyper-V содержатся в разделе [основные положения](../manage-containers/hyperv-container.md) наших документов.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Установка Docker

Dock является основным Native для работы с контейнерами Windows. Док-станция позволяет пользователям управлять контейнерами на определенном узле, создавать контейнеры, удалять контейнеры и многое другое. Дополнительные сведения о стыковочных модулях можно найти в разделе [основные положения](../manage-containers/configure-docker-daemon.md) наших документов.

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

В Windows Server Dock устанавливается через [модуль PowerShell поставщика онежет](https://github.com/oneget/oneget) , опубликованный корпорацией Майкрософт, который называется [доккермикрософтпровидер](https://github.com/OneGet/MicrosoftDockerProvider). Этот поставщик:

- включает функцию контейнеров на компьютере.
- Установка на компьютер подсистемы стыковочного процессора и клиента.

Чтобы установить стыковочный элемент, откройте сеанс PowerShell с повышенными привилегиями и установите на него службу Dock-Microsoft PackageManagement из [коллекции PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Затем используйте модуль PowerShell PackageManagement, чтобы установить последнюю версию Dock.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Когда в PowerShell появится запрос, доверять ли источнику пакета DockerDefault, введите `A`, чтобы продолжить установку. После завершения установки необходимо перезагрузить компьютер.

```powershell
Restart-Computer -Force
```

> [!TIP]
> Если вы хотите обновить закрепление позже, выполните указанные ниже действия.
>  - Проверьте установленную версию с помощью команды `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Найдите текущую версию с помощью команды `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - После этого выполните обновление с помощью команды `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` и команды `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 профессиональная и Корпоративная](#tab/Windows-10-Client)

В Windows 10 профессиональная и Enterprise стыковочный узел устанавливается с помощью классического установщика. Скачайте [закрепление на компьютере](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите установщик. Вам потребуется выполнить вход. Создайте учетную запись, если она еще не установлена. Более детальные инструкции по установке можно найти в [документации по Dock](https://docs.docker.com/docker-for-windows/install).

После установки на рабочем столе по умолчанию выполняются контейнеры Linux. Переключитесь на контейнеры Windows с помощью закрепляемого меню или в командной строке PowerShell, выполнив следующую команду:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Дальнейшие действия

После правильной настройки вашей среды перейдите по ссылке, чтобы получить сведения о том, как забирать и запускать контейнер.

> [!div class="nextstepaction"]
> [Выполнение первого контейнера](./run-your-first-container.md)
