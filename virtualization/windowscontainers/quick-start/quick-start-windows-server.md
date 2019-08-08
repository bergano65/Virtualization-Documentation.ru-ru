---
title: Контейнеры Windows в Windows Server
description: Краткое руководство по развертыванию контейнеров
keywords: docker, контейнеры
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 665e6186ff5f9530f12ba3d0a400d82bcac755a7
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999201"
---
# <a name="windows-containers-on-windows-server"></a>Контейнеры Windows в Windows Server

В этом упражнении вы ознакомитесь с базовым развертыванием и использованием функции контейнера Windows в Windows Server 2019 и Windows Server 2016.

Это краткое руководство поможет вам сделать следующее:

1. Включение функции контейнеров в Windows Server
2. Установка стыковочного узла
3. Выполнение простого контейнера Windows

Если вам необходимо ознакомиться с контейнерами, изучите раздел [О контейнерах](../about/index.md).

Это краткое руководство относится только к контейнерам Windows Server в Windows Server 2019 и Windows Server 2016. Дополнительную документацию к краткому руководству, в том числе по контейнерам в Windows10, можно найти в содержании в левой части этой страницы.

## <a name="prerequisites"></a>Что вам понадобится

Убедитесь, что вы отвечаете на следующие требования:
- Одна компьютерная система (физическая или виртуальная) под управлением Windows Server 2019. Если вы используете Windows Server 2019 Insider Preview, пожалуйста, обновите [ознакомительную версию до Window server 2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ).

> Для функционирования функции контейнера Windows необходимо, чтобы были важны важные обновления. Установите все обновления перед выполнением этого учебника.

Если вы хотите провести развертывание в Azure, это легко сделать с помощью следующего [шаблона](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template).

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>Установка Docker

Чтобы установить Dock, мы будем использовать [модуль PowerShell поставщика онежет](https://github.com/oneget/oneget) , который работает с поставщиками для выполнения установки, в этом случае [микрософтдоккерпровидер](https://github.com/OneGet/MicrosoftDockerProvider). Поставщик включит функцию контейнеров на вашем компьютере. Вы также установите Docker, после чего потребуется перезагрузка. Docker необходим для работы с контейнерами Windows. Он состоит из подсистемы Docker и клиента Docker.

Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

Сначала установите поставщик Docker-Microsoft PackageManagement из [коллекции PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Затем можно использовать модуль PackageManagement PowerShell, чтобы установить последнюю версию Docker.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Когда в PowerShell появится запрос, доверять ли источнику пакета DockerDefault, введите `A`, чтобы продолжить установку. После завершения установки перезагрузите компьютер.

```powershell
Restart-Computer -Force
```

> [!TIP]
> Если вы хотите обновить закрепление позже, выполните указанные ниже действия.
>  - Проверьте установленную версию с помощью команды `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Найдите текущую версию с помощью команды `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - После этого выполните обновление с помощью команды `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` и команды `Start-Service Docker`

## <a name="install-windows-updates"></a>Установка обновлений Windows

Убедитесь, что система Windows Server обновлена, запустив следующее:

```console
sconfig
```

Появится текстовое меню настройки, где можно будет выбрать вариант6 (скачивание и установка обновлений):

```console
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

При появлении запроса выберите вариантA (скачать все обновления).

## <a name="deploy-your-first-container"></a>Развертывание первого контейнера

В этом упражнении вы скачаете предварительно созданный пример образа .NET из реестра Docker Hub и развернете простой контейнер с приложением .NET Hello World.  

Используйте команду `docker run` для развертывания контейнера .Net. Эта команда также скачает образ контейнера, что может занять несколько минут. В зависимости от версии Windows Server, используемой на вашем компьютере, выполните следующую команду ниже.

#### <a name="windows-server-2019"></a>WindowsServer2019

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

#### <a name="windows-server-2016"></a>WindowsServer2016

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-sac2016
```

Контейнер запустится, выведет сообщение "Hello world" и завершит работу.

```console
         Hello from .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....


**Environment**
Platform: .NET Core
OS: Microsoft Windows 10.0.17763
```

Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com](https://docs.docker.com/engine/reference/run/).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о том, как автоматизировать построения контейнера и сохранение изображений.](./quick-start-images.md)
