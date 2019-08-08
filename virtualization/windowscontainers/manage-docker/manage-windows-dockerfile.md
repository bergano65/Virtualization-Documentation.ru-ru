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
ms.openlocfilehash: f23fe8c5e5ad9dc3257f8b99d239b5fc97607add
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998241"
---
# <a name="dockerfile-on-windows"></a>Dockerfile в Windows

Подсистема стыковки содержит инструменты для автоматизации создания изображений контейнера. Несмотря на то, что вы можете создавать изображения контейнера `docker commit` вручную, выполнив команду, применив процесс автоматического создания образа, вы сможете воспользоваться одним из самых разнообразных преимуществ, в том числе:

- Сохранение образов контейнеров в виде кода.
- Быстрое и точное воссоздание образов контейнеров для обслуживания и обновления.
- Непрерывная интеграция между образами контейнеров и циклом разработки.

За такую автоматизацию отвечают два компонента Docker— файл Dockerfile и команда `docker build`.

Доккерфиле — это текстовый файл, содержащий инструкции, необходимые для создания нового изображения контейнера. Эти инструкции включают идентификацию существующего образа, используемого в качестве основы, команды, выполняемые в процессе создания образа, и команду, которая будет выполняться при развертывании новых экземпляров этого образа контейнера.

Сборка Dock — это команда подпрограммы-обработчика, которая потребляет Доккерфиле и инициирует процесс создания изображения.

В этой статье рассказывается о том, как использовать Доккерфилес с контейнерами Windows, понимать их базовый синтаксис и наиболее распространенные инструкции по Доккерфиле.

В этом документе обсуждается концепция изображений контейнера и слоев изображений контейнера. Если вы хотите узнать больше о изображениях и слоях изображений, ознакомьтесь [с руководством краткое руководство](../quick-start/quick-start-images.md)по работе с изображениями.

Полный взгляд на Доккерфилес можно найти в разделе [Справочник по доккерфиле](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Базовый синтаксис

В исходной форме файл Dockerfile может быть очень простым. Следующий пример создает образ, включающий IIS и сайт "hello world". Этот пример включает комментарии (обозначенные с помощью `#`), поясняющие каждый шаг. В последующих разделах этой статьи более подробно рассматриваются правила синтаксиса Dockerfile и инструкции Dockerfile.

>[!NOTE]
>Доккерфиле должно быть создано без расширения. Чтобы сделать это в Windows, создайте файл с вашим редактором, а затем сохраните его с помощью нотации "Доккерфиле" (в том числе кавычек).

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

Дополнительные примеры Доккерфилес для Windows можно найти в репозитории [доккерфиле для Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Инструкции

Доккерфиле инструкции предоставляют подсистему подсистемы стыковки о том, что необходимо для создания изображения контейнера. Эти инструкции выполняются один на один по заказу. Ниже приведены примеры наиболее часто используемых инструкций в Доккерфилес. Полный список Доккерфиле инструкций можно найти в разделе Справочник по [доккерфиле](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

Инструкция `FROM` задает образ контейнера, который будет применяться при создании нового образа. Например, при использовании инструкции `FROM microsoft/windowsservercore` полученный образ является производным и зависимым от базового образа ОС Windows Server Core. Если указанный образ отсутствует в системе, где выполняется процесс сборки Docker, подсистема Docker попытается скачать его из общедоступного или частного реестра образов.

Формат инструкции FROM имеет следующий вид:

```dockerfile
FROM <image>
```

Ниже приведен пример команды FROM.

Чтобы загрузить версию ltsc2019 версии Windows Server Core из реестра Microsoft Container (мкр), выполните указанные ниже действия.
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Более подробную информацию можно найти в разделе [from (Справка](https://docs.docker.com/engine/reference/builder/#from)).

### <a name="run"></a>RUN

Инструкция `RUN` задает команды, которые следует выполнить и поместить в новый образ контейнера. Эти команды могут включать такие элементы, как установка программного обеспечения, создание файлов и папок, а также создание конфигурации среды.

Инструкция по ЗАПУСКу выглядит примерно так:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Различие между формой Exec и Shell заключается в способе выполнения `RUN` инструкции. При использовании формы исполняемого файла указанная программа запускается явным образом.

Ниже приведен пример формы Exec.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

Полученное изображение запускает `powershell New-Item c:/test` команду:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Напротив, в следующем примере одна и та же операция выполняется в виде оболочки:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

У конечного изображения есть инструкция по запуску `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Рекомендации по использованию функции "выполнить в Windows"

В Windows при использовании инструкции `RUN` в формате исполняемого файла необходимо экранировать инструкции символы обратной косой черты.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Если конечная программа является установщиком Windows, вам потребуется извлечь настройку с помощью `/x:<directory>` флага, прежде чем можно будет запустить фактическую (автоматическую) процедуру установки. Вы также должны дождаться завершения команды, прежде чем делать что-то еще. В противном случае процесс завершится преждевременно, не устанавливая ничего. Дополнительные сведения см. в приведенном ниже примере.

#### <a name="examples-of-using-run-with-windows"></a>Примеры использования функции "ЗАПУСКАТЬ с Windows"

В следующем примере Доккерфиле с помощью DISM устанавливаются службы IIS в контейнерном изображении:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Этот пример устанавливает распространяемый пакет Visual Studio. `Start-Process` `-Wait` параметр используется для запуска установщика. Это гарантирует, что установка завершится, прежде чем переходить к следующей инструкции в Доккерфиле.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Подробные сведения о инструкции по ЗАПУСКу можно найти в разделе [Справка](https://docs.docker.com/engine/reference/builder/#run)по запуску.

### <a name="copy"></a>COPY

`COPY` Инструкция копирует файлы и каталоги в файловую систему контейнера. Файлы и каталоги должны находиться в пути, относящихся к Доккерфиле.

Формат `COPY` инструкции выглядит следующим образом:

```dockerfile
COPY <source> <destination>
```

Если в качестве источника или назначения используются пробелы, заключите путь в квадратные скобки и двойные кавычки, как показано в следующем примере:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Рекомендации по использованию функции "Копировать вместе с Windows"

В Windows для путей назначения необходимо использовать символы косой черты. Например, можно использовать `COPY` следующие инструкции:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

В то же время не работают следующие форматы с обратной косой чертой:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Примеры использования команды "Копировать в Windows"

В следующем примере показано, как добавить содержимое исходного каталога в каталог, указанный `sqllite` в рисунке Container:

```dockerfile
COPY source /sqlite/
```

В следующем примере все файлы, которые начинаются с config, будут добавлены `c:\temp` в каталог изображения контейнера:

```dockerfile
COPY config* c:/temp/
```

Более подробные сведения об этой `COPY` инструкции можно найти в [ссылке Копировать](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

Инструкция ADD похожа на инструкцию COPY, но с помощью еще большего количества возможностей. Кроме копирования файлов с узла в образ контейнера, инструкция `ADD` также позволяет скопировать файлы из удаленного расположения с помощью задания URL-адреса.

Формат `ADD` инструкции выглядит следующим образом:

```dockerfile
ADD <source> <destination>
```

Если исходный или конечный объект содержат пробелы, заключите его в квадратные скобки и двойные кавычки.

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Рекомендации по запуску команды "добавить в Windows"

В Windows для путей назначения необходимо использовать символы косой черты. Например, можно использовать `ADD` следующие инструкции:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

В то же время не работают следующие форматы с обратной косой чертой:

```dockerfile
ADD test1.txt c:\temp\
```

Кроме того, в системе Linux во время копирования инструкция `ADD` распаковывает сжатые пакеты. В Windows эта функция недоступна.

#### <a name="examples-of-using-add-with-windows"></a>Примеры использования команды "добавить в Windows"

В следующем примере показано, как добавить содержимое исходного каталога в каталог, указанный `sqllite` в рисунке Container:

```dockerfile
ADD source /sqlite/
```

В следующем примере будут добавлены все файлы, которые начинаются с "config", `c:\temp` в каталог изображения контейнера.

```dockerfile
ADD config* c:/temp/
```

В следующем примере кода Python для Windows загружается `c:\temp` в каталог изображения контейнера.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Более подробные сведения об этой `ADD` инструкции можно найти в разделе [Добавление ссылки](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

Инструкция `WORKDIR` задает рабочий каталог для других инструкций Dockerfile, например `RUN` и `CMD`, а также рабочий каталог для запущенных экземпляров образа контейнера.

Формат `WORKDIR` инструкции выглядит следующим образом:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Рекомендации по использованию ВОРКДИР в Windows

Если в Windows рабочий каталог содержит обратную косую черту, ее следует экранировать.

```dockerfile
WORKDIR c:\\windows
```

**Примеры**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Подробное описание `WORKDIR` инструкции можно найти в разделе [Справка по воркдир](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

Инструкция `CMD` задает команду по умолчанию, выполняемую при развертывании экземпляра образа контейнера. Например, если контейнер будет размещен на веб-сервере НГИНКС, в нем `CMD` могут содержаться инструкции по запуску веб-сервера с помощью `nginx.exe`команды например. Если в файле Dockerfile указано несколько инструкций `CMD`, вычисляется только последняя из них.

Формат `CMD` инструкции выглядит следующим образом:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Рекомендации по использованию CMD в Windows

В Windows для путей к файлам, указанным в инструкции `CMD`, следует использовать символы косой черты или экранировать символы обратной косой черты `\\`. Ниже приведены допустимые `CMD` инструкции.

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Однако следующий формат без заданной косой черты не будет работать.

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Более подробные сведения о `CMD` инструкции можно найти в разделе Справочник по [cmd](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Символ переключения режима

Во многих случаях инструкция Доккерфиле должна охватывать несколько строк. Для этого можно использовать escape-символ. Escape-символ Dockerfile по умолчанию— обратная косая черта (`\`). Тем не менее, так как обратная косая черта также является разделителем пути к файлу в Windows, использование его в нескольких строках может привести к возникновению проблем. Чтобы избежать этого, можно использовать директиву анализатора для изменения символа перехода по умолчанию. Дополнительные сведения об директивах синтаксического анализаторе: в разделе [директивы средства синтаксического анализа](https://docs.docker.com/engine/reference/builder/#parser-directives).

В следующем примере показана одна инструкция запуска, которая охватывает несколько строк с помощью escape-символа по умолчанию:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Чтобы изменить escape-символ, поместите директиву Parser для escape-символа на первую строку Dockerfile. Это может быть показано в приведенном ниже примере.

>[!NOTE]
>В качестве escape-знаков можно использовать только два значения `\` : `` ` ``и.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Дополнительные сведения об директиве средства синтаксического анализа можно найти в разделе [директива синтаксического анализатора escape](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell в Dockerfile

### <a name="powershell-cmdlets"></a>Командлеты PowerShell

Командлеты PowerShell можно запускать в Доккерфиле с помощью `RUN` операции.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Звонки в другие телефоны

`Invoke-WebRequest` Командлет PowerShell может быть полезен при сборе данных или файлов из веб-службы. Например, если вы создаете изображение, включающее Python, вы можете настроить `$ProgressPreference` `SilentlyContinue` его для более быстрых Скачиваний, как показано в следующем примере.

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
>`Invoke-WebRequest` также работает на Nano Server.

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

В некоторых случаях может оказаться полезным скопировать сценарий в контейнеры, которые вы используете во время создания изображения, а затем запустить сценарий из него в контейнере.

>[!NOTE]
>Это ограничит количество кэшированных уровней изображения и уменьшает возможности чтения Доккерфиле.

Этот пример копирует сценарий с компьютера сборки в контейнер с помощью инструкции `ADD`. Затем этот сценарий выполняется с помощью инструкции RUN.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Сборка Dock

После создания и сохранения Доккерфиле на диске можно `docker build` создать новое изображение. Команда `docker build` принимает несколько необязательных параметров и путь к файлу Dockerfile. Полную документацию по сборке Dock, включая список всех параметров сборки, можно найти в [ссылке сборка](https://docs.docker.com/engine/reference/commandline/build/#build).

Формат `docker build` команды выглядит так:

```dockerfile
docker build [OPTIONS] PATH
```

Например, следующая команда создаст изображение с именем "IIS".

```dockerfile
docker build -t iis .
```

Когда инициируется процесс сборки, выходные данные указывают состояние и возвращают все возникшие ошибки.

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

Результат — это новый контейнерный объект, который в этом примере называется "IIS".

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Дальнейшие чтение и ссылки

- [Оптимизация сборки Доккерфилес и Dock для Windows](optimize-windows-dockerfile.md)
- [Ссылка на доккерфиле](https://docs.docker.com/engine/reference/builder/)
