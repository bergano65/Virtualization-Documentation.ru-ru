---
title: "Контейнер Windows в Windows10"
description: "Краткое руководство по развертыванию контейнеров"
keywords: "docker, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a6ed0cccb984d303990973a1e2009cc2922f9443
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: ru-RU
---
# <a name="windows-containers-on-windows-10"></a>Контейнеры Windows в Windows10

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows10 Профессиональная и Windows10 Корпоративная (Anniversary Edition). В его рамках вы установите Docker для Windows и запустите простой контейнер. Перед началом работы с этим кратким руководством ознакомьтесь с основными понятиями и терминологией для контейнеров. Эти сведения можно найти в статье [Знакомство с кратким руководством](./index.md).

Это краткое руководство применимо только к Windows 10. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

- Одна физическая компьютерная система под управлением Windows 10 Anniversary Edition или Creators Update (Профессиональная или Корпоративная).   
- Это краткое руководство можно выполнять на виртуальной машине Windows10, однако при этом нужно включить вложенную виртуализацию. Дополнительные сведения см. в [руководстве по вложенной виртуализации](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Чтобы контейнеры Windows работали, необходимо установить критические обновления.
> Чтобы узнать версию ОС, запустите `winver.exe` и сравните указанную версию с версией в [журнале обновлений Windows10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history).
> Убедитесь, что у вас установлена версия 14393.222 или более поздняя, перед тем как продолжить.

## <a name="1-install-docker-for-windows"></a>1. Установка Docker для Windows

[Скачайте Docker для Windows](https://download.docker.com/win/stable/InstallDocker.msi) и запустите программу установки. [Подробные инструкции по установке](https://docs.docker.com/docker-for-windows/install) доступны в документации по Docker.

## <a name="2-switch-to-windows-containers"></a>2. Переключение на контейнеры Windows

После установки в Docker для Windows по умолчанию будут запущены контейнеры Linux. Переключитесь на контейнеры Windows, используя меню области уведомлений Docker или выполнив следующую команду в командной строке PowerShell `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)

## <a name="3-install-base-container-images"></a>3. Установка базовых образов контейнеров

Контейнеры Windows создаются из базовых образов. Приведенная ниже команда извлекает базовый образ Nano Server.

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

## <a name="4-run-your-first-container"></a>4. Запуск первого контейнера

В этом простом примере будет создан и развернут образ контейнера "Hello World". Для получения наилучших результатов выполните следующие команды в командной строке Windows или PowerShell с повышенными привилегиями.

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
В последующих кратких руководствах, посвященных Windows10 и контейнерам, будет подробно описано создание и развертывание приложений в контейнерах на базе Windows10.

## <a name="next-steps"></a>Дальнейшие шаги

[Контейнеры Windows в Windows Server](./quick-start-windows-server.md)
