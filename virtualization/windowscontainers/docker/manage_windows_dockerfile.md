---
title: "Dockerfile и контейнеры Windows"
description: "Создание файлов Dockerfile для контейнеров Windows."
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: ea84ac6c688fa258c9b72f50565ec6e21e8051db

---

# Dockerfile в Windows

Подсистема Docker содержит средства, автоматизирующие создание образов контейнеров. Хотя образы контейнеров можно создавать вручную с помощью команды `docker commit`, внедрение процесса автоматического создания образа предоставляет множество преимуществ, в том числе:

- Сохранение образов контейнеров в виде кода.
- Быстрое и точное воссоздание образов контейнеров для обслуживания и обновления.
- Непрерывная интеграция между образами контейнеров и циклом разработки.

За такую автоматизацию отвечают два компонента Docker — файл Dockerfile и команда `docker build`.

- **Dockerfile** — это текстовый файл с инструкциями, необходимыми для создания образа контейнера. Эти инструкции включают идентификацию существующего образа, используемого в качестве основы, команды, выполняемые в процессе создания образа, и команду, которая будет выполняться при развертывании новых экземпляров этого образа контейнера.
- **Docker build** — команда подсистемы Docker, использующая файл Dockerfile и запускающая процесс создания образа.

В этом документе описывается использование файла Dockerfile с контейнерами Windows, синтаксис, а также рассматриваются часто применяемые инструкции Dockerfile. 

Кроме того, в нем поясняется концепция образов контейнеров и их слоев. Дополнительные сведения об образах и разделении их на слои см. в статье [Управление образами контейнеров Windows](../management/manage_images.md). 

Полноценный обзор файлов Dockerfile см. в [справке по Dockerfile на сайте docker.com]( https://docs.docker.com/engine/reference/builder/).

## Общие сведения о файлах Dockerfile

### Базовый синтаксис

В исходной форме файл Dockerfile может быть очень простым. Следующий пример создает образ, включающий IIS и сайт "hello world". Этот пример включает комментарии (обозначенные с помощью `#`), поясняющие каждый шаг. В последующих разделах этой статьи более подробно рассматриваются правила синтаксиса Dockerfile и инструкции Dockerfile.

```none
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM windowsservercore

# Metadata indicating an image maintainer.
MAINTAINER jshelton@contoso.com

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an html file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Дополнительные примеры файлов Dockerfile для Windows см. в документе [Dockerfile for Windows Repository (Dockerfile для репозитория Windows)] (https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## Инструкции

Инструкции Dockerfile сообщают подсистеме Docker о необходимых шагах для создания образа контейнера. Эти инструкции выполняются по порядку, одна за другой. Ниже приведены сведения о некоторых основных инструкциях Dockerfile. Полный список инструкций Dockerfile см. в [Справочник по файлам Dockerfile на сайте Docker.com] (https://docs.docker.com/engine/reference/builder/).

### FROM

Инструкция `FROM` задает образ контейнера, который будет применяться при создании нового образа. Например, при использовании инструкции `FROM windowsservercore` полученный образ является производным и зависимым от базового образа ОС Windows Server Core. Если указанный образ отсутствует в системе, где выполняется процесс сборки Docker, подсистема Docker попытается скачать его из общедоступного или частного реестра образов.

**Формат**

Инструкция FROM имеет следующий формат: 

```
FROM <image>
```

**Пример**

```
FROM windowsservercore
```

Подробные сведения об инструкции FROM см. в [справочнике по FROM на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#from). 

### RUN

Инструкция `RUN` задает команды, которые следует выполнить и поместить в новый образ контейнера. Эти команды могут включать такие элементы, как установка программного обеспечения, создание файлов и папок, а также создание конфигурации среды.

**Формат**

Инструкция RUN имеет следующий формат: 

```none
# exec form

RUN ["<executable", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Различие между формой исполняемого файла (exec form) и формой оболочки (shell form) заключается в способе выполнения инструкции `RUN`. При использовании формы исполняемого файла указанная программа запускается явным образом. 

В следующем примере используется форма исполняемого файла.

```none
FROM windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

Анализируя полученный образ, можно сделать вывод изображения, что была выполнена команда `powershell new-item c:/test`.

```none
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

В отличие от предыдущего примера здесь выполняется та же операция, однако с использованием формы оболочки.

```none
FROM windowsservercore

RUN powershell new-item c:\test
```

Это приводит к выполнению инструкции `cmd /S /C powershell new-item c:\test`. 

```none
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell new-item c:\test   30.76 MB
```

**Рекомендации для Windows**
 
В Windows при использовании инструкции `RUN` в формате исполняемого файла необходимо экранировать инструкции символы обратной косой черты.

```none
RUN ["powershell", "New-Item", "c:\\test"]
```

Если целевая программа представляет собой установщик Windows, то перед запуском самой процедуры (автоматической) установки необходимо извлечь установочные файлы, указав флаг `/x:<directory>`. Также необходимо подождать, пока команда завершит свою работу. В противном случае процесс будет завершен преждевременно и без установки. Дополнительные сведения см. в приведенном ниже примере.

**Примеры**

Этот пример использует систему DISM для установки служб IIS в образе контейнера.
```none
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Этот пример устанавливает распространяемый пакет Visual Studio. Обратите внимание, что `start-process` и параметр `-wait` используются для запуска программы установки. Это гарантирует, что установка будет завершена до перехода к следующему шагу в Dockerfile.

```none
RUN start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
``` 

Подробные сведения об инструкции RUN см. в [справочнике по RUN на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#run). 

### COPY

Инструкция `COPY` копирует файлы и каталоги в файловую систему контейнера. Эти файлы и каталоги должны иметь путь, являющийся относительным для Dockerfile.

**Формат**

Инструкция `COPY` имеет следующий формат: 

```none
COPY <source> <destination>
``` 

Если источник или назначение содержит пробел, заключите путь в квадратные скобки и двойные кавычки.
 
```none
COPY ["<source>" "<destination>"]
```

**Рекомендации для Windows**
 
В Windows для путей назначения необходимо использовать символы косой черты. Например, здесь показаны допустимые инструкции `COPY`.

```none
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Однако приведенная ниже инструкция работать не будет.

```none
COPY test1.txt c:\temp\
```

**Примеры**

Этот пример добавляет содержимое исходного каталога в каталог с именем `sqllite` в образе контейнера.
```none
COPY source /sqlite/
```

Этот пример добавляет все файлы, начинающиеся с config, в каталог `c:\temp` образа контейнера.
```none
COPY config* c:/temp/
```

Подробные сведения об инструкции `COPY` см. в [справочнике по COPY на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#copy).

### ДОБАВИТЬ

Инструкция ADD во многом похожа на инструкцию COPY, однако имеет дополнительные возможности. Кроме копирования файлов с узла в образ контейнера, инструкция `ADD` также позволяет скопировать файлы из удаленного расположения с помощью задания URL-адреса.

**Формат**

Инструкция `ADD` имеет следующий формат: 

```none
ADD <source> <destination>
``` 

Если источник или назначение содержит пробел, заключите путь в квадратные скобки и двойные кавычки.
 
```none
ADD ["<source>" "<destination>"]
```

**Рекомендации для Windows**
 
В Windows для путей назначения необходимо использовать символы косой черты. Например, здесь показаны допустимые инструкции `ADD`.

```none
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Однако приведенная ниже инструкция работать не будет.

```none
ADD test1.txt c:\temp\
```

Кроме того, в системе Linux во время копирования инструкция `ADD` распаковывает сжатые пакеты. В Windows эта функция недоступна.

**Примеры**

Этот пример добавляет содержимое исходного каталога в каталог с именем `sqllite` в образе контейнера.
```none
ADD source /sqlite/
```

Этот пример добавляет все файлы, начинающиеся с config, в каталог `c:\temp` образа контейнера.
```none
ADD config* c:/temp/
```

Этот пример скачивает Python для Windows в каталог `c:\temp` образа контейнера.
```none
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Подробные сведения об инструкции `ADD` см. в [справочнике по ADD на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#add). 

### WORKDIR

Инструкция `WORKDIR` задает рабочий каталог для других инструкций Dockerfile, таких как `RUN`, `CMD`, а также рабочий каталог для запущенных экземпляров образа контейнера.

**Формат**

Инструкция `WORKDIR` имеет следующий формат: 

```none
WORKDIR <path to working directory>
``` 

**Рекомендации для Windows**

Если в Windows рабочий каталог содержит обратную косую черту, ее следует экранировать.

```none
WORKDIR c:\\windows
```

**Примеры**

```none
WORKDIR c:\\Apache24\\bin
```

Подробные сведения об инструкции `WORKDIR` см. в [справочнике по WORKDIR на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#workdir). 

### CMD

Инструкция `CMD` задает команду по умолчанию, выполняемую при развертывании экземпляра образа контейнера. Например, если в контейнере будет размещен веб-сервер NGINX, `CMD` может включать инструкции для запуска этого веб-сервера, например `nginx.exe`. Если в файле Dockerfile указано несколько инструкций `CMD`, вычисляется только последняя из них.

**Формат**

Инструкция `CMD` имеет следующий формат: 

```none
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

**Рекомендации для Windows**

В Windows для путей к файлам, указанным в инструкции `CMD`, следует использовать символы косой черты или экранировать символы обратной косой черты `\\`. Например, здесь показаны допустимые инструкции `CMD`.

```none
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```
Однако приведенная ниже инструкция работать не будет.

```none
CMD c:\Apache24\bin\httpd.exe -w
```

Подробные сведения об инструкции `CMD` см. в [справочнике по CMD на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#cmd). 

## Escape-символ

Во многих случаях инструкция Dockerfile должна занимать несколько строк. Этого можно добиться с помощью escape-символа. Escape-символ Dockerfile по умолчанию — обратная косая черта (`\`). Так как обратная косая черта также является разделителем путей файлов в Windows, это может вызвать проблемы. Чтобы изменить escape-символ по умолчанию, можно использовать директиву Parser. Дополнительные сведения о директивах Parser см. в разделе [Директивы Parser на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#parser-directives).

В следующем примере показана инструкция RUN, которая занимает несколько строк и использует escape-символ по умолчанию.

```none
FROM windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Чтобы изменить escape-символ, поместите директиву Parser для escape-символа на первую строку Dockerfile. Это показано в следующем примере.

> Обратите внимание, что в качестве escape-символов можно использовать только два значения: `\` и `` ` ``.

```none
# escape=`

FROM windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Дополнительные сведения о директиве Parser для escape-символа см. в разделе [Директива Parser для escape-символов на сайте Docker.com]( https://docs.docker.com/engine/reference/builder/#escape).

## PowerShell в Dockerfile

### Команды PowerShell

Команды PowerShell можно выполнять в файле Dockerfile с помощью операции `RUN`. 

```none
FROM windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### Вызовы REST

PowerShell и команда `Invoke-WebRequest` могут оказаться полезными при сборе данных или файлов из веб-службы. Например, при создании образа, включающего в себя Python, можно использовать приведенный ниже пример.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Сейчас Invoke-WebRequest в Nano Server не поддерживается.

Кроме того, с помощью PowerShell можно скачивать файлы во время создания образа, используя библиотеку .NET WebClient. Это может повысить производительность скачивания. Следующий пример скачивает программное обеспечение Python, используя библиотеку WebClient.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Сейчас WebClient в Nano Server не поддерживается.

### Сценарии оболочки PowerShell

В некоторых случаях может оказаться удобным скопировать сценарий в контейнеры, используемые при создании образа, и затем запустить их из контейнера. Примечание. Это ограничивает возможности кэширования слоев образов, а также ухудшает удобочитаемость файла Dockerfile.

Этот пример копирует сценарий с компьютера сборки в контейнер с помощью инструкции `ADD`. Затем этот сценарий выполняется с помощью инструкции RUN.

```
FROM windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## Сборка Docker 

После создания файла Dockerfile и сохранения его на диск можно запустить `docker build` для создания нового образа. Команда `docker build` принимает несколько необязательных параметров и путь к файлу Dockerfile. Полную документацию по сборке Docker, включая список всех параметров сборки, см. в [описании сборки на сайте Docker.com](https://docs.docker.com/engine/reference/commandline/build/#build).

```none
Docker build [OPTIONS] PATH
```
Например, следующая команда создает образ с именем "iis":

```none
docker build -t iis .
```

При инициации процесса сборки в выходных данных указывается состояние и выводятся все возникающие ошибки.

```none
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM windowsservercore
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

В результате создается новый образ контейнера, который в этом примере имеет имя "iis".

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## Дополнительные материалы и справочники

[Optimize Dockerfiles and Docker build for Windows] (Оптимизация файлов Dockerfile и сборка Docker для Windows) (./optimize_windows_dockerfile.md)

[Справочник по файлам Dockerfile на сайте Docker.com](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Sep16_HO4-->


