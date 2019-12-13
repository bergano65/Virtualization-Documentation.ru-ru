---
title: Dockerfile и контейнеры Windows
description: Создание файлов Dockerfile для контейнеров Windows.
keywords: docker, контейнеры
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 9fef74c029dc3efc220b1f9924d2695cdbaa61be
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909664"
---
# <a name="dockerfile-on-windows"></a>Dockerfile в Windows

Подсистема DOCKER включает средства, автоматизирующие создание образов контейнеров. Вы можете создавать образы контейнеров вручную, выполняя команду `docker commit`. при этом процесс автоматического создания образа имеет множество преимуществ, в том числе:

- Сохранение образов контейнеров в виде кода.
- Быстрое и точное воссоздание образов контейнеров для обслуживания и обновления.
- Непрерывная интеграция между образами контейнеров и циклом разработки.

За такую автоматизацию отвечают два компонента Docker — файл Dockerfile и команда `docker build`.

Dockerfile — это текстовый файл, содержащий инструкции, необходимые для создания нового образа контейнера. Эти инструкции включают идентификацию существующего образа, используемого в качестве основы, команды, выполняемые в процессе создания образа, и команду, которая будет выполняться при развертывании новых экземпляров этого образа контейнера.

Сборка DOCKER — это команда DOCKER Engine, которая использует Dockerfile и запускает процесс создания образа.

В этом разделе показано, как использовать файлы dockerfile с контейнерами Windows, понять их базовый синтаксис и каковы наиболее распространенные инструкции Dockerfile.

В этом документе обсуждается понятие образов контейнеров и уровней образа контейнера. Дополнительные сведения об образах и слоях изображений см. в разделе [базовые образы контейнера](../manage-containers/container-base-images.md).

Полный взгляд на файлы dockerfile см. в [справочнике по Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Базовый синтаксис

В исходной форме файл Dockerfile может быть очень простым. Следующий пример создает образ, включающий IIS и сайт "hello world". Этот пример включает комментарии (обозначенные с помощью `#`), поясняющие каждый шаг. В последующих разделах этой статьи более подробно рассматриваются правила синтаксиса Dockerfile и инструкции Dockerfile.

>[!NOTE]
>Необходимо создать Dockerfile без расширения. Чтобы сделать это в Windows, создайте файл с помощью выбранного редактора, а затем сохраните его с обозначением «Dockerfile» (включая кавычки).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Дополнительные примеры файлы dockerfile для Windows см. в [репозитории Dockerfile для Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Инструкция

Инструкции Dockerfile предоставляют подсистему DOCKER инструкции, необходимые для создания образа контейнера. Эти инструкции выполняются по одному и по порядку. Ниже приведены наиболее часто используемые инструкции в файлы dockerfile. Полный список инструкций Dockerfile см. в [справочнике по Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

Инструкция `FROM` задает образ контейнера, который будет применяться при создании нового образа. Например, при использовании инструкции `FROM mcr.microsoft.com/windows/servercore` полученный образ является производным и зависимым от базового образа ОС Windows Server Core. Если указанный образ отсутствует в системе, где выполняется процесс сборки Docker, подсистема Docker попытается скачать его из общедоступного или частного реестра образов.

Формат инструкции FROM выглядит следующим образом:

```dockerfile
FROM <image>
```

Ниже приведен пример команды FROM:

Чтобы загрузить ltsc2019 версию Windows Server Core из реестра контейнеров (Майкрософт) (мкр), выполните следующие действия.
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Более подробные сведения см. в [справочнике](https://docs.docker.com/engine/reference/builder/#from)по.

### <a name="run"></a>RUN

Инструкция `RUN` задает команды, которые следует выполнить и поместить в новый образ контейнера. Эти команды могут включать такие элементы, как установка программного обеспечения, создание файлов и папок, а также создание конфигурации среды.

Инструкция RUN выглядит следующим образом:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Различие между формой Exec и Shell заключается в том, как выполняется инструкция `RUN`. При использовании формы исполняемого файла указанная программа запускается явным образом.

Ниже приведен пример вида exec:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

Полученный образ выполняет команду `powershell New-Item c:/test`:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Для сравнения, следующий пример выполняет ту же операцию в форме оболочки:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

Полученный образ содержит инструкцию Run `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Рекомендации по использованию команды "запустить с Windows"

В Windows при использовании инструкции `RUN` в формате исполняемого файла необходимо экранировать инструкции символы обратной косой черты.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Если целевая программа является установщиком Windows, необходимо извлечь программу установки с помощью флага `/x:<directory>`, прежде чем можно будет запустить реальную (автоматическую) процедуру установки. Необходимо также дождаться завершения команды, прежде чем выполнять все остальные действия. В противном случае процесс завершится преждевременно без установки каких-либо компонентов. Дополнительные сведения см. в приведенном ниже примере.

#### <a name="examples-of-using-run-with-windows"></a>Примеры использования команды RUN с Windows

В следующем примере Dockerfile использует DISM для установки служб IIS в образе контейнера:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Этот пример устанавливает распространяемый пакет Visual Studio. `Start-Process` и параметр `-Wait` используются для запуска установщика. Это гарантирует, что установка завершится до перехода к следующей инструкции в Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Подробные сведения о инструкции RUN см. в [справочнике по запуску](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPY

Инструкция `COPY` копирует файлы и каталоги в файловую систему контейнера. Файлы и каталоги должны находиться в пути относительно Dockerfile.

Формат инструкции `COPY` выглядит следующим образом:

```dockerfile
COPY <source> <destination>
```

Если источник или назначение содержит пробелы, заключите путь в квадратные скобки и двойные кавычки, как показано в следующем примере:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Рекомендации по использованию функции копирования в Windows

В Windows для путей назначения необходимо использовать символы косой черты. Например, это допустимые `COPY` инструкции:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

В то же время следующий формат с обратными косыми чертами не будет работать:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Примеры использования функции копирования в Windows

В следующем примере содержимое исходного каталога добавляется в каталог с именем `sqllite` в образе контейнера:

```dockerfile
COPY source /sqlite/
```

В следующем примере все файлы, начинающиеся с config, будут добавлены в каталог `c:\temp` образа контейнера:

```dockerfile
COPY config* c:/temp/
```

Более подробные сведения о `COPY` инструкции см. в [справочнике по копированию](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ДОБАВИТЬ

Инструкция ADD похожа на инструкцию COPY, но с еще большим количеством возможностей. Кроме копирования файлов с узла в образ контейнера, инструкция `ADD` также позволяет скопировать файлы из удаленного расположения с помощью задания URL-адреса.

Формат инструкции `ADD` выглядит следующим образом:

```dockerfile
ADD <source> <destination>
```

Если источник или назначение содержат пробелы, заключите путь в квадратные скобки и двойные кавычки:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Рекомендации по выполнению добавления с помощью Windows

В Windows для путей назначения необходимо использовать символы косой черты. Например, это допустимые `ADD` инструкции:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

В то же время следующий формат с обратными косыми чертами не будет работать:

```dockerfile
ADD test1.txt c:\temp\
```

Кроме того, в системе Linux во время копирования инструкция `ADD` распаковывает сжатые пакеты. В Windows эта функция недоступна.

#### <a name="examples-of-using-add-with-windows"></a>Примеры использования команды "добавить с Windows"

В следующем примере содержимое исходного каталога добавляется в каталог с именем `sqllite` в образе контейнера:

```dockerfile
ADD source /sqlite/
```

В следующем примере будут добавлены все файлы, начинающиеся с "config", в каталог `c:\temp` образа контейнера.

```dockerfile
ADD config* c:/temp/
```

В следующем примере будет загружен Python для Windows в каталог `c:\temp` образа контейнера.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Более подробные сведения о `ADD` инструкции см. в разделе [Добавление ссылки](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

Инструкция `WORKDIR` задает рабочий каталог для других инструкций Dockerfile, например `RUN` и `CMD`, а также рабочий каталог для запущенных экземпляров образа контейнера.

Формат инструкции `WORKDIR` выглядит следующим образом:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Рекомендации по использованию WORKDIR с Windows

Если в Windows рабочий каталог содержит обратную косую черту, ее следует экранировать.

```dockerfile
WORKDIR c:\\windows
```

**Примеры**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Подробные сведения о `WORKDIR` инструкции см. в [справочнике по WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

Инструкция `CMD` задает команду по умолчанию, выполняемую при развертывании экземпляра образа контейнера. Например, если контейнер будет размещать веб-сервер NGINX, `CMD` может содержать инструкции по запуску веб-сервера с помощью команды, такой как `nginx.exe`. Если в файле Dockerfile указано несколько инструкций `CMD`, вычисляется только последняя из них.

Формат инструкции `CMD` выглядит следующим образом:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Рекомендации по использованию CMD с Windows

В Windows для путей к файлам, указанным в инструкции `CMD`, следует использовать символы косой черты или экранировать символы обратной косой черты `\\`. Ниже приведены допустимые `CMD` инструкции.

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Однако следующий формат без соответствующих косых черт не будет работать:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Более подробные сведения о `CMD` инструкции см. в [справочнике по командлетам](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Escape-символ

Во многих случаях инструкция Dockerfile должна охватывать несколько строк. Для этого можно использовать escape-символ. Escape-символ Dockerfile по умолчанию — обратная косая черта (`\`). Однако, поскольку обратная косая черта также является разделителем пути к файлу в Windows, использование его для разделения нескольких строк может вызвать проблемы. Чтобы обойти это, можно использовать директиву анализатора для изменения escape-символа по умолчанию. Дополнительные сведения об директивах синтаксического анализатора см. в разделе [директивы средства синтаксического](https://docs.docker.com/engine/reference/builder/#parser-directives)анализа.

В следующем примере показана одна инструкция RUN, охватывающая несколько строк с помощью escape-символа по умолчанию:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Чтобы изменить escape-символ, поместите директиву Parser для escape-символа на первую строку Dockerfile. Это можно увидеть в следующем примере.

>[!NOTE]
>В качестве escape-символов можно использовать только два значения: `\` и `` ` ``.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Дополнительные сведения об директиве escape-анализатора см. в разделе [escape-директива средства синтаксического](https://docs.docker.com/engine/reference/builder/#escape)анализа.

## <a name="powershell-in-dockerfile"></a>PowerShell в Dockerfile

### <a name="powershell-cmdlets"></a>Командлеты PowerShell

Командлеты PowerShell можно выполнять в Dockerfile с помощью операции `RUN`.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Вызовы RESTFUL

Командлет PowerShell `Invoke-WebRequest` может быть полезен при сборе информации или файлов из веб-службы. Например, если создается образ, включающий Python, можно задать для `$ProgressPreference` значение `SilentlyContinue` для ускорения загрузки, как показано в следующем примере.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` также работает в Nano Server.

Кроме того, с помощью PowerShell можно скачивать файлы во время создания образа, используя библиотеку .NET WebClient. Это может повысить производительность скачивания. Следующий пример скачивает программное обеспечение Python, используя библиотеку WebClient.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server в настоящее время не поддерживает WebClient.

### <a name="powershell-scripts"></a>Сценарии PowerShell

В некоторых случаях может оказаться полезным скопировать сценарий в контейнеры, используемые в процессе создания образа, а затем запустить его из контейнера.

>[!NOTE]
>Это ограничит кэширование на уровне изображения и уменьшает удобочитаемость Dockerfile.

Этот пример копирует сценарий с компьютера сборки в контейнер с помощью инструкции `ADD`. Затем этот сценарий выполняется с помощью инструкции RUN.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Сборка DOCKER

После создания и сохранения Dockerfile на диске можно запустить `docker build`, чтобы создать новый образ. Команда `docker build` принимает несколько необязательных параметров и путь к файлу Dockerfile. Полную документацию по сборке DOCKER, включая список всех параметров сборки, см. в [справочнике по сборке](https://docs.docker.com/engine/reference/commandline/build/#build).

Формат команды `docker build` выглядит следующим образом:

```dockerfile
docker build [OPTIONS] PATH
```

Например, следующая команда создаст образ с именем "IIS".

```dockerfile
docker build -t iis .
```

После запуска процесса сборки выходные данные будут указывать на состояние и возвращать все возникшие ошибки.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

Результатом является новый образ контейнера, который в этом примере называется "IIS".

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Дополнительные материалы и ссылки

- [Оптимизация сборки файлы dockerfile и DOCKER для Windows](optimize-windows-dockerfile.md)
- [Справочник по Dockerfile](https://docs.docker.com/engine/reference/builder/)
