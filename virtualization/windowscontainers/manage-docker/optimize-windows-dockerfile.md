---
title: Оптимизация файлов Dockerfile в Windows
description: Оптимизация файлов Dockerfile для контейнеров Windows.
keywords: docker, контейнеры
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910154"
---
# <a name="optimize-windows-dockerfiles"></a>Оптимизация файлов Dockerfile в Windows

Существует множество способов оптимизации процесса сборки DOCKER и результирующих образов DOCKER. В этой статье объясняется, как работает процесс сборки DOCKER и как оптимально создавать образы для контейнеров Windows.

## <a name="image-layers-in-docker-build"></a>Слои изображений в сборке DOCKER

Прежде чем можно будет оптимизировать сборку DOCKER, необходимо знать, как работает сборка DOCKER. В процессе сборки Docker используется файл Dockerfile, а также поочередно выполняются все активные инструкции, каждая в своем собственном временном контейнере. В результате для каждой активной инструкции создается новый слой образа.

Например, следующий пример Dockerfile использует базовый образ ОС `mcr.microsoft.com/windows/servercore:ltsc2019`, устанавливает службы IIS, а затем создает простой веб-сайт.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Вы можете ожидать, что эта Dockerfile будет создавать изображение с двумя слоями: один для образа ОС контейнера, а второй — со службами IIS и веб-сайтом. Однако фактический образ имеет много уровней, и каждый уровень зависит от него.

Чтобы сделать это более четким, давайте выполним команду `docker history` для образа нашего примера Dockerfile.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Выходные данные показывают, что в этом изображении есть четыре слоя: базовый и три дополнительных слоя, сопоставленных с каждой инструкцией в Dockerfile. Нижний слой (в этом примере — `6801d964fda5`) представляет базовый образ ОС. Одним из слоев является установка IIS. Следующий слой включает новый веб-сайт и т. д.

Файлы dockerfile можно написать, чтобы сократить уровни изображения, оптимизировать производительность сборки и оптимизировать доступность благодаря удобочитаемости. По сути, существует множество способов выполнить одну и ту же задачу сборки образа. Понимание того, как формат Dockerfile влияет на время сборки, и создаваемое им изображение улучшает процесс автоматизации.

## <a name="optimize-image-size"></a>Оптимизировать размер изображения

В зависимости от требований к месту на диске размер изображения может быть важным фактором при создании образов контейнеров DOCKER. Образы контейнеров перемещаются между реестрами и узлом, экспортируются и импортируются и в конечном счете занимают определенное место. В этом разделе вы узнаете, как максимально сокращать размер образа в процессе сборки DOCKER для контейнеров Windows.

Дополнительные сведения о рекомендациях по Dockerfile см. [в статье рекомендации по написанию файлы dockerfile на DOCKER.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Группирование связанных действий

Поскольку каждая инструкция `RUN` создает новый слой в образе контейнера, группирование действий в одну `RUN`ную инструкцию может уменьшить количество слоев в Dockerfile. Хотя минимизация числа слоев может не влиять на размер образа, группирование связанных действий влияет на него, что видно в приведенных ниже примерах.

В этом разделе мы будем сравнивать два примера файлы dockerfile, которые выполняют те же действия. Однако один Dockerfile имеет одну инструкцию для каждого действия, а другая — связанные с ней действия, сгруппированные вместе.

В следующем несгруппированном примере Dockerfile скачивает Python для Windows, устанавливает его и удаляет скачанный файл установки после завершения установки. В этом Dockerfile каждое действие получает собственную инструкцию `RUN`.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

Полученный образ имеет три дополнительных слоя — по одному для каждой инструкции `RUN`.

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Вторым примером является Dockerfile, выполняющий точную операцию. Однако все связанные действия были сгруппированы в рамках одной инструкции `RUN`. Каждый шаг в инструкции `RUN` находится на новой строке Dockerfile, а символ "\\" используется для переноса строк.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Полученный образ имеет только один дополнительный слой для инструкции `RUN`.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Удаление лишних файлов

Если в Dockerfile есть файл, например установщик, который не требуется после его использования, его можно удалить, чтобы уменьшить размер изображения. Это следует делать на том же шаге, где данный файл был скопирован в слой образа. Это предотвращает сохранение файла на слое изображений более низкого уровня.

В следующем примере Dockerfile пакет Python загружается, выполняется, а затем удаляется. Все это выполняется в рамках одной операции `RUN` и создает всего один слой образа.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Оптимизация скорости сборки

### <a name="multiple-lines"></a>Несколько строк

Вы можете разделить операции на несколько отдельных инструкций, чтобы оптимизировать скорость сборки DOCKER. Несколько `RUN` операций увеличивают эффективность кэширования, так как отдельные слои создаются для каждой инструкции `RUN`. Если идентичная инструкция уже выполнялась в другой операции сборки DOCKER, эта операция кэширования (уровень изображения) используется повторно, что приводит к уменьшению времени выполнения сборки DOCKER.

В следующем примере выполняется скачивание, установка и удаление пакетов для приложений Apache и Visual Studio, а затем их очистка путем удаления файлов, которые больше не нужны. Все это выполняется с помощью одной инструкции `RUN`. При обновлении любого из этих действий все действия будут перезапущены.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \

  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \

  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

Полученное изображение имеет два слоя: один для основного образа ОС и один, содержащий все операции из одной инструкции `RUN`.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

По сравнению, здесь приведены те же действия, которые делятся на три инструкции `RUN`. В этом случае каждая инструкция `RUN` кэшируется на уровне образа контейнера, и только те из них, которые были изменены, должны быть перезапущены в последующих сборках Dockerfile.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

Полученный образ состоит из четырех слоев. один уровень для основного образа ОС и все три инструкции `RUN`. Поскольку каждая `RUN` инструкция выполнялась в своем собственном слое, все последующие запуски этого Dockerfile или идентичный набор инструкций в разных Dockerfile будут использовать кэшированные слои изображения, уменьшая время сборки.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Порядок выполнения инструкций важен при работе с кэшами изображений, как показано в следующем разделе.

### <a name="ordering-instructions"></a>Инструкции по упорядочиванию

Файл Dockerfile обрабатывается сверху вниз, при этом каждая инструкция сравнивается с кэшированными слоями. При обнаружении инструкции без кэшированного слоя она и все последующие инструкции обрабатываются в новых слоях образа контейнера. Поэтому порядок инструкций имеет важное значение. Инструкции, которые останутся постоянными, располагайте ближе к началу файла Dockerfile. Инструкции, которые могут изменяться, располагайте ближе к концу файла Dockerfile. Это снижает вероятность отмены существующего кэша.

В следующих примерах показано, как упорядочение инструкций Dockerfile может повлиять на эффективность кэширования. В этом простом примере Dockerfile имеется четыре пронумерованных папки.  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

Полученный образ имеет пять слоев, один для основного образа ОС и все инструкции `RUN`.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Следующий Dockerfile теперь немного изменился, а третья инструкция `RUN` изменилась на новый файл. При запуске сборки Docker для этого файла Dockerfile три первых инструкции, которые идентичны инструкциям з прошлого примера, используют кэшированные слои образа. Однако, поскольку измененная инструкция `RUN` не кэшируется, для измененной инструкции создается новый слой и все последующие инструкции.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

При сравнении идентификаторов образа нового изображения с этим в первом примере этого раздела можно заметить, что первые три слоя снизу вверх являются общими, но четвертый и пятый являются уникальными.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Оптимизация косметических функций

### <a name="instruction-case"></a>Вариант инструкции

Инструкции Dockerfile не учитывают регистр, но в этом случае используется прописная буква. Это повышает удобочитаемость путем различения вызова инструкций и операций с инструкциями. В следующих двух примерах сравниваются Непрописные и прописные Dockerfile.

Ниже приведена непрописная Dockerfile:

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Ниже приведен тот же Dockerfile с использованием прописных букв:

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Перенос строк

Длинные и сложные операции можно разделить на несколько строк с помощью обратной косой черты `\` символа. Следующий файл Dockerfile устанавливает распространяемый пакет Visual Studio, удаляет файлы установщика и затем создает файл конфигурации. Все эти три операции указаны на одной строке.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

Команда может быть разбита с помощью обратной косой черты, чтобы каждая операция из одной инструкции `RUN` была указана в отдельной строке.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Дополнительные материалы и ссылки

[Dockerfile в Windows](manage-windows-dockerfile.md)

[Рекомендации по написанию файлы dockerfile на Docker.com](https://docs.docker.com/engine/reference/builder/)
