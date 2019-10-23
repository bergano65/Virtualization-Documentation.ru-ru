---
title: Использование контейнеров с программой предварительной оценки Windows
description: Узнайте, как начать работу с контейнерами Windows с помощью программы предварительной оценки Windows
keywords: Dock, контейнеры, программа предварительной оценки, Windows
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254416"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Использование контейнеров с программой предварительной оценки Windows

В этом упражнении вы узнаете, как развернуть и использовать контейнеры Windows в последней сборке Windows Server для участников программы предварительной оценки Windows. Во время этого упражнения вы установите роль контейнера и развернете предварительный выпуск базовых образов ОС. Если вам необходимо ознакомиться с контейнерами, изучите раздел [О контейнерах](../about/index.md).

> [!NOTE]
> Это содержимое зависит от контейнеров Windows Server в программе предварительной оценки Windows Server. Если вы ищете инструкции, не предназначенные для предварительной оценки, для работы с контейнерами Windows, ознакомьтесь с руководством Приступая к [работе](../quick-start/set-up-environment.md) .

## <a name="join-the-windows-insider-program"></a>Присоединяйтесь к программе предварительной оценки Windows

Чтобы запустить версию программы предварительной оценки для контейнеров Windows, на компьютере должна быть установлена новейшая сборка Windows Server в программе предварительной оценки Windows и/или новейшая сборка Windows 10 из программы предварительной оценки Windows. Присоединяйтесь к [программе предварительной оценки Windows](https://insider.windows.com/GettingStarted) и ознакомьтесь с условиями использования.

> [!IMPORTANT]
> Вы должны использовать сборку Windows Server из программы предварительной оценки Windows Server или для сборки Windows 10 из программы предварительной оценки Windows, чтобы использовать базовое изображение, описанное ниже. Если у вас не установлена одна из этих сборок, использование этих базовых образов приведет к сбою запуска контейнера.

## <a name="install-docker"></a>Установка Docker

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Программа предварительной оценки Windows Server](#tab/Windows-Server-Insider)

Для установки Docker EE будет использоваться модуль PowerShell поставщика OneGet. Поставщик обеспечит работу контейнеров на компьютере и установит Docker EE. После этого потребуется перезагрузка. Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

> [!NOTE]
> Для установки дока в сборках программы предварительной оценки Windows Server требуется иной поставщик Онежет, чем тот, который используется для сборок, не являющихся участниками программы предварительной оценки. Если Docker EE и поставщик DockerMsftProvider OneGet уже установлены, удалите их перед продолжением.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Установите модуль OneGet PowerShell для использования в сборках программы предварительной оценки Windows.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

С помощью OneGet установите последнюю версию Docker EE Preview.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

После завершения установки перезагрузите компьютер.

```powershell
Restart-Computer -Force
```

# [<a name="windows-10-insider"></a>Программа предварительной оценки Windows 10](#tab/Windows-10-Insider)

В рамках программы предварительной оценки Windows 10 приложение Dock Edge устанавливается с помощью того же установщика, что и настольный компьютер стыковочного устройства. Скачайте [закрепление на компьютере](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите установщик. Вам потребуется выполнить вход. Создайте учетную запись, если она еще не установлена. Более детальные инструкции по установке можно найти в [документации по Dock](https://docs.docker.com/docker-for-windows/install).

После установки откройте настройки Dock и перейдите к каналу Edge.

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>Вытяните изображение из контейнера для участников программы предварительной оценки

Перед началом работы с контейнерами Windows необходимо установить базовый образ. После того как вы участвуете в программе предварительной оценки Windows, вы можете использовать наши последние сборки для базовых изображений. Вы можете прочитать дополнительные сведения о доступных базовых изображениях в документе " [базовые изображения контейнера](../manage-containers/container-base-images.md) ".

Чтобы получить базовый образ Nano Server для участников программы предварительной оценки, выполните следующую команду:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Чтобы получить базовый образ Windows Server Core для участников программы предварительной оценки, выполните следующую команду:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Пожалуйста, прочтите [лицензионное соглашение](../images-eula.md ) на использование образа ОС для контейнеров Windows и [условия использования](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)программы предварительной оценки Windows.
