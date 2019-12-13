---
title: Контейнеризовать приложения .NET Core
description: Узнайте, как создать пример приложения .NET Core с контейнерами
keywords: docker, контейнеры
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910144"
---
# <a name="containerize-a-net-core-app"></a>Контейнеризовать приложения .NET Core

В этом разделе описывается упаковка существующего примера приложения .NET для развертывания в качестве контейнера Windows после настройки среды, как описано в разделе [Начало работы: подготовка окон для контейнеров](set-up-environment.md)и запуск первого контейнера, как описано в статье [Запуск первого контейнера Windows](run-your-first-container.md).

Кроме того, на компьютере должна быть установлена система управления исходным кодом Git. Чтобы установить его, посетите [Git](https://git-scm.com/download).

## <a name="clone-the-sample-code-from-github"></a>Клонирование примера кода из GitHub

Весь исходный код примера контейнера находится в репозитории Git [— документации](https://github.com/MicrosoftDocs/Virtualization-Documentation) (известный как репозиторий) в папке с именем `windows-container-samples`.

1. Откройте сеанс PowerShell и перейдите в папку, в которой вы хотите сохранить этот репозиторий. (Другие типы окон командной строки работают также, но в примерах команд используется PowerShell.)
2. Клонировать репозиторий в текущий рабочий каталог:

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Перейдите к каталогу примеров, найденному в разделе `Virtualization-Documentation\windows-container-samples\asp-net-getting-started`, и создайте Dockerfile с помощью следующих команд.

   [Dockerfile](https://docs.docker.com/engine/reference/builder/) похож на файл Makefile — это список инструкций, указывающих обработчику контейнеров, как создать образ контейнера.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Напишите Dockerfile

Откройте только что созданный Dockerfile в любом текстовом редакторе, а затем добавьте следующее содержимое:

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Давайте разделите его на строки и объясните, что делает каждая инструкция.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

В первой группе строк объявляется базовый образ, который будет использоваться для создания контейнера. Если в локальной системе еще нет этого образа, Docker автоматически попробует извлечь его. `mcr.microsoft.com/dotnet/core/sdk:2.1` поставляется вместе с установленным пакетом SDK для .NET Core 2,1, поэтому необходимо выполнить задачу создания проектов ASP .NET Core, предназначенных для версии 2,1. Следующая инструкция изменяет рабочий каталог в контейнере на `/app`, поэтому все команды, следующие за этим, выполняются в этом контексте.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Затем эти инструкции копируют файлы CSPROJ в каталог `/app` контейнера `build-env`. После копирования этого файла .NET прочитает его, а затем выберет все зависимости и средства, необходимые для нашего проекта.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

После того как .NET извлекают все зависимости в контейнер `build-env`, следующая инструкция копирует все исходные файлы проекта в контейнер. Затем мы указываем, что .NET публикует наше приложение с конфигурацией выпуска и указывает выходной путь в.

Компиляция должна быть выполнена. Теперь необходимо создать окончательный образ. 

> [!TIP]
> В этом кратком руководстве выполняется сборка проекта .NET Core из источника. При создании образов контейнеров рекомендуется включать в образ контейнера _только_ полезные данные рабочей нагрузки и их зависимости. Мы не хотим, чтобы пакет SDK для .NET Core включался в окончательный образ, так как нам нужна только среда выполнения .NET Core, поэтому dockerfile предназначен для использования временного контейнера, упакованного с помощью пакета SDK, который называется `build-env` для создания приложения.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Так как наше приложение — ASP.NET, мы указываем образ с включенной средой выполнения. Затем мы скопируем все файлы из выходного каталога временного контейнера в финальный контейнер. Мы настроим контейнер для работы с нашим новым приложением в качестве точки входа при запуске контейнера.

Мы написали dockerfile для выполнения _многоэтапной сборки_. При выполнении dockerfile будет использоваться временный контейнер, `build-env`с пакетом SDK для .NET Core 2,1, чтобы создать пример приложения, а затем скопировать введенные двоичные файлы в другой контейнер, содержащий только среду выполнения .NET Core 2,1, чтобы мы максимально сокращаем размер окончательного контейнера.

## <a name="build-and-run-the-app"></a>Сборка и запуск приложения

После написания Dockerfile можно указать DOCKER на нашем Dockerfile и сообщить ему о необходимости сборки и запуска образа:

1. В окне командной строки перейдите в каталог, в котором находится dockerfile, а затем выполните команду [DOCKER Build](https://docs.docker.com/engine/reference/commandline/build/) , чтобы создать контейнер из dockerfile.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Чтобы запустить только что созданный контейнер, выполните команду [DOCKER Run](https://docs.docker.com/engine/reference/commandline/run/) .

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Давайте закрепить эту команду:

   * `-d` указывает DOCKER Тун запускать контейнер "отсоединен", означающее, что консоль не подключена к консоли в контейнере. Контейнер выполняется в фоновом режиме. 
   * `-p 5000:80` указывает DOCKER подключить порт 5000 на узле к порту 80 в контейнере. Каждый контейнер получает свой собственный IP-адрес. По умолчанию ASP .NET прослушивает порт 80. Сопоставление портов позволяет нам переходить на IP-адрес узла на сопоставленном порте, и DOCKER будет пересылать весь трафик на целевой порт в контейнере.
   * `--name myapp` указывает DOCKER предоставить этому контейнеру удобное имя для запроса (вместо того, чтобы искать идентификатор контаиенр, назначенный в среде выполнения DOCKER).
   * `my-asp-app` — это образ, который нужно запустить DOCKER. Это образ контейнера, созданный в качестве кульминацией процесса `docker build`.

3. Откройте веб-браузер веб-браузера и перейдите к `http://localhost:5000`, чтобы просмотреть контейнерное приложение, как показано на следующем снимке экрана:

   >![ASP.NET Core веб-страница, выполняемая с локального узла в контейнере](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Дальнейшие действия

1. Следующим шагом является публикация контейнерного веб-приложения ASP.NET в частном реестре с помощью реестра контейнеров Azure. Это позволяет развернуть его в Организации.

   > [!div class="nextstepaction"]
   > [Создание закрытого реестра контейнеров](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Когда вы [помещаете образ контейнера в реестр](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry), укажите имя только что упакованного приложения ASP.NET (`my-asp-app`) вместе с реестром контейнеров (например: `contoso-container-registry`):

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Дополнительные примеры приложений и связанные с ними файлы dockerfile см. в разделе [Дополнительные примеры контейнеров](../samples.md).

2. После публикации приложения в реестре контейнеров следующим шагом будет развертывание приложения в кластер Kubernetes, созданный с помощью службы Kubernetes Azure.

   > [!div class="nextstepaction"]
   > [Создание кластера Kubernetes](https://docs.microsoft.com/azure/aks/windows-container-cli)
