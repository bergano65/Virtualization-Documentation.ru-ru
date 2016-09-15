---
title: "Образы контейнеров Windows"
description: "Создание образов контейнеров и управление ими с помощью контейнеров Windows."
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
redirect_url: https://docs.docker.com/v1.8/userguide/dockerimages/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 49e29949fc91533cf1d3ed0aef47e829276772f4

---

# Образы контейнеров Windows

**Это предварительное содержимое. Возможны изменения.** 

>Управление контейнерами Windows осуществляется с помощью Docker. Документация по контейнерам Windows дополняет документацию, размещенную на сайте [docs.docker.com](https://docs.docker.com/).

Образы контейнеров используются для развертывания контейнеров. Эти образы могут содержать приложения и все зависимости приложений. Например, можно создать образ контейнера, который предварительно настроен с использованием Nano Server, IIS и приложения, работающего в IIS. Этот образ контейнера можно хранить в реестре контейнеров для создания нового образа контейнера на его основе или для дальнейшего использования. При этом выполняется его развертывание на любом узле контейнера Windows (локально, в облаке или даже в службе контейнеров).

### Установка образа

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве базовой операционной системы. Сведения о поддерживаемых конфигурациях см. в статье [Требования к контейнеру Windows](../deployment/system_requirements.md).

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```none
docker pull microsoft/windowsservercore
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```none
docker pull microsoft/nanoserver
```

### Образы списков

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
```

### Создание образа

Образ контейнера можно создать из любого существующего контейнера. Для этого используйте команду `docker commit`. В следующем примере создается образ контейнера с именем windowsservercoreiis.

```none
docker commit 475059caef8f windowsservercoreiis
```

### Удаление образа

Образы контейнеров невозможно удалить, если какой-либо контейнер, даже в режиме остановки, зависит от образа.

При удалении образа с помощью Docker можно указать его идентификатор или имя.

```none
docker rmi windowsservercoreiis
```

### Зависимость образа

Чтобы просмотреть зависимости образов с помощью Docker, можно использовать команду `docker history`.

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Реестр Docker Hub содержит предварительно созданные образы, которые можно скачать на узел контейнера. Скачав эти образы, вы сможете применить их как основу для приложений контейнера Windows.

Чтобы просмотреть список образов, доступных в Docker Hub, выполните команду `docker search`. Примечание. Прежде чем извлекать зависимые образы контейнеров из Docker Hub, необходимо установить базовые образы Windows Server Core или Nano Server.

Большинство этих образов имеют версию Windows Server Core и Nano Server. Чтобы получить конкретную версию, просто добавьте тег ":windowsservercore" или ":nanoserver". Тег "latest" по умолчанию возвращает версию Windows Server Core, кроме случаев, когда доступна только версия Nano Server.


```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

### Операция запроса в Docker

Чтобы скачать образ из Docker Hub, используйте команду `docker pull`. Дополнительные сведения см. в статье [Docker Pull](https://docs.docker.com/engine/reference/commandline/pull/) (Операция запроса в Docker) на сайте Docker.com.

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

Теперь этот образ отобразится при выполнении команды `docker images`.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

> При сбое операции запроса в Docker убедитесь, что к узлу контейнера были применены последние накопительные пакеты обновления. Обновление TP5 можно найти в [статье базы знаний 3157663]( https://support.microsoft.com/en-us/kb/3157663).

### Операция отправки в Docker

Образы контейнеров можно также отправить в Центр Docker или в доверенный реестр Docker. После отправки эти образы можно скачать и многократно использовать в разных средах контейнеров Windows.

Чтобы отправить образ контейнера в Центр Docker, сначала войдите в реестр. Дополнительные сведения см. в статье [Docker Login]( https://docs.docker.com/engine/reference/commandline/login/) (Вход в Docker) на сайте Docker.com.

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

После входа в Центр Docker или свой доверенный реестр Docker используйте команду `docker push` для отправки образа контейнера. На образ контейнера можно сослаться по имени или идентификатору. Дополнительные сведения см. в статье [Docker Push]( https://docs.docker.com/engine/reference/commandline/push/) (Операция отправки в Docker) на сайте Docker.com.

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

После этого образ контейнера станет доступен через команду `docker pull`.






<!--HONumber=Sep16_HO2-->


