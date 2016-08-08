---
title: "Контейнер Windows в Windows 10"
description: "Краткое руководство по развертыванию контейнеров"
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 07/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: bd93f5a73268b552710304d7da568e1497239679

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

После перезагрузки выполните следующую команду, чтобы устранить известную проблему с версией Technical Preview контейнеров Windows.  

 ```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

## 2. Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента. Для этого выполните приведенные ниже команды. 

Скачайте подсистему Docker и клиент в виде ZIP-архива.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Разархивируйте ZIP-архив в Program Files; содержимое архива уже находится в каталоге Docker.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Добавьте каталог Docker в системный путь.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

Перезапустите сеанс PowerShell для распознавания измененного пути.

Чтобы установить Docker в качестве службы Windows, выполните следующую команду:

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
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

В этом простом примере будет создан и развернут образ контейнера "Привет мир". Для получения наилучших результатов выполните следующие команды в оболочке CMD Windows с повышенными привилегиями.

Сначала запустите контейнер с помощью интерактивного сеанса из образа `nanoserver`. После запуска контейнера в его рамках появится командная оболочка.  

```none
docker run -it nanoserver cmd
```

Внутри контейнера создайте простой скрипт "Привет мир".

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

После завершения выйдите из контейнера.

```none
exit
```

Теперь создайте новый образ контейнера из измененного контейнера. Чтобы просмотреть список контейнеров, выполните следующую команду и запишите идентификатор контейнера.

```none
docker ps -a
```

Выполните следующую команду для создания нового образа "Привет мир". Замените <containerid> на идентификатор контейнера.

```none
docker commit <containerid> helloworld
```

После завершения вы получите пользовательский образ, содержащий скрипт "Привет мир". Чтобы убедиться в этом, используйте следующую команду.

```none
docker images
```

Наконец, чтобы запустить контейнер, используйте команду `docker run`.

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

В результате выполнения команды `docker run` создается контейнер Hyper-V из образа "Привет мир", затем запускается скрипт-пример "Привет мир" (выходные данные при этом выводятся в оболочке), после чего контейнер останавливается и удаляется. В последующих кратких руководствах, посвященных Windows 10 и контейнерам, будет подробно описано создание и развертывание приложений в контейнерах на базе Windows 10.

## Дальнейшие шаги

[Контейнеры Windows в Windows Server](./quick_start_windows_server.md)





<!--HONumber=Aug16_HO1-->


