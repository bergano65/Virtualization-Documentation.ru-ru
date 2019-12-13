---
title: Контейнеры Windows и Linux в Windows 10
description: Краткое руководство по развертыванию контейнеров
keywords: DOCKER, контейнеры, ЛКОВ
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909584"
---
# <a name="linux-containers-on-windows-10"></a>Контейнеры Linux в Windows 10

В этом упражнении будет рассмотрено создание и запуск контейнеров Linux в Windows 10.

В этом кратком руководстве вы сможете выполнить следующие действия.

1. Установка DOCKER Desktop
2. Запуск простого контейнера Linux с помощью контейнеров Linux в Windows (ЛКОВ)

Это краткое руководство применимо только к Windows 10. Дополнительную документацию по быстрому запуску можно найти в оглавлении в левой части этой страницы.

## <a name="prerequisites"></a>Предварительные условия

Убедитесь в соблюдении следующих требований.
- Одна физическая компьютерная система под управлением Windows 10 профессиональная, Windows 10 Корпоративная или Windows Server 2019 версии 1809 или более поздней.
- Убедитесь, что [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) включен.

***Изоляция Hyper-V:*** Для использования контейнеров Linux в Windows требуется изоляция Hyper-V в Windows 10, чтобы предоставить разработчикам соответствующий ядро Linux для запуска контейнера. Дополнительные сведения о изоляции Hyper-V можно найти на странице [о контейнерах Windows](../about/index.md) .

## <a name="install-docker-desktop"></a>Установка DOCKER Desktop

Скачайте [DOCKER Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите установщик (вам потребуется войти в систему. Создайте учетную запись, если она еще не создана. [Подробные инструкции по установке](https://docs.docker.com/docker-for-windows/install) доступны в документации по Docker.

> Если вы уже установили DOCKER, убедитесь, что у вас установлена версия 18,02 или более поздняя для поддержки ЛКОВ. Проверьте, запустив `docker -v` или проверив наличие *DOCKER*.

> Параметр "экспериментальные возможности" в *параметрах docker > DAEMON* должен быть активирован для запуска контейнеров лков.

## <a name="run-your-first-lcow-container"></a>Запуск первого контейнера ЛКОВ

В этом примере будет развернут контейнер Бусибокс. Сначала попытайтесь запустить образ Бусибокс "Hello World".

```console
docker run --rm busybox echo hello_world
```

Обратите внимание, что это возвращает ошибку, когда DOCKER пытается извлечь изображение. Это происходит потому, что DOCKER требует директиву с помощью флага `--platform`, чтобы убедиться, что образ и операционная система узла соответствуют соответствующим образом. Так как платформа по умолчанию в режиме контейнера Windows — Windows, добавьте флаг `--platform linux` для извлечения и запуска контейнера.

```console
docker run --rm --platform linux busybox echo hello_world
```

После того как образ был извлечен с указанной платформой, флаг `--platform` больше не требуется. Выполните команду без нее, чтобы проверить это.

```console
docker run --rm busybox echo hello_world
```

Запустите `docker images`, чтобы получить список установленных образов. В данном случае это образы Windows и Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Премия. см. соответствующую [запись блога](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) DOCKER, посвященную запуску лков.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Узнайте, как создать пример приложения](./building-sample-app.md)
