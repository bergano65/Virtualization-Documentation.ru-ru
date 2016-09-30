---
title: "Контейнеры Windows в Windows Server"
description: "Краткое руководство по развертыванию контейнеров"
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 891c9e9805bf2089fd11f86420de5ed251916c3f
ms.openlocfilehash: 77dca1499abf406b1d599c28afdb19dd823b8401

---

# Контейнеры Windows в Windows Server

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows Server. В его рамках вы установите роль контейнера и развернете простой контейнер Windows Server. Перед началом работы с этим кратким руководством ознакомьтесь с основными понятиями и терминологией для контейнеров. Эти сведения можно найти в статье [Знакомство с кратким руководством](./quick_start.md).

В этом кратком руководстве рассматриваются контейнеры Windows Server в Windows Server 2016. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

Одна компьютерная система (физическая или виртуальная), работающая под управлением Windows Server 2016. Если вы используете Windows Server 2016 TP5, обновите [Window Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

## 1. Установка компонента контейнеров

Чтобы начать работу с контейнерами Windows, требуется включить контейнер компонентов. Для этого выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```none
Install-WindowsFeature containers
```

После завершения установки компонента перезагрузите компьютер.

```none
Restart-Computer -Force
```

## 2. Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента.

Скачайте версию-кандидат коммерчески поддерживаемого механизма Docker и клиент в виде ZIP-архива.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Разархивируйте ZIP-архив в Program Files.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Добавьте каталог Docker в системный путь.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Чтобы установить Docker в качестве службы Windows, выполните следующую команду.

```none
dockerd.exe --register-service
```

После установки эту службу можно запустить.

```none
Start-Service docker
```

## 3. Развертывание первого контейнера

В этом упражнении вы скачаете предварительно созданный пример образа .NET из реестра Docker Hub и развернете простой контейнер с приложением .NET Hello World.  

Используйте команду `docker run` для развертывания контейнера .NET. Эта команда также скачает образ контейнера, что может занять несколько минут.

```none
docker run microsoft/sample-dotnet
```

Контейнер запустится, выведет сообщение "Hello world" и завершит работу.

```none
       Welcome to .NET Core!
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
```

Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com]( https://docs.docker.com/engine/reference/run/).

## Дальнейшие действия

[Образы контейнеров в Windows Server](./quick_start_images.md)

[Контейнеры Windows в Windows 10](./quick_start_windows_10.md)


<!--HONumber=Sep16_HO4-->


