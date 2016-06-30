---
title: "Образы контейнеров Windows"
description: "Создание образов контейнеров и управление ими с помощью контейнеров Windows."
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
translationtype: Human Translation
ms.sourcegitcommit: 3db43b433e7b1a9484d530cf209ea80ef269a307
ms.openlocfilehash: 505cc64fa19fb9fc8c2d5c109830f460f09332dd

---

# Образы контейнеров Windows

**Это предварительное содержимое. Возможны изменения.** 

Образы контейнеров используются для развертывания контейнеров. Эти образы могут содержать операционную систему, приложения и все зависимости приложений. Например, можно создать образ контейнера, который предварительно настроен с использованием Nano Server, IIS и приложения, работающего в IIS. Этот образ контейнера можно хранить в реестре контейнеров для создания нового образа контейнера на его основе или для дальнейшего использования. При этом выполняется его развертывание на любом узле контейнера Windows (локально, в облаке или даже в службе контейнеров).

Существует два типа образов контейнеров.

**Базовые образы ОС** предоставляются корпорацией Майкрософт и включают в себя основные компоненты ОС. 

**Образы контейнеров** — пользовательские образы, созданные на основе базовых образов ОС.

## Базовые образы ОС

### Установка образа

Образы ОС контейнеров можно найти и установить с помощью модуля ContainerImage PowerShell. Чтобы использовать этот модуль, его необходимо установить. Для установки модуля можно использовать приведенную ниже команду. Дополнительные сведения об использовании модуля OneGet PowerShell для образов контейнеров см. в статье [Поставщик образов контейнеров](https://github.com/PowerShell/ContainerProvider). 

```none
Install-PackageProvider ContainerImage -Force
```

После установки можно вернуть список базовых образов ОС с помощью команды `Find-ContainerImage`.

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

Чтобы скачать и установить базовый образ Nano Server, выполните приведенную ниже команду. Параметр `-version` не является обязательным. Если не указана базовая версия образа ОС, будет установлена последняя версия.

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

Эта команда также позволяет скачать и установить базовый образ ОС Windows Server Core. Параметр `-version` не является обязательным. Если не указана базовая версия образа ОС, будет установлена последняя версия.

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

С помощью команды `docker images` убедитесь, что образы были установлены. 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

После установки может потребоваться пометить образы тегом "latest". Соответствующие инструкции приведены в разделе о тегах ниже.

> Если базовый образ ОС скачивается, но не отображается при выполнении `docker images`, перезапустите службу Docker с помощью приложения панели управления службами или с помощью команд "sc stop docker" и "sc start docker".

### Пометка образов тегами

При ссылке на образ контейнера по имени подсистема Docker будет искать последнюю версию образа. Если последнюю версию невозможно определить, появится следующая ошибка.

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

После установки базовых образов ОС Windows Server Core или Nano Server их потребуется пометить тегами с версией "последняя". Для этого используйте команду `docker tag`. 

Дополнительные сведения о `docker tag` см. в статье о [тегах, принудительной отправке и извлечении образов на сайте docker.com](https://docs.docker.com/mac/step_six/). 

```none
docker tag <image id> windowsservercore:latest
```

При назначении тегов в выходных данных `docker images` отображаются две версии одного образа: с тегом версии образа и с тегом "latest" (последняя). Теперь на образ можно ссылаться по имени.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Автономная установка

Базовые образы ОС также можно установить без подключения к Интернету. Для этого скачайте образ на компьютер с подключением к Интернету, скопируйте его в целевую систему, а затем импортируйте с помощью команды `Install-ContainerOSImages`.

Перед скачиванием базового образа ОС подготовьте **подключенную к Интернету** систему с поставщиком образа контейнера, запустив следующую команду.

```none
Install-PackageProvider ContainerImage -Force
```

Чтобы вернуть список образов из диспетчера пакетов OneGet в PowerShell:

```none
Find-ContainerImage
```

Вывод:

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

Чтобы скачать образ, используйте команду `Save-ContainerImage`.

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

После этого скачанный образ контейнера можно скопировать на **автономный узел контейнера** и установить с помощью команды `Install-ContainerOSImage`.

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Удаление образа ОС

Базовые образы ОС можно удалить с помощью команды `Uninstall-ContainerOSImage`. В следующем примере будет удален базовый образ ОС NanoServer.

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## Образы контейнеров

### Образы списков

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
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






<!--HONumber=Jun16_HO4-->


