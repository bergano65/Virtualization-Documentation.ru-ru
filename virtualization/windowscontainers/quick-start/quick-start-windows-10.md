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
ms.openlocfilehash: dc500a7b6c0f8f078820407e6ed80ca5868bf4f3
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973654"
---
# <a name="windows-containers-on-windows-10"></a>Контейнеры Windows в Windows10

> [!div class="op_single_selector"]
> - [Контейнеры Linux в Windows](quick-start-windows-10-linux.md)
> - [Контейнеры Windows в Windows](quick-start-windows-10.md)

Упражнение описывается создание и запуск контейнеров Windows в Windows 10.

В этом кратком руководстве будут выполнены:

1. Установка Docker для Windows
2. Так как простой контейнер Windows

Это краткое руководство применимо только к Windows 10. Краткое руководство по дополнительной документации можно найти в содержании в левой части этой страницы.

## <a name="prerequisites"></a>Необходимые условия
Убедитесь, что соблюдаются следующие требования:
- Одна физическая компьютерная система под управлением Windows 10 Профессиональная или Корпоративная Юбилейное обновление (версия 1607) или более поздней версии. 
- Убедитесь, что включен [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) .

***Изоляции Hyper-V:*** Контейнеры Windows Server требуется изоляции Hyper-V в Windows 10, чтобы разработчики с тем же версию ядра и конфигурации, который будет использоваться в рабочей среде, более сведения о Hyper-V изоляции можно найти на странице " [сведения о контейнерах Windows](../about/index.md) ".

> [!NOTE]
> В выпуске Windows октября обновления 2018 г. мы больше не запрещает пользователям запускать контейнер Windows в режиме изоляции процессов в Windows 10 Корпоративная или Профессиональная для разработки и тестирования. См. [: вопросы и ответы](../about/faq.md) для получения дополнительных сведений.

## <a name="install-docker-for-windows"></a>Установка Docker для Windows

Скачайте [Docker для Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите программу установки (вам будет необходимо войти в систему. Создайте учетную запись, если у вас нет уже). [Подробные инструкции по установке](https://docs.docker.com/docker-for-windows/install) доступны в документации по Docker.

## <a name="switch-to-windows-containers"></a>Переключение на контейнеры Windows

После установки в Docker для Windows по умолчанию будут запущены контейнеры Linux. Переключитесь на контейнеры Windows с помощью меню задач Docker или выполнив следующую команду в PowerShell командной строке:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>Установка базовых образов контейнеров

Контейнеры Windows создаются из базовых образов. Приведенная ниже команда извлекает базовый образ Nano Server.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

После его получения выполните команду `docker images`, чтобы вывести список установленных образов. В данном случае будет отображен образ Nano Server.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Внимательно прочитайте [Лицензионное соглашение](../images-eula.md)образа ОС контейнеров Windows.

## <a name="run-your-first-windows-container"></a>Запуск первого контейнера Windows

В этом простом примере будет создана и развернут образ контейнера «Привет мир». Для получения наилучших результатов выполните следующие команды в командной строке Windows или PowerShell с повышенными привилегиями.

> Интегрированная среда сценариев Windows PowerShell не работает в интерактивных сеансах с контейнерами. Даже если контейнер запускается, он зависает.

Сначала запустите контейнер с помощью интерактивного сеанса из образа `nanoserver`. После запуска контейнера в его рамках появится командная оболочка.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Внутри контейнера мы создадим простое текстовый файл «Привет мир».

```cmd
echo "Hello World!" > Hello.txt
```   

После завершения выйдите из контейнера.

```cmd
exit
```

Теперь создайте новый образ контейнера из измененного контейнера. Чтобы просмотреть список контейнеров, выполните следующую команду и запишите идентификатор контейнера.

```console
docker ps -a
```

Выполните следующую команду для создания нового образа "Привет мир". Замените <containerid> на идентификатор контейнера.

```console
docker commit <containerid> helloworld
```

После завершения вы получите пользовательский образ, содержащий скрипт "Привет мир". Чтобы убедиться в этом, используйте следующую команду.

```console
docker images
```

Наконец, чтобы запустить контейнер, используйте команду `docker run`.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

Результат `docker run` команда — создается контейнер Hyper-V из образа «Привет мир», экземпляр cmd запущен в контейнере и выполнить чтение из нашего файла (выходные данные при этом выводятся в оболочке), после чего контейнер останавливается и удаляется.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Узнайте, как создать пример приложения](./building-sample-app.md)