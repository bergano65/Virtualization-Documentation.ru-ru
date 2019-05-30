---
title: Контейнеры для Windows и Linux в Windows 10
description: Краткое руководство по развертыванию контейнеров
keywords: Dock, Containers, ЛКОВ
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 91031f9394cb3fcb1af6c4813f8805ad6f79bf8c
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681104"
---
# <a name="linux-containers-on-windows-10"></a>Контейнеры Linux в Windows 10

> [!div class="op_single_selector"]
> - [Контейнеры Linux в Windows](quick-start-windows-10-linux.md)
> - [Контейнеры Windows в Windows](quick-start-windows-10.md)

С помощью этого упражнения вы сможете создавать и запускать контейнеры Linux в Windows 10.

Это краткое руководство поможет вам сделать следующее:

1. Установка стыковочного компьютера
2. Выполнение простого контейнера Linux с помощью контейнеров Linux в Windows (ЛКОВ)

Это краткое руководство применимо только к Windows 10. Дополнительные краткие руководства по началу работы можно найти в оглавлении в левой части страницы.

## <a name="prerequisites"></a>Что вам понадобится

Убедитесь, что вы отвечаете на следующие требования: _Лт__лт__лт__лт__лт__лт__лт_ HEAD
- Одна физическая компьютерная система под управлением Windows 10 профессиональная или Корпоративная с обновлениями для дизайнеров (версия 1709) или более поздней.
- Убедитесь, что включена [технология Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) .
=======
- Одна физическая компьютерная система под управлением Windows 10 профессиональная, Windows 10 корпоративный или Windows Server 2019 версии 1809 или более поздней.
- Убедитесь, что включена [технология Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) .
>>>>>>> Источник и образец

***Изоляция Hyper-V:*** Контейнеры Linux в Windows требуют изоляции Hyper-V в Windows 10 для предоставления разработчикам соответствующего ядра Linux для выполнения контейнера. Дополнительные сведения об изоляции Hyper-V можно найти на странице [о контейнерах Windows](../about/index.md) .

## <a name="install-docker-desktop"></a>Установка стыковочного компьютера

Скачайте закрепление для настольного [компьютера](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите установщик (вам потребуется выполнить вход. Создайте учетную запись, если она еще не установлена. [Подробные инструкции по установке](https://docs.docker.com/docker-for-windows/install) доступны в документации по Docker.

> Если вы уже установили Dock, убедитесь в том, что у вас установлена версия 18,02 или более поздней версии для поддержки ЛКОВ. Проверьте, запустив `docker -v` или проверяя закрепление. **

> Для запуска ЛКОВ контейнеров необходимо активировать параметр "экспериментальные функции" в *_Гт_ демоне параметров Dock* .

## <a name="run-your-first-lcow-container"></a>Выполнение первого контейнера ЛКОВ

В этом примере будет развернут контейнер Бусибокс. Во-первых, попробуйте выполнить Бусибокс изображение "Hello World".

```console
docker run --rm busybox echo hello_world
```

Обратите внимание, что в этом случае возвращается ошибка, когда закрепление пытается извлечь изображение. Это случается из-за того, что Docks `--platform` требуется директива с помощью флага, чтобы убедиться в правильности соответствия образа и операционной системы хоста. Так как платформа по умолчанию в режиме контейнера Windows — Windows, `--platform linux` добавьте пометку для затягивания и запуска контейнера.

```console
docker run --rm --platform linux busybox echo hello_world
```

После того как изображение будет извлечено с указанной платформой, `--platform` этот флаг больше не нужен. Для проверки выполните команду без нее.

```console
docker run --rm busybox echo hello_world
```

Запустите `docker images` , чтобы вернуть список установленных изображений. В этом случае образы Windows и Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Премия: Просмотрите соответствующую [запись в блоге на веб](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) -сервере в лков.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о том, как создать пример приложения](./building-sample-app.md)
