---
title: Контейнеры для Windows и Linux в Windows 10
description: Настройте Windows 10 или Windows Server для контейнеров, а затем перейдите к нужному первому образу контейнера.
keywords: Dock, Containers, ЛКОВ
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288122"
---
# <a name="get-started-prep-windows-for-containers"></a>Начало работы: подготовка окон подсистемы для контейнеров

В этом руководстве описано, как выполнять указанные ниже действия.

- Настройка Windows 10 или Windows Server для контейнеров
- Выполнение первого изображения контейнера
- Контаинеризе простого приложения .NET Core

## <a name="prerequisites"></a>Что вам понадобится

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Для работы с контейнерами на компьютере с Windows Server требуется физический сервер или виртуальная машина под управлением Windows Server (половина канала), Windows Server 2019 или Windows Server 2016.

Для тестирования вы можете скачать копию Windows [server 2019 Evaluation или ознакомительную](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) версию программы [предварительной оценки](https://insider.windows.com/for-business-getting-started-server/).

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

Для выполнения контейнеров в Windows 10 вам понадобятся следующие возможности:

- Одна физическая компьютерная система под управлением Windows 10 профессиональная или Корпоративная с обновлением юбилея (версия 1607) или более поздней.
- [Технология Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) должна быть включена.

> [!NOTE]
>  Начиная с Windows 10, Октябрь обновление 2018, мы больше не запрещают пользователям запускать контейнер Windows в режиме изоляции процессов в Windows 10 Корпоративная или профессиональная для разработки и тестирования. Дополнительные сведения можно найти в разделе " [часто задаваемые вопросы](../about/faq.md) ". 
> 
> Контейнеры Windows Server по умолчанию используют изоляцию Hyper-V в Windows 10 для предоставления разработчикам одной и той же версии ядра и конфигурации, которая будет использоваться в производстве. Дополнительные сведения об изоляции Hyper-V содержатся в разделе [основные положения](../manage-containers/hyperv-container.md) наших документов.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Установка Docker

Первый этап — Установка Dock, необходимого для работы с контейнерами Windows. Dock предоставляет стандартную среду выполнения для контейнеров с общим API и интерфейсом командной строки (CLI).

Дополнительные сведения о конфигурации можно найти [в разделе Подсистема стыковки в Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Чтобы установить Dock на Windows Server, можно использовать [модуль PowerShell поставщика онежет](https://github.com/oneget/oneget) , опубликованный корпорацией Майкрософт, который называется [доккермикрософтпровидер](https://github.com/OneGet/MicrosoftDockerProvider). Этот поставщик включает функцию контейнеров в Windows и устанавливает подсистему и клиентскую подкрепление. Вот как это сделать.

1. Откройте сеанс PowerShell с повышенными привилегиями и установите Dock-поставщик Microsoft PackageManagement из [коллекции PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Если вам будет предложено установить поставщик NuGet, введите `Y` его также и для его установки.

2. Используйте модуль PowerShell PackageManagement, чтобы установить последнюю версию Dock.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Когда в PowerShell появится запрос, доверять ли источнику пакета DockerDefault, введите `A`, чтобы продолжить установку.
3. После завершения установки перезагрузите компьютер.

   ```powershell
   Restart-Computer -Force
   ```

Если вы хотите обновить закрепление позже, выполните указанные ниже действия.

- Проверьте установленную версию с помощью команды `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- Найдите текущую версию с помощью команды `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- После этого выполните обновление с помощью команды `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` и команды `Start-Service Docker`

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

Вы можете установить Dock в выпусках Windows 10 профессиональная и Enterprise Edition, выполнив указанные ниже действия. 

1. Скачайте и установите [закрепление на компьютере](https://store.docker.com/editions/community/docker-ce-desktop-windows), чтобы создать бесплатную учетную запись Dock, если у вас ее еще нет. Дополнительные сведения можно найти в [документации по Dock](https://docs.docker.com/docker-for-windows/install).

2. Во время установки задайте тип контейнера по умолчанию для контейнеров Windows. Чтобы переключиться после завершения установки, можно использовать либо элемент Dock в панели задач Windows (как показано ниже), либо следующую команду в командной строке PowerShell:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Меню панели задач "закрепляемое окно" с командой "перейти в контейнеры Windows".](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда ваша среда настроена правильно, перейдите по ссылке, чтобы получить сведения о том, как выполнить контейнер.

> [!div class="nextstepaction"]
> [Выполнение первого контейнера](./run-your-first-container.md)
