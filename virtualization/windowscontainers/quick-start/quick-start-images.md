---
title: Краткое руководство по развертыванию контейнеров— образы
description: Краткое руководство по развертыванию контейнеров
keywords: docker, контейнеры
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 0350e62deef06402991f505dd263db7fd506cba1
ms.sourcegitcommit: 1aef193cf56dd0870139b5b8f901a8d9808ebdcd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001590"
---
# <a name="automating-builds-and-saving-images"></a>Автоматизация сборки и сохранения образов

В предыдущем кратком руководстве по Windows Server мы создали контейнер Windows на основе предварительно созданного примера .Net Core. В этом упражнении подробно рассматриваются автоматическое создание образа контейнера изображения с помощью Dockerfile и сохранение образов контейнеров в общедоступном реестре Docker Hub.

Это краткое руководство относится к контейнерам Windows Server в Windows Server2016. В нем будет использоваться базовый образ контейнера Windows Server Core. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

## <a name="prerequisites"></a>Необходимые условия

Убедитесь, что соблюдаются следующие требования:

- Одна компьютерная система (физическая или виртуальная), работающая под управлением Windows Server2016.
- Настройте на компьютере компонент контейнеров Windows и Docker. Пошаговые инструкции по этим этапам см. в статье [Контейнеры Windows в Windows Server](./quick-start-windows-server.md).
- Идентификатор Docker, который будет использоваться для отправки образа контейнера в Docker Hub. Если у вас нет идентификатора Docker, зарегистрируйтесь для его получения в [Docker Cloud](https://cloud.docker.com/).

## <a name="container-image---dockerfile"></a>Образ контейнера — Dockerfile

Хотя контейнер можно вручную создать, изменить и поместить в новый образ контейнера, Docker предоставляет способ автоматизации этого процесса с помощью Dockerfile. Для этого упражнения требуется идентификатор Docker. Если у вас нет идентификатора Docker, зарегистрируйтесь для его получения в [Docker Cloud]( https://cloud.docker.com/).

На узле контейнера создайте каталог `c:\build`, а в нем— файл с именем `Dockerfile`. Обратите внимание, что этот файл не должен иметь расширение.

```console
powershell new-item c:\build\Dockerfile -Force
```

Откройте файл Dockerfile в блокноте.

```console
notepad c:\build\Dockerfile
```

Скопируйте в него приведенный ниже текст и сохраните файл. Эти команды дают Docker указание создать новый образ, приняв `microsoft/iis` в качестве основы. После этого файл Dockerfile выполняет команды, заданные в инструкции `RUN`. В этом случае обновляется содержимое файла index.html.

Дополнительные сведения о файлах Dockerfile см. в статье [Файлы Dockerfile в Windows](../manage-docker/manage-windows-dockerfile.md).

```dockerfile
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

Команда `docker build` запускает процесс создания образа. Параметр `-t` дает процессу указание присвоить новому образу имя `iis-dockerfile`. **Замените "user" на имя пользователя учетной записи Docker**. Если у вас нет учетной записи Docker, зарегистрируйтесь для ее получения в [Docker Cloud](https://cloud.docker.com/).

```console
docker build -t <user>/iis-dockerfile c:\Build
```

После завершения можно убедиться, что образ создан, используя команду `docker images`.

```console
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Теперь разверните контейнер с помощью следующей команды, снова заменив имя пользователя на свой идентификатор Docker.

```console
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

После создания контейнера введите IP-адрес узла контейнера в браузере. Должно открыться приложение "Hello, World!".

![](media/dockerfile2.png)

Вернитесь на узел контейнера и выполните команду `docker ps`, чтобы получить имя контейнера, и команду `docker rm`, чтобы удалить контейнер. Примечание. Замените имя контейнера в данном примере на используемое вами.

Получите имя контейнера.

```console
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Остановка контейнера.

```console
docker stop <container name>
```

Удалите контейнер.

```console
docker rm -f <container name>
```

## <a name="docker-push"></a>Операция отправки в docker

Образы контейнеров Docker можно хранить в реестре контейнеров. Если образ хранится в реестре, его можно извлечь для последующего использования на нескольких различных узлах контейнеров. Docker предоставляет открытый реестр для хранения образов контейнеров в [Docker Hub](https://hub.docker.com/).

В этом упражнении мы отправим пользовательский образ "Hello world" в вашу учетную запись на Docker Hub.

Войдите в учетную запись Docker с помощью `docker login command`.

```console
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

После входа в систему можно отправить образ контейнера в Docker Hub. Для этого используйте команду `docker push`. **Замените "user" на идентификатор Docker**. 

```console
docker push <user>/iis-dockerfile
```

Как Docker помещает слоями до Docker Hub, docker пропустит слоев, которые уже существуют в центре Docker или в других реестрами (внешнего слои).  Например последних версиях Windows Server Core, размещенных в реестре Microsoft или слои из частного реестра организации, пропущен и не помещается в Docker Hub.

Теперь можно скачать образ контейнера из Docker Hub на любой узел контейнера Windows с помощью `docker pull`. В этом учебнике мы удалим существующий образ и затем извлечем его из Docker Hub. 

```console
docker rmi <user>/iis-dockerfile
```

Команда `docker images` подтвердит, что образ был удален.

```console
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

Наконец, с помощью команды "docker pull" можно поместить образ обратно в узел контейнера. Замените "user" на имя пользователя учетной записи Docker. 

```
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать, как создать пакет примера приложения ASP.NET, посетите ссылки на учебники для Windows 10, показанные ниже.

> [!div class="nextstepaction"]
> [Контейнеры в Windows 10](./quick-start-windows-10.md)
