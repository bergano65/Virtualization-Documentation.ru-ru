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
ms.openlocfilehash: 9ff6256ab9708533f72e9b3210f8a5fd32f4048a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610274"
---
# <a name="dockerfile-on-windows"></a>Dockerfile в Windows

Подсистема Docker содержит средства, автоматизирующие Создание образа контейнера. При создании образов контейнеров вручную, выполнив `docker commit` команд, внедрение процесса автоматического создания образа имеет множество преимуществ, в том числе:

- Сохранение образов контейнеров в виде кода.
- Быстрое и точное воссоздание образов контейнеров для обслуживания и обновления.
- Непрерывная интеграция между образами контейнеров и циклом разработки.

За такую автоматизацию отвечают два компонента Docker— файл Dockerfile и команда `docker build`.

Dockerfile — это текстовый файл, содержащий инструкции, необходимые для создания образа контейнера. Эти инструкции включают идентификацию существующего образа, используемого в качестве основы, команды, выполняемые в процессе создания образа, и команду, которая будет выполняться при развертывании новых экземпляров этого образа контейнера.

Построение docker — команда подсистемы Docker, использующая файл Dockerfile и запускающая процесс создания образа.

В этом разделе показано, как использовать файлы Dockerfile с контейнерами Windows, понять их основные синтаксис и каковы наиболее распространенных инструкции Dockerfile.

В этом документе рассматриваются концепция образов контейнеров и слоев. Если вы хотите узнать больше об изображениях и иерархическое представление изображения, см. [руководство Краткое руководство для изображения](../quick-start/quick-start-images.md).

Полный обзор файлов Dockerfile см. в [справке по Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Базовый синтаксис

В исходной форме файл Dockerfile может быть очень простым. Следующий пример создает образ, включающий IIS и сайт "hello world". Этот пример включает комментарии (обозначенные с помощью `#`), поясняющие каждый шаг. В последующих разделах этой статьи более подробно рассматриваются правила синтаксиса Dockerfile и инструкции Dockerfile.

>[!NOTE]
>Необходимо создать файл Dockerfile без расширения. Для этого в Windows, создайте файл с помощью редактора по выбору, а затем сохраните его с помощью нотации «Dockerfile» (включая кавычки).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Дополнительные примеры файлов Dockerfile для Windows см. в [репозитории Dockerfile для Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Инструкции

Инструкции Dockerfile предоставляют подсистемы Docker инструкции, необходимые для создания образа контейнера. Эти инструкции выполняются по одному, а также в порядке. В следующих примерах показаны наиболее часто используемые инструкции из файлов Dockerfile. Полный список инструкций Dockerfile см. в [справке по Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

Инструкция `FROM` задает образ контейнера, который будет применяться при создании нового образа. Например, при использовании инструкции `FROM microsoft/windowsservercore` полученный образ является производным и зависимым от базового образа ОС Windows Server Core. Если указанный образ отсутствует в системе, где выполняется процесс сборки Docker, подсистема Docker попытается скачать его из общедоступного или частного реестра образов.

Инструкция FROM формат выглядит следующим образом.

```dockerfile
FROM <image>
```

Вот пример команды от:

```dockerfile
FROM microsoft/windowsservercore
```

Более подробные сведения см. в разделе [ссылки FROM](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

Инструкция `RUN` задает команды, которые следует выполнить и поместить в новый образ контейнера. Эти команды могут включать такие элементы, как установка программного обеспечения, создание файлов и папок, а также создание конфигурации среды.

Инструкции RUN выглядит следующим образом.

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Разница между формы исполняемого файла и оболочки заключается в способе `RUN` выполнения инструкции. При использовании формы исполняемого файла указанная программа запускается явным образом.

Ниже приведен пример формы исполняемого файла.

```dockerfile
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

Полученный образ выполняет `powershell New-Item c:/test` команды:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

В отличие от предыдущего примера здесь выполняется та же операция в виде оболочки:

```dockerfile
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

Полученный образ имеет к выполнению инструкции `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Рекомендации по использованию работать с Windows

В Windows при использовании инструкции `RUN` в формате исполняемого файла необходимо экранировать инструкции символы обратной косой черты.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Если целевая программа представляет собой установщик Windows, вам потребуется извлечь программу установки через `/x:<directory>` флаг перед запуском самой (автоматической) процедуры установки. Также необходимо подождать для команды для выхода из перед выполнением других действий. В противном случае процесс будет завершен преждевременно без установки. Дополнительные сведения см. в приведенном ниже примере.

#### <a name="examples-of-using-run-with-windows"></a>Примеры использования работать с Windows

В следующем примере Dockerfile используется DISM для установки служб IIS в образе контейнера:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Этот пример устанавливает распространяемый пакет Visual Studio. `Start-Process` и `-Wait` параметра используются для запуска программы установки. Это гарантирует, что завершения установки осуществляется переход к следующей инструкции в файле Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Подробные сведения об инструкции RUN см. в разделе, [ЗАПУСТИТЕ ссылку](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPY

`COPY` Инструкция копирует файлы и каталоги в файловую систему контейнера. Файлы и каталоги должны находиться в путь, являющийся относительным для Dockerfile.

`COPY` Инструкция в формате выглядит следующим образом:

```dockerfile
COPY <source> <destination>
```

Если источник или назначение содержит пробел, заключите путь в квадратные скобки и двойные кавычки, как показано в следующем примере:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Рекомендации по использованию СКОПИРОВАТЬ с Windows

В Windows для путей назначения необходимо использовать символы косой черты. Например, здесь показаны допустимые `COPY` инструкции:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

В то же время не будут работать символа обратной косой черты следующий формат:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Примеры использования СКОПИРОВАТЬ с Windows

Следующий пример добавляет содержимое исходного каталога в каталог с именем `sqllite` в образе контейнера:

```dockerfile
COPY source /sqlite/
```

Следующий пример добавляет все файлы, начинающиеся с config для `c:\temp` каталог образа контейнера:

```dockerfile
COPY config* c:/temp/
```

Более подробные сведения о `COPY` инструкции см. в [справочнике по COPY](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

Инструкция ADD — как КОПИРОВАНИЯ инструкция, но с еще больше возможностей. Кроме копирования файлов с узла в образ контейнера, инструкция `ADD` также позволяет скопировать файлы из удаленного расположения с помощью задания URL-адреса.

`ADD` Инструкция в формате выглядит следующим образом:

```dockerfile
ADD <source> <destination>
```

Если источник или назначение содержит пробел, заключите путь в квадратные скобки и двойные кавычки:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Рекомендации по запуску добавить с Windows

В Windows для путей назначения необходимо использовать символы косой черты. Например, здесь показаны допустимые `ADD` инструкции:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

В то же время не будут работать символа обратной косой черты следующий формат:

```dockerfile
ADD test1.txt c:\temp\
```

Кроме того, в системе Linux во время копирования инструкция `ADD` распаковывает сжатые пакеты. В Windows эта функция недоступна.

#### <a name="examples-of-using-add-with-windows"></a>Примеры использования ADD с Windows

Следующий пример добавляет содержимое исходного каталога в каталог с именем `sqllite` в образе контейнера:

```dockerfile
ADD source /sqlite/
```

Следующий пример добавляет все файлы, начинающиеся с «config» для `c:\temp` каталог образа контейнера.

```dockerfile
ADD config* c:/temp/
```

Следующий пример скачивает Python для Windows в `c:\temp` каталог образа контейнера.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Более подробные сведения о `ADD` инструкции см. в разделе, [ДОБАВЬТЕ ссылку](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

Инструкция `WORKDIR` задает рабочий каталог для других инструкций Dockerfile, например `RUN` и `CMD`, а также рабочий каталог для запущенных экземпляров образа контейнера.

`WORKDIR` Инструкция в формате выглядит следующим образом:

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

Подробные сведения об `WORKDIR` инструкции см. в [справочнике по WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

Инструкция `CMD` задает команду по умолчанию, выполняемую при развертывании экземпляра образа контейнера. Например, если в контейнере будет размещен веб-сервер NGINX `CMD` может включать инструкции для запуска веб-сервера с помощью команды, такие как `nginx.exe`. Если в файле Dockerfile указано несколько инструкций `CMD`, вычисляется только последняя из них.

`CMD` Инструкция в формате выглядит следующим образом:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Рекомендации по использованию CMD с Windows

В Windows для путей к файлам, указанным в инструкции `CMD`, следует использовать символы косой черты или экранировать символы обратной косой черты `\\`. Ниже перечислены допустимые `CMD` инструкции:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Однако следующий формат без правильной черты не будет работать:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Более подробные сведения о `CMD` инструкции см. в [справочнике по CMD](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Escape-символ

Во многих случаях инструкция Dockerfile должна занимать несколько строк. Для этого можно использовать escape-символа. Escape-символ Dockerfile по умолчанию— обратная косая черта (`\`). Тем не менее так как обратная косая черта также является разделителем путей файлов в Windows, используя занимать несколько строк может вызвать проблемы. Чтобы обойти эту проблему, можно использовать директиву parser изменить escape-символ по умолчанию. Дополнительные сведения о директивах см. в разделе [директивах](https://docs.docker.com/engine/reference/builder/#parser-directives).

В следующем примере показано инструкция RUN, которая занимает несколько строк и использует escape-символ по умолчанию.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Чтобы изменить escape-символ, поместите директиву Parser для escape-символа на первую строку Dockerfile. Это показано в следующем примере.

>[!NOTE]
>Можно использовать только два значения в качестве escape-символы: `\` и `` ` ``.

```dockerfile
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Дополнительные сведения о директиве escape см. в разделе [директиве Escape](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell в Dockerfile

### <a name="powershell-cmdlets"></a>Командлеты PowerShell

Командлеты PowerShell можно выполнять в Dockerfile с `RUN` операции.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Вызовы REST

PowerShell `Invoke-WebRequest` командлета может быть полезно при сборе данных или файлов из веб-службы. Например, при создании образа, включающего в себя Python, вы можете задать `$ProgressPreference` для `SilentlyContinue` для ускорения процессов скачивания, как показано в следующем примере.

```dockerfile
FROM microsoft/windowsservercore

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

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server в настоящее время не поддерживает WebClient.

### <a name="powershell-scripts"></a>Сценарии PowerShell

В некоторых случаях может оказаться полезным скопировать сценарий в контейнеры, которые можно использовать в процессе создания образа, а затем запустите сценарий из контейнера.

>[!NOTE]
>Будет ограничить кэширования слоев образов, а также ухудшает удобочитаемость файла Dockerfile.

Этот пример копирует сценарий с компьютера сборки в контейнер с помощью инструкции `ADD`. Затем этот сценарий выполняется с помощью инструкции RUN.

```dockerfile
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Сборка docker

После создания файла Dockerfile и сохранения его на диск, можно запустить `docker build` для создания нового образа. Команда `docker build` принимает несколько необязательных параметров и путь к файлу Dockerfile. Полную документацию по сборке Docker, включая список всех параметров сборки, см. в разделе [сборки](https://docs.docker.com/engine/reference/commandline/build/#build).

Формат `docker build` команда выглядит следующим образом:

```dockerfile
docker build [OPTIONS] PATH
```

Например следующая команда создает образ с именем «iis».

```dockerfile
docker build -t iis .
```

При инициации процесса сборки, выходных данных указывается состояние и выводятся все возникающие ошибки.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
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

Результат — новый образ контейнера, который в этом примере имеет имя «iis».

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Дополнительные материалы и ссылки

- [Оптимизация файлов Dockerfile и Docker сборки для Windows](optimize-windows-dockerfile.md)
- [Справочник по файлам Dockerfile](https://docs.docker.com/engine/reference/builder/)
