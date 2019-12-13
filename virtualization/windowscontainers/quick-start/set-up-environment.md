---
title: Контейнеры Windows и Linux в Windows 10
description: Настройте Windows 10 или Windows Server для контейнеров, а затем перейдите к запуску первого образа контейнера.
keywords: DOCKER, контейнеры, ЛКОВ
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909564"
---
# <a name="get-started-prep-windows-for-containers"></a>Начало работы: окна подготовки для контейнеров

В этом учебнике описывается:

- Настройка Windows 10 или Windows Server для контейнеров
- Запуск первого образа контейнера
- Контейнеризовать простое приложение .NET Core

## <a name="prerequisites"></a>Предварительные условия

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Для запуска контейнеров в Windows Server требуется физический сервер или виртуальная машина под управлением Windows Server (полугодовой канал), Windows Server 2019 или Windows Server 2016.

Для тестирования можно загрузить копию [ознакомительной версии Window server 2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) или [предварительной версии Windows Server Insider](https://insider.windows.com/for-business-getting-started-server/)Preview.

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Для запуска контейнеров в Windows 10 необходимо следующее:

- Одна физическая компьютерная система под управлением Windows 10 профессиональная или Корпоративная с годовщиной обновления (версии 1607) или более поздней.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) должен быть включен.

> [!NOTE]
>  Начиная с Windows 10 октября с обновлением 2018, мы больше не запрещают пользователям запускать контейнер Windows в режиме изоляции процессов в Windows 10 Корпоративная или Professional для целей разработки и тестирования. Дополнительные сведения см. в разделе [часто задаваемые вопросы](../about/faq.md) . 
> 
> Контейнеры Windows Server по умолчанию используют изоляцию Hyper-V в Windows 10 для предоставления разработчикам той же версии и конфигурации ядра, которая будет использоваться в рабочей среде. Дополнительные сведения о изоляции Hyper-V см. в области " [Основные понятия](../manage-containers/hyperv-container.md) " наших документов.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Установка Docker

Первым шагом является установка DOCKER, необходимого для работы с контейнерами Windows. DOCKER предоставляет стандартную среду выполнения для контейнеров с общим API и интерфейсом командной строки (CLI).

Дополнительные сведения о конфигурации см. [в разделе Подсистема DOCKER в Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Чтобы установить DOCKER в Windows Server, можно использовать [модуль PowerShell поставщика OneGet](https://github.com/oneget/oneget) , опубликованный корпорацией Майкрософт под названием [доккермикрософтпровидер](https://github.com/OneGet/MicrosoftDockerProvider). Этот поставщик включает функцию контейнеров в Windows и устанавливает подсистему и клиент DOCKER. Вот как это сделать.

1. Откройте сеанс PowerShell с повышенными привилегиями и установите поставщик DOCKER-Microsoft на основе [коллекция PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Если будет предложено установить поставщик NuGet, введите также `Y`, чтобы установить его.

2. Используйте модуль PackageManagement PowerShell для установки последней версии DOCKER.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Когда в PowerShell появится запрос, доверять ли источнику пакета DockerDefault, введите `A`, чтобы продолжить установку.
3. После завершения установки перезагрузите компьютер.

   ```powershell
   Restart-Computer -Force
   ```

Если вы хотите обновить DOCKER позже:

- Проверьте установленную версию с помощью `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- Найти текущую версию с помощью `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- Когда будете готовы, выполните обновление до `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, а затем `Start-Service Docker`

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Вы можете установить DOCKER в выпусках Windows 10 профессиональная и Enterprise, выполнив следующие действия. 

1. Скачайте и установите [DOCKER Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows), создав бесплатную учетную запись DOCKER, если она еще не установлена. Дополнительные сведения см. в [документации по DOCKER](https://docs.docker.com/docker-for-windows/install).

2. Во время установки укажите контейнеры Windows по умолчанию. Чтобы переключиться после завершения установки, можно использовать либо элемент DOCKER в области уведомлений Windows (как показано ниже), либо следующую команду в командной строке PowerShell:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Меню области уведомлений DOCKER, в котором отображается команда "переключиться на контейнеры Windows".](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда среда настроена правильно, перейдите по ссылке, чтобы узнать, как запустить контейнер.

> [!div class="nextstepaction"]
> [Запуск первого контейнера](./run-your-first-container.md)
