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
ms.openlocfilehash: f9b54dbc9fc7c79bdb9b9aa106d5811401c365f3
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380468"
---
# <a name="linux-containers-on-windows-10"></a>Контейнеры Linux в Windows 10

> [!div class="op_single_selector"]
> - [Контейнеры Linux в Windows](quick-start-windows-10-linux.md)
> - [Контейнеры Windows в Windows](quick-start-windows-10.md)

Упражнение описывается создание и запуск контейнеров Linux в Windows 10.

В этом кратком руководстве будут выполнены:

1. Установленные Docker для Windows
2. Запустите простой контейнер Linux с помощью контейнеров Linux в Windows (LCOW)

Это краткое руководство применимо только к Windows 10. Краткое руководство по дополнительной документации можно найти в содержании в левой части этой страницы.

## <a name="prerequisites"></a>Что вам понадобится

Убедитесь, что соблюдаются следующие требования:
- Одна физическая компьютерная система под управлением Windows 10 Профессиональная или Корпоративная начиная с Fall Creators Update (версия 1709) или более поздней версии
- Убедитесь, что включен [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) .

***Изоляции Hyper-V:*** Контейнеры Linux в Windows требуется изоляции Hyper-V в Windows 10, чтобы разработчики с соответствующего ядро Linux для запуска контейнера. Более сведения о Hyper-V изоляции можно найти на странице " [контейнеры о Windows](../about/index.md) ".

## <a name="install-docker-for-windows"></a>Установка Docker для Windows

Скачайте [Docker для Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите программу установки (вам будет необходимо войти в систему. Создайте учетную запись, если у вас нет уже). [Подробные инструкции по установке](https://docs.docker.com/docker-for-windows/install) доступны в документации по Docker.

> Если у вас уже есть Docker установки, убедитесь, что у вас есть версии 18.02 или более поздней версии для поддержки LCOW. Проверьте, выполнив `docker -v` или проверки *О Docker*.

> Для запуска контейнеров LCOW необходимо активировать параметр «экспериментальных функций» в *Docker параметры > управляющей программы* .

## <a name="run-your-first-lcow-container"></a>Запуск первого контейнера LCOW

В этом примере будет развернута BusyBox контейнера. Во-первых попытаться выполнить образ BusyBox «Привет мир».

```console
docker run --rm busybox echo hello_world
```

Обратите внимание, что этот метод возвращает ошибку при попытке извлечь образ Docker. Это происходит, поскольку Dockers требует директиву через `--platform` флаг, чтобы убедиться, что изображения и узла операционной системе сопоставляются соответствующим образом. Так как платформу по умолчанию в режиме Windows контейнера Windows, добавьте `--platform linux` флаг для получения и запуска контейнера.

```console
docker run --rm --platform linux busybox echo hello_world
```

После полученного изображения на платформе указано, `--platform` флаг не является обязательным. Выполните команду без него, чтобы проверить это.

```console
docker run --rm busybox echo hello_world
```

Запустите `docker images` возвращает список установленных образов. В этом случае Windows и Linux изображения.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Дополнительно: См. в разделе Docker соответствующие [записи блога](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) о выполнении LCOW.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Узнайте, как создать пример приложения](./building-sample-app.md)