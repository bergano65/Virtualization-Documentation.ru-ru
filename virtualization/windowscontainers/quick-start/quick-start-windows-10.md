---
title: Контейнеры Linux в Windows 10 и Windows
description: Краткое руководство по развертыванию контейнеров
keywords: docker, контейнеры, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224903"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Контейнеры Linux в Windows 10 и Windows

Упражнение описывается создание и запуск контейнеров Windows и Linux в Windows 10. После завершения у вас будет:

1. Установленные Docker для Windows
2. Запустите простой контейнер Windows
3. Запустите простой контейнер Linux с помощью контейнеров Linux в Windows (LCOW)

Это краткое руководство применимо только к Windows 10. Краткое руководство по дополнительной документации можно найти в содержании в левой части этой страницы.

***Изоляции Hyper-V:*** Контейнеры Windows Server требуется изоляции Hyper-V в Windows 10, чтобы разработчики с тем же версию ядра и конфигурации, который будет использоваться в рабочей среде, более сведения о Hyper-V изоляции можно найти на странице " [сведения о контейнерах Windows](../about/index.md) ".

**Предварительные требования:**

- Одна физическая компьютерная система под управлением Windows 10 Fall Creators Update (версия 1709) или более поздней версии (Профессиональная или Корпоративная), [можно запустить Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)

> Если вы не хотите выполнить LCOW в этом руководстве, контейнеры Windows будет выполняться на Юбилейное обновление Windows 10 (версия 1607) или более поздней версии.

## <a name="1-install-docker-for-windows"></a>1. Установка Docker для Windows

[Скачайте Docker для Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите установщик (вам потребуется для входа в систему. Создайте учетную запись, если ее у вас нет уже). [Подробные инструкции по установке](https://docs.docker.com/docker-for-windows/install) доступны в документации по Docker.

> Если у вас уже есть установить Docker, убедитесь, что у вас есть версии 18.02 или более поздней, для поддержки LCOW. Проверьте, запустив `docker -v` или проверки *О Docker*.

> Параметр экспериментальных функций в *Docker параметры > управляющей программы* для запуска контейнеров LCOW необходимо активировать.

## <a name="2-switch-to-windows-containers"></a>2. Переключение на контейнеры Windows

После установки в Docker для Windows по умолчанию будут запущены контейнеры Linux. Переключитесь на контейнеры Windows, используя меню области уведомлений Docker или выполнив следующую команду в командной строке PowerShell `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)
> Обратите внимание, что режим контейнеры Windows позволяет LCOW контейнеры в дополнение к контейнерам Windows.

## <a name="3-install-base-container-images"></a>3. Установка базовых образов контейнеров

Контейнеры Windows создаются из базовых образов. Приведенная ниже команда извлекает базовый образ Nano Server.

```
docker pull microsoft/nanoserver
```

После его получения выполните команду `docker images`, чтобы вывести список установленных образов. В данном случае будет отображен образ Nano Server.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Прочтите лицензионное соглашение для образов ОС контейнеров Windows на странице [Лицензионное соглашение](../images-eula.md).

## <a name="4-run-your-first-windows-container"></a>4. запуск первого контейнера Windows

В этом простом примере будет создан и развернут образ контейнера "Hello World". Для получения наилучших результатов выполните следующие команды в командной строке Windows или PowerShell с повышенными привилегиями.
> Интегрированная среда сценариев Windows PowerShell не работает в интерактивных сеансах с контейнерами. Даже если контейнер запускается, он зависает.

Сначала запустите контейнер с помощью интерактивного сеанса из образа `nanoserver`. После запуска контейнера в его рамках появится командная оболочка.  

```
docker run -it microsoft/nanoserver cmd
```

Внутри контейнера создайте простой скрипт "Привет мир".

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

После завершения выйдите из контейнера.

```
exit
```

Теперь создайте новый образ контейнера из измененного контейнера. Чтобы просмотреть список контейнеров, выполните следующую команду и запишите идентификатор контейнера.

```
docker ps -a
```

Выполните следующую команду для создания нового образа "Привет мир". Замените <containerid> на идентификатор контейнера.

```
docker commit <containerid> helloworld
```

После завершения вы получите пользовательский образ, содержащий скрипт "Привет мир". Чтобы убедиться в этом, используйте следующую команду.

```
docker images
```

Наконец, чтобы запустить контейнер, используйте команду `docker run`.

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

В результате выполнения команды `docker run` создается контейнер Hyper-V из образа "Привет мир", затем запускается скрипт-пример "Привет мир" (выходные данные при этом выводятся в оболочке), после чего контейнер останавливается и удаляется.
В последующих кратких руководствах, посвященных Windows10 и контейнерам, будет подробно описано создание и развертывание приложений в контейнерах на базе Windows10.

## <a name="run-your-first-lcow-container"></a>Запуск первого контейнера LCOW

В этом примере будет развернута BusyBox контейнера. Во-первых пытаться запускать образа «Привет мир» BusyBox.

```
docker run --rm busybox echo hello_world
```

Обратите внимание, что этот метод возвращает ошибку при попытке извлечь образ Docker. Это происходит потому, что Dockers требуется директива через `--platform` флаг, чтобы убедиться, что операционная система изображение и узла сопоставляются соответствующим образом. Так как платформу по умолчанию в режиме Windows контейнера Windows, добавить `--platform linux` флаг для получения и запуска контейнера.

```
docker run --rm --platform linux busybox echo hello_world
```

После образе отозвано на платформе указано, `--platform` флаг не является обязательным. Выполните команду без него, чтобы проверить это.

```
docker run --rm busybox echo hello_world
```

Запустите `docker images` возвращает список установленных образов. В этом случае Windows и Linux изображения.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительно: См. в разделе Docker соответствующие [записи блога](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) на выполнение LCOW

Перейдите к следующему учебнику, чтобы ознакомиться с примером [создания приложения](./building-sample-app.md)
