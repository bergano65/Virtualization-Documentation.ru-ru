---
title: Краткое руководство по развертыванию контейнеров— образы
description: Краткое руководство по развертыванию контейнеров
keywords: docker, контейнеры
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 7ab445212a400487bff182a2e73107b47ab00b1a
ms.sourcegitcommit: 5e5644bff6dba70e384db6c80787b3bbe7adb93c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/03/2018
ms.locfileid: "4303880"
---
# <a name="automating-builds-and-saving-images"></a>Автоматизация сборки и сохранения образов

В предыдущем кратком руководстве по Windows Server мы создали контейнер Windows на основе предварительно созданного примера .Net Core. В этом упражнении подробно рассматриваются автоматическое создание образа контейнера изображения с помощью Dockerfile и сохранение образов контейнеров в общедоступном реестре Docker Hub.

Это краткое руководство относится к контейнерам Windows Server в Windows Server2016. В нем будет использоваться базовый образ контейнера Windows Server Core. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

- Одна компьютерная система (физическая или виртуальная), работающая под управлением Windows Server2016.
- Настройте на компьютере компонент контейнеров Windows и Docker. Пошаговые инструкции по этим этапам см. в статье [Контейнеры Windows в Windows Server](./quick-start-windows-server.md).
- Идентификатор Docker, который будет использоваться для отправки образа контейнера в Docker Hub. Если у вас нет идентификатора Docker, зарегистрируйтесь для его получения в [Docker Cloud](https://cloud.docker.com/).

## <a name="1-container-image---dockerfile"></a>1. Образ контейнера— Dockerfile

Хотя контейнер можно вручную создать, изменить и поместить в новый образ контейнера, Docker предоставляет способ автоматизации этого процесса с помощью Dockerfile. Для этого упражнения требуется идентификатор Docker. Если у вас нет идентификатора Docker, зарегистрируйтесь для его получения в [Docker Cloud]( https://cloud.docker.com/).

На узле контейнера создайте каталог `c:\build`, а в нем— файл с именем `Dockerfile`. Обратите внимание, что этот файл не должен иметь расширение.

```
powershell new-item c:\build\Dockerfile -Force
```

Откройте файл Dockerfile в блокноте.

```
notepad c:\build\Dockerfile
```

Скопируйте в него приведенный ниже текст и сохраните файл. Эти команды дают Docker указание создать новый образ, приняв `microsoft/iis` в качестве основы. После этого файл Dockerfile выполняет команды, заданные в инструкции `RUN`. В этом случае обновляется содержимое файла index.html. 

Дополнительные сведения о файлах Dockerfile см. в статье [Файлы Dockerfile в Windows](../manage-docker/manage-windows-dockerfile.md).

```
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

Команда `docker build` запускает процесс создания образа. Параметр `-t` дает процессу указание присвоить новому образу имя `iis-dockerfile`. **Замените "user" на имя пользователя учетной записи Docker**. Если у вас нет учетной записи Docker, зарегистрируйтесь для ее получения в [Docker Cloud](https://cloud.docker.com/).

```
docker build -t <user>/iis-dockerfile c:\Build
```

После завершения можно убедиться, что образ создан, используя команду `docker images`.

```
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Теперь разверните контейнер с помощью следующей команды, снова заменив имя пользователя на свой идентификатор Docker.

```
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

После создания контейнера введите IP-адрес узла контейнера в браузере. Должно открыться приложение "Hello, World!".

![](media/dockerfile2.png)

Вернитесь на узел контейнера и выполните команду `docker ps`, чтобы получить имя контейнера, и команду `docker rm`, чтобы удалить контейнер. Примечание. Замените имя контейнера в данном примере на используемое вами.

Получите имя контейнера.

```
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```
Остановка контейнера.

```
docker stop <container name>
```

Удалите контейнер.

```
docker rm -f <container name>
```

## <a name="2-docker-push"></a>2. Операция отправки в Docker

Образы контейнеров Docker можно хранить в реестре контейнеров. Если образ хранится в реестре, его можно извлечь для последующего использования на нескольких различных узлах контейнеров. Docker предоставляет открытый реестр для хранения образов контейнеров в [Docker Hub](https://hub.docker.com/).

В этом упражнении мы отправим пользовательский образ "Hello world" в вашу учетную запись на Docker Hub.

Войдите в учетную запись Docker с помощью `docker login command`.

```
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

После входа в систему можно отправить образ контейнера в Docker Hub. Для этого используйте команду `docker push`. **Замените "user" на идентификатор Docker**. 

```
docker push <user>/iis-dockerfile
```

Теперь можно скачать образ контейнера из Docker Hub на любой узел контейнера Windows с помощью `docker pull`. В этом учебнике мы удалим существующий образ и затем извлечем его из Docker Hub. 

```
docker rmi <user>/iis-dockerfile
```

Команда `docker images` подтвердит, что образ был удален.

```
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

[Контейнеры Windows в Windows10](./quick-start-windows-10.md)
