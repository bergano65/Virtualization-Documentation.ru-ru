---
title: "Контейнеры Windows в Windows Server"
description: "Краткое руководство по развертыванию контейнеров"
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 39e480b8bf3f90cfe9b7d4d17141b9dbec5f93e5
ms.openlocfilehash: 6fae5625731c936903c808e286dbb9b4abcddd06

---

# Контейнеры Windows в Windows Server

**Это предварительное содержимое. Возможны изменения.**

Это упражнение посвящено основным аспектам развертывания и использования функции контейнеров Windows в Windows Server. В его рамках вы установите роль контейнера и развернете простой контейнер Windows Server. Перед началом работы с этим кратким руководством ознакомьтесь с основными понятиями и терминологией для контейнеров. Эти сведения можно найти в статье [Знакомство с кратким руководством](./quick_start.md).

В этом кратком руководстве рассматриваются контейнеры Windows Server в Windows Server 2016. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

Одна компьютерная система (физическая или виртуальная), работающая под управлением [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview).

Полностью настроенный образ Windows Server доступен в Azure. Чтобы использовать этот образ, разверните виртуальную машину, нажав кнопку ниже. Развертывание займет около 10 минут. После его завершения войдите в виртуальную машину Azure и перейдите к шагу четыре этого руководства. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

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

Скачайте подсистему Docker и клиент в виде ZIP-архива.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Разархивируйте ZIP-архив в Program Files.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Добавьте каталог Docker в системный путь.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
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

Контейнеры Windows развертываются из шаблонов или образов. Перед развертыванием контейнера требуется загрузить базовый образ ОС. Следующая команда скачает базовый образ Windows Server Core.

```none
docker pull microsoft/windowsservercore
```

Этот процесс может занять некоторое время, поэтому сделайте перерыв и возвращайтесь к работе после скачивания.

На этом этапе выполнение команды `docker images` возвращает список установленных образов. В данном случае будет отображен образ Windows Server Core.

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

Подробную информацию об образах контейнеров Windows см. в статье [Управление образами контейнеров](../management/manage_images.md).

## 4. Развертывание первого контейнера

В этом упражнении вы скачиваете готовый образ IIS из реестра Docker Hub и развертываете простой контейнер с запущенными службами IIS.  

Для поиска образов контейнеров Windows в Docker Hub выполните команду `docker search Microsoft`.  

```none
docker search microsoft

NAME                                         DESCRIPTION
microsoft/aspnet                             ASP.NET is an open source server-side Web ...
microsoft/dotnet                             Official images for working with .NET Core...
mono                                         Mono is an open source implementation of M...
microsoft/azure-cli                          Docker image for Microsoft Azure Command L...
microsoft/iis                                Internet Information Services (IIS) instal...
microsoft/mssql-server-2014-express-windows  Microsoft SQL Server 2014 Express installe...
microsoft/nanoserver                         Nano Server base OS image for Windows cont...
microsoft/windowsservercore                  Windows Server Core base OS image for Wind...
microsoft/oms                                Monitor your containers using the Operatio...
microsoft/dotnet-preview                     Preview bits for microsoft/dotnet image
microsoft/dotnet35
microsoft/applicationinsights                Application Insights for Docker helps you ...
microsoft/sample-redis                       Redis installed in Windows Server Core and...
microsoft/sample-node                        Node installed in a Nano Server based cont...
microsoft/sample-nginx                       Nginx installed in Windows Server Core and...
microsoft/sample-httpd                       Apache httpd installed in Windows Server C...
microsoft/sample-dotnet                      .NET Core running in a Nano Server container
microsoft/sqlite                             SQLite installed in a Windows Server Core ...
...
```

Скачайте образ IIS с помощью `docker pull`.  

```none
docker pull microsoft/iis
```

Скачивание образа можно проверить с помощью команды `docker images`. Обратите внимание, что отображается как базовый образ (windowsservercore), так и образ IIS.

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis                 latest              accd044753c1        11 days ago         7.907 GB
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

Используйте команду `docker run` для развертывания контейнера IIS.

```none
docker run -d -p 80:80 microsoft/iis ping -t localhost
```

Здесь образ IIS выполняется как фоновая служба (-d) и настраивает сетевое взаимодействие таким образом, что порт 80 узла контейнера сопоставляется с портом 80 контейнера.
Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com]( https://docs.docker.com/engine/reference/run/).


Запущенные контейнеры можно просмотреть с помощью команды `docker ps`. Запишите имя контейнера, так как оно понадобится позднее.

```none
docker ps

CONTAINER ID  IMAGE          COMMAND              CREATED             STATUS             PORTS               NAME
09c9cc6e4f83  microsoft/iis  "ping -t localhost"  About a minute ago  Up About a minute  0.0.0.0:80->80/tcp  big_jang
```

На другом компьютере откройте браузер и введите IP-адрес узла контейнера. Если все настроено правильно, отображается экран-заставка IIS. Он предоставляется из экземпляра IIS, размещенного в контейнере Windows.

**Примечание.** Если вы работаете в Azure, потребуется внешний IP-адрес виртуальной машины и настроенная безопасность сети. Дополнительные сведения см. в разделе [Create Rule in a Network Security Group]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg) (Создание правила в группе безопасности сети).

![](media/iis1.png)

Вернитесь на узел контейнера и выполните команду `docker rm`, чтобы удалить контейнер. Примечание. Замените имя контейнера в данном примере на используемое вами.

```none
docker rm -f big_jang
```
## Дальнейшие действия

[Образы контейнеров в Windows Server](./quick_start_images.md)

[Контейнеры Windows в Windows 10](./quick_start_windows_10.md)



<!--HONumber=Aug16_HO4-->


