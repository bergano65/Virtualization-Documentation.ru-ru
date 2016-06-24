---
title: Контейнеры Windows в Windows Server
description: Краткое руководство по развертыванию контейнеров
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
---

# Контейнеры Windows в Windows Server

**Это предварительное содержимое. Возможны изменения.**

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows Server. В его рамках вы установите роль контейнера и развернете простой контейнер Windows Server. Перед началом работы с этим кратким руководством ознакомьтесь с основными понятиями и терминологией для контейнеров. Эти сведения можно найти в статье [Знакомство с кратким руководством](./quick_start.md).

В этом кратком руководстве рассматриваются контейнеры Windows Server в Windows Server 2016. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

- Одна компьютерная система (физическая или виртуальная), работающая под управлением [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview).

## 1. Установка компонента контейнеров

Чтобы начать работу с контейнерами Windows, требуется включить контейнер компонентов. Для этого выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```none
Install-WindowsFeature containers
```

После завершения установки компонента перезагрузите компьютер.

```none
Restart-Computer -Force
```

## 2. Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента.

Создайте папку для исполняемых файлов Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Скачайте управляющую программу Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Загрузите клиент Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Добавьте каталог Docker в системный путь. По завершении перезапустите сеанс PowerShell, чтобы был распознан измененный путь.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Чтобы установить Docker в качестве службы Windows, выполните следующую команду.

```none
dockerd --register-service
```

После установки эту службу можно запустить.

```none
Start-Service docker
```

## 3. Установка базовых образов контейнеров

Контейнеры Windows развертываются из шаблонов или образов. Перед развертыванием контейнера требуется загрузить базовый образ ОС. Приведенные ниже команды скачивают базовый образ Windows Server Core.

Сначала установите поставщик пакетов образов контейнеров.

```none
Install-PackageProvider ContainerImage -Force
```

Затем установите образ Windows Server Core. Этот процесс может занять некоторое время, поэтому сделайте перерыв и возвращайтесь к работе после скачивания.

```none
Install-ContainerImage -Name WindowsServerCore    
```

После установки базового образа следует перезапустить службу Docker.

```none
Restart-Service docker
```

На этом этапе выполнение `docker images` возвращает список установленных образов — в данном случае это образ Windows Server Core.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
windowsservercore   10.0.14300.1000     dbfee88ee9fd        7 weeks ago         9.344 GB
```

Перед продолжением этот образ следует пометить как последнюю версию "latest". Для этого выполните следующую команду:

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

Дополнительные сведения об образах контейнеров Windows см. в статье [Управление образами контейнеров](../management/manage_images.md).

## 4. Развертывание первого контейнера

В этом упражнении вы скачиваете готовый образ IIS из реестра Docker Hub и развертываете простой контейнер с запущенными службами IIS.  

Для поиска образов контейнеров Windows в Docker Hub выполните команду `docker search Microsoft`.  

```none
docker search microsoft

NAME                                         DESCRIPTION                                     
microsoft/sample-django:windowsservercore    Django installed in a Windows Server Core ...   
microsoft/dotnet35:windowsservercore         .NET 3.5 Runtime installed in a Windows Se...   
microsoft/sample-golang:windowsservercore    Go Programming Language installed in a Win...   
microsoft/sample-httpd:windowsservercore     Apache httpd installed in a Windows Server...   
microsoft/iis:windowsservercore              Internet Information Services (IIS) instal...   
microsoft/sample-mongodb:windowsservercore   MongoDB installed in a Windows Server Core...   
microsoft/sample-mysql:windowsservercore     MySQL installed in a Windows Server Core b...   
microsoft/sample-nginx:windowsservercore     Nginx installed in a Windows Server Core b...  
microsoft/sample-python:windowsservercore    Python installed in a Windows Server Core ...   
microsoft/sample-rails:windowsservercore     Ruby on Rails installed in a Windows Serve...  
microsoft/sample-redis:windowsservercore     Redis installed in a Windows Server Core b...   
microsoft/sample-ruby:windowsservercore      Ruby installed in a Windows Server Core ba...   
microsoft/sample-sqlite:windowsservercore    SQLite installed in a Windows Server Core ...  
```

Скачайте образ IIS с помощью `docker pull`.  

```none
docker pull microsoft/iis:windowsservercore
```

Скачивание образа можно проверить с помощью команды `docker images`. Обратите внимание, что отображается как базовый образ (windowsservercore), так и образ IIS.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Используйте команду `docker run` для развертывания контейнера IIS.

```none
docker run -d -p 80:80 microsoft/iis:windowsservercore ping -t localhost
```

Здесь образ IIS выполняется как фоновая служба (-d) и настраивает сетевое взаимодействие таким образом, что порт 80 узла контейнера сопоставляется с портом 80 контейнера.
Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com]( https://docs.docker.com/engine/reference/run/).


Запущенные контейнеры можно просмотреть с помощью команды `docker ps`. Запишите имя контейнера, так как оно понадобится позднее.

```none
docker ps

CONTAINER ID    IMAGE                             COMMAND               CREATED              STATUS   PORTS                NAMES
9cad3ea5b7bc    microsoft/iis:windowsservercore   "ping -t localhost"   About a minute ago   Up       0.0.0.0:80->80/tcp   grave_jang
```

На другом компьютере откройте браузер и введите IP-адрес узла контейнера. Если все настроено правильно, отображается экран-заставка IIS. Он предоставляется из экземпляра IIS, размещенного в контейнере Windows.

**Примечание.** Если вы работаете в Azure, потребуется правило группы безопасности сети, разрешающее передачу трафика через порт 80. Дополнительные сведения см. в разделе [Create Rule in a Network Security Group]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg) (Создание правила в группе безопасности сети).

![](media/iis1.png)

Вернитесь на узел контейнера и выполните команду `docker rm`, чтобы удалить контейнер. Примечание. Замените имя контейнера в данном примере на используемое вами.

```none
docker rm -f grave_jang
```
## Дальнейшие шаги

[Образы контейнеров в Windows Server](./quick_start_images.md)

[Контейнеры Windows в Windows 10](./quick_start_windows_10.md)


<!--HONumber=Jun16_HO3-->


