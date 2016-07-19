---
title: "Контейнер Windows в Windows 10"
description: "Краткое руководство по развертыванию контейнеров"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/07/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 5f42cae373b1f8f0484ffac82f5ebc761c37d050
ms.openlocfilehash: 9ef41ff031e8b7bc463e71f39ee6a3b8e4fd846e

---

# Контейнеры Windows в Windows 10

**Это предварительное содержимое. Возможны изменения.** 

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows 10 (начиная со сборки 14372 для участников программы предварительной оценки). В его рамках вы установите роль контейнера и развернете простой контейнер Hyper-V. Перед началом работы с этим кратким руководством ознакомьтесь с основными понятиями и терминологией для контейнеров. Эти сведения можно найти в статье [Знакомство с кратким руководством](./quick_start.md). 

В этом кратком руководстве рассматриваются контейнеры Hyper-V в Windows 10. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

- Одна физическая компьютерная система под управлением [выпуск Windows 10 для предварительной оценки](https://insider.windows.com/).   
- Это краткое руководство можно выполнять на виртуальной машине Windows 10, однако при этом нужно включить вложенную виртуализацию. Дополнительные сведения см. в [руководстве по вложенной виртуализации](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

## 1. Установка компонента контейнеров

Чтобы начать работу с контейнерами Windows, требуется включить контейнер компонентов. Для этого выполните приведенную ниже команду в сеансе PowerShell с повышенными правами. 

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

Поскольку Windows 10 поддерживает только контейнеры Hyper-V, необходимо также включить компонент Hyper-V. Чтобы включить компонент Hyper-V с помощью PowerShell, выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

После завершения установки перезагрузите компьютер.

```none
Restart-Computer -Force
```

## 2. Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента. Для этого выполните приведенные ниже команды. 

Создайте папку для исполняемых файлов Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles\docker\
```

Скачайте управляющую программу Docker.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/dockerd.exe -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Загрузите клиент Docker.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/docker.exe -OutFile $env:ProgramFiles\docker\docker.exe
```

Добавьте каталог Docker в системный путь.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

Перезапустите сеанс PowerShell для распознавания измененного пути.

Чтобы установить Docker в качестве службы Windows, выполните следующую команду:

```none
dockerd --register-service
```

После установки эту службу можно запустить.

```none
Start-Service Docker
```

## 3. Установка базовых образов контейнеров

Контейнеры Windows развертываются из шаблонов или образов. Перед развертыванием контейнера требуется скачать базовый образ ОС контейнера. Приведенные ниже команды скачивают базовый образ Nano Server.
    
> Эта процедура относится к более поздней, чем 14372, версии сборки для участников программы предварительной оценки Windows и является временной до начала работы Docker Pull.

Скачайте базовый образ Nano Server. 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

Установите базовый образ.

```none  
docker load -i nanoserver.tar.gz
```

На этом этапе выполнение `docker images` возвращает список установленных образов — в данном случае это образ Nano Server.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

Перед продолжением этот образ следует пометить как последнюю версию "latest". Для этого выполните следующую команду:

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

Дополнительные сведения об образах контейнеров Windows см. в статье [Управление образами контейнеров](../management/manage_images.md).

## 4. Развертывание первого контейнера

Для этого простого примера используется готовый образ .NET Core. Скачайте образ с помощью команды `docker pull`.

При выполнении открывается контейнер, запускается простое приложение .NET Core, после чего контейнер закрывается. 

```none
docker pull microsoft/sample-dotnet
```

Это можно проверить с помощью команды `docker images`.

```none
docker 

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

Запустите контейнер с помощью команды `docker run`. Следующий пример задает параметр `--rm`, давая подсистеме Docker указание удалить контейнер после завершения его работы. 

Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com]( https://docs.docker.com/engine/reference/run/).

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

**Примечание**. При возникновении ошибки, указывающей на истечение времени ожидания, запустите следующий скрипт PowerShell и повторите операцию.

```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

В результате выполнения команды `docker run` создается контейнер Hyper-V из примера образа dotnet, затем запускается образец приложения (выходные данные при этом выводятся в оболочке), после чего контейнер останавливается и удаляется. В последующих кратких руководствах, посвященных Windows 10 и контейнерам, будет подробно описано создание и развертывание приложений в контейнерах на базе Windows 10.

## Дальнейшие шаги

[Контейнеры Windows в Windows Server](./quick_start_windows_server.md)





<!--HONumber=Jul16_HO2-->


