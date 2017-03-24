---
title: "Контейнер Windows в Windows 10"
description: "Краткое руководство по развертыванию контейнеров"
keywords: "docker, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 996d3b1a8f7c8325ac66d331e1d62208c0cf6b53
ms.openlocfilehash: 091a3570291624a3be40e3aabb9f99a482cb6470
ms.lasthandoff: 02/27/2017

---

# Контейнеры Windows в Windows 10

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows 10 Professional и Windows 10 Корпоративная (Anniversary Edition). В его рамках вы установите роль контейнера и развернете простой контейнер Hyper-V. Перед началом работы с этим кратким руководством ознакомьтесь с основными понятиями и терминологией для контейнеров. Эти сведения можно найти в статье [Знакомство с кратким руководством](./index.md).

В этом кратком руководстве рассматриваются контейнеры Hyper-V в Windows 10. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

- Одна физическая компьютерная система под управлением Windows 10 Anniversary Edition (Профессиональная или Корпоративная).   
- Это краткое руководство можно выполнять на виртуальной машине Windows 10, однако при этом нужно включить вложенную виртуализацию. Дополнительные сведения см. в [руководстве по вложенной виртуализации](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Чтобы контейнеры Windows работали, необходимо установить критические обновления. 
> Чтобы узнать версию ОС, запустите `winver.exe` и сравните указанную версию с версией в [журнале обновлений Windows 10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history). 
> Убедитесь, что у вас установлена версия 14393.222 или более поздняя перед тем, как продолжить.

## 1. Установка компонента контейнеров

Чтобы начать работу с контейнерами Windows, требуется включить контейнер компонентов. Для этого выполните приведенную ниже команду в сеансе PowerShell с **повышенными привилегиями**.

Если появилось сообщение о том, что `Enable-WindowsOptionalFeature` не существует, убедитесь, что вы запустили PowerShell от имени администратора.

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

> Если вы ранее использовали контейнеры Hyper-V в Windows 10 с базовыми образами контейнера Technical Preview 5, не забудьте снова включить OpLocks. Выполните следующую команду:  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2. Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента. Для этого выполните приведенные ниже команды.

Скачайте подсистему Docker и клиент в виде ZIP-архива.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-17.03.0-ce.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Разархивируйте ZIP-архив в Program Files; содержимое архива уже находится в каталоге Docker.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Добавьте каталог Docker в системный путь.

```none
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Чтобы установить Docker в качестве службы Windows, выполните следующую команду.

```none
dockerd --register-service
```

После установки эту службу можно запустить.

```none
Start-Service Docker
```

## 3. Установка базовых образов контейнеров

Контейнеры Windows развертываются из шаблонов или образов. Перед развертыванием контейнера требуется скачать базовый образ ОС контейнера. Приведенные ниже команды скачивают базовый образ Nano Server.

Получите базовый образ Nano Server.

```none
docker pull microsoft/nanoserver
```

После его получения выполните команду `docker images`, чтобы вывести список установленных образов. В данном случае будет отображен образ Nano Server.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Прочтите лицензионное соглашение для образов ОС контейнеров Windows на странице [Лицензионное соглашение](../images-eula.md).

## 4. Развертывание первого контейнера

В этом простом примере будет создан и развернут образ контейнера "Привет мир". Для получения наилучших результатов выполните следующие команды в командной строке Windows или PowerShell с повышенными привилегиями.

> Интегрированная среда сценариев Windows PowerShell не работает в интерактивных сеансах с контейнерами. Даже если контейнер запускается, он зависает.

Сначала запустите контейнер с помощью интерактивного сеанса из образа `nanoserver`. После запуска контейнера в его рамках появится командная оболочка.  

```none
docker run -it microsoft/nanoserver cmd
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

В результате выполнения команды `docker run` создается контейнер Hyper-V из образа "Привет мир", затем запускается скрипт-пример "Привет мир" (выходные данные при этом выводятся в оболочке), после чего контейнер останавливается и удаляется.
В последующих кратких руководствах, посвященных Windows 10 и контейнерам, будет подробно описано создание и развертывание приложений в контейнерах на базе Windows 10.

## Дальнейшие шаги

[Контейнеры Windows в Windows Server](./quick-start-windows-server.md)

