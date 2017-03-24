---
title: "Контейнеры Windows в Windows Server"
description: "Краткое руководство по развертыванию контейнеров"
keywords: "docker, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 76e041aac426604280208f616f7994181112215a
ms.openlocfilehash: 766a99a74738fa41ef77410c70aefa7e664f014e
ms.lasthandoff: 03/01/2017

---

# Контейнеры Windows в Windows Server

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows Server 2016. Во время этого упражнения вы установите роль контейнера и развернете простой контейнер Windows Server. Перед началом работы с этим кратким руководством ознакомьтесь с основными понятиями и терминологией для контейнеров. Эти сведения можно найти в статье [Знакомство с кратким руководством](./index.md).

В этом кратком руководстве рассматриваются контейнеры Windows Server в Windows Server 2016. Дополнительную документацию к краткому руководству, в том числе по контейнерам в Windows 10, можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

Одна компьютерная система (физическая или виртуальная), работающая под управлением Windows Server 2016. Если вы используете Windows Server 2016 TP5, обновите [Window Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Критические обновления необходимы для работы контейнеров Windows. Установите все обновления перед выполнением этого учебника.

Если вы хотите провести развертывание в Azure, это легко сделать с помощью следующего [шаблона](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template).<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## 1. Установка Docker

Для установки Docker будет использоваться [модуль PowerShell поставщика OneGet](https://github.com/oneget/oneget). Поставщик включит функцию контейнеров на вашем компьютере. Вы также установите Docker, после чего потребуется перезагрузка. Docker необходим для работы с контейнерами Windows. Он состоит из подсистемы Docker и клиента Docker.

Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

Сначала установите поставщик Docker-Microsoft PackageManagement из коллекции PowerShell.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Затем можно использовать модуль PackageManagement PowerShell, чтобы установить последнюю версию Docker.
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Когда в PowerShell появится запрос, доверять ли источнику пакета DockerDefault, введите `A`, чтобы продолжить установку. После завершения установки перезагрузите компьютер.

```none
Restart-Computer -Force
```

## 2. Установка обновлений Windows

Убедитесь, что система Windows Server обновлена, запустив следующее:

```none
sconfig
```

Появится текстовое меню настройки, где можно будет выбрать вариант 6 (скачивание и установка обновлений):

```none
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

При появлении запроса выберите вариант A (скачать все обновления).

## 3. Развертывание первого контейнера

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
Platform: .NET Core 1.0
OS: Microsoft Windows 10.0.14393
```

Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com]( https://docs.docker.com/engine/reference/run/).

## Дальнейшие действия

[Образы контейнеров в Windows Server](./quick-start-images.md)

[Контейнеры Windows в Windows 10](./quick-start-windows-10.md)

