---
title: Контейнеры Windows в Windows Server
description: Краткое руководство по развертыванию контейнеров
keywords: docker, контейнеры
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 7d526aa64d478516a3f66acaf62b62b45282e5af
ms.sourcegitcommit: 3d72f15651da378908f134916cd5c9d2064f8f95
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/12/2018
ms.locfileid: "2256937"
---
# <a name="windows-containers-on-windows-server"></a>Контейнеры Windows в Windows Server

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows Server2016. Во время этого упражнения вы установите роль контейнера и развернете простой контейнер Windows Server. Если вам необходимо ознакомиться с контейнерами, изучите раздел [О контейнерах](../about/index.md).

В этом кратком руководстве рассматриваются контейнеры Windows Server в Windows Server2016. Дополнительную документацию к краткому руководству, в том числе по контейнерам в Windows10, можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

Одна компьютерная система (физическая или виртуальная), работающая под управлением Windows Server2016. Если вы используете Windows Server2016TP5, обновите [Window Server2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ).

> Критические обновления необходимы для работы контейнеров Windows. Установите все обновления перед выполнением этого учебника.

Если вы хотите провести развертывание в Azure, это легко сделать с помощью следующего [шаблона](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template).<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="1-install-docker"></a>1. Установка Docker

Для установки Docker будет использоваться [модуль PowerShell поставщика OneGet](https://github.com/oneget/oneget), который работает с поставщиками для выполнения установки (в нашем случае это поставщик [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider)). Поставщик включит функцию контейнеров на вашем компьютере. Вы также установите Docker, после чего потребуется перезагрузка. Docker необходим для работы с контейнерами Windows. Он состоит из подсистемы Docker и клиента Docker.

Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

Сначала установите поставщик Docker-Microsoft PackageManagement из [коллекции PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Затем можно использовать модуль PackageManagement PowerShell, чтобы установить последнюю версию Docker.
```
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Когда в PowerShell появится запрос, доверять ли источнику пакета DockerDefault, введите `A`, чтобы продолжить установку. После завершения установки перезагрузите компьютер.

```
Restart-Computer -Force
```

> Совет. Если потребуется обновить Docker позже, выполните следующие действия.
>  - Проверьте установленную версию с помощью команды `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Найдите текущую версию с помощью команды `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - После этого выполните обновление с помощью команды `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` и команды `Start-Service Docker`

## <a name="2-install-windows-updates"></a>2. Установка обновлений Windows

Убедитесь, что система Windows Server обновлена, запустив следующее:

```
sconfig
```

Появится текстовое меню настройки, где можно будет выбрать вариант6 (скачивание и установка обновлений):

```
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

## <a name="3-deploy-your-first-container"></a>3. Развертывание первого контейнера

В этом упражнении вы скачаете предварительно созданный пример образа .NET из реестра Docker Hub и развернете простой контейнер с приложением .NET Hello World.  

Используйте команду `docker run` для развертывания контейнера .Net. Эта команда также скачает образ контейнера, что может занять несколько минут.

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

Контейнер запустится, выведет сообщение "Hello world" и завершит работу.

```console
         Dotnet-bot: Welcome to using .NET Core!
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
Platform: .NET Core 2.0
OS: Microsoft Windows 10.0.14393
```

Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com]( https://docs.docker.com/engine/reference/run/).

## <a name="next-steps"></a>Дальнейшие действия

[Автоматизация сборки и сохранения образов](./quick-start-images.md)
