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
ms.openlocfilehash: 056ab87189e8e423df5758be0f622a43b92c9056
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882957"
---
# <a name="optimize-windows-dockerfiles"></a>Оптимизация файлов Dockerfile в Windows

Существует множество способов оптимизации процесса сборки Dock и конечных изображений стыковочного узла. В этой статье объясняется, как работает процесс сборки дока и как оптимально создавать изображения для контейнеров Windows.

## <a name="image-layers-in-docker-build"></a>Слои изображения в сборке Dock

Прежде чем вы сможете оптимизировать свою сборку дока, вам нужно знать, как работает сборка Dock. В процессе сборки Docker используется файл Dockerfile, а также поочередно выполняются все активные инструкции, каждая в своем собственном временном контейнере. В результате для каждой активной инструкции создается новый слой образа.

Например, в следующем примере Доккерфиле используется образ `windowsservercore` базовой ОС, установка IIS и создание простого веб-сайта.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Возможно, вы захотите, чтобы этот Доккерфиле создал изображение с двумя слоями, один — для образа ОС контейнера, а второй — со службами IIS и веб-сайтом. Однако фактическое изображение содержит много слоев, и каждый из них зависит от него.

Чтобы сделать этот элемент более четким, давайте добавим `docker history` команду к изображению, которое мы сделали в нашем примере доккерфиле.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

В выходных данных показано, что в этом изображении есть четыре слоя: базовый уровень и три дополнительных слоя, которые сопоставлены с каждой инструкцией в Доккерфиле. Нижний слой (в этом примере— `6801d964fda5`) представляет базовый образ ОС. Один слой вверх — установка IIS. Следующий слой включает новый веб-сайт ит.д.

Доккерфилес можно написать так, чтобы свести к минимуму слои изображений, оптимизировать производительность построения и оптимизировать специальные возможности для удобочитаемости. По сути, существует множество способов выполнить одну и ту же задачу сборки образа. Общие сведения о том, как формат Доккерфиле влияет на время сборки, а также на создаваемое изображение улучшает процесс автоматизации.

## <a name="optimize-image-size"></a>Оптимизация размера изображения

В зависимости от требований к пространству размер изображения может быть важным фактором при создании изображений контейнера DOCKER. Образы контейнеров перемещаются между реестрами и узлом, экспортируются и импортируются и в конечном счете занимают определенное место. В этом разделе вы узнаете, как минимизировать размер изображения во время процесса сборки для контейнеров Windows.

Дополнительные сведения о Доккерфиле рекомендации можно найти в разделе Советы и [рекомендации по написанию доккерфилес на DOCKER.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Группирование связанных действий

Так как `RUN` каждая инструкция создает новый слой в изображении контейнера, Группировка действий в `RUN` одной инструкции может уменьшить количество слоев в доккерфиле. Хотя минимизация числа слоев может не влиять на размер образа, группирование связанных действий влияет на него, что видно в приведенных ниже примерах.

В этом разделе мы будем сравнивать два примера Доккерфилес, которые выполняют те же действия. Тем не менее, один Доккерфиле имеет одну инструкцию для каждого действия, а другая — связанные с ней действия, сгруппированные вместе.

Следующий несгруппированный пример Доккерфиле загружает Python для Windows, устанавливает его и удаляет скачанный установочный файл после завершения установки. В этом Доккерфиле каждому действию назначается собственная `RUN` инструкция.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

Полученный образ имеет три дополнительных слоя— по одному для каждой инструкции `RUN`.

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Второй пример — это Доккерфиле, который выполняет точную операцию. Однако все связанные действия сгруппированы в рамках одной `RUN` инструкции. Каждый шаг `RUN` инструкции находится на новой строке доккерфиле, а символ "\ \" используется для переноса строк.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Полученное изображение имеет только один дополнительный слой для `RUN` инструкции.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Удаление лишних файлов

Если в Доккерфиле есть файл, например установщик, который вы не хотите использовать после его использования, вы можете удалить его, чтобы уменьшить размер изображения. Это следует делать на том же шаге, где данный файл был скопирован в слой образа. Это предотвратит сохранение файла на уровне изображения нижнего уровня.

В следующем примере Доккерфиле пакеты Python загружаются, выполняются, а затем удаляются. Все это выполняется в рамках одной операции `RUN` и создает всего один слой образа.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Оптимизация скорости сборки

### <a name="multiple-lines"></a>Несколько строк

Вы можете разбить операции на несколько отдельных инструкций, чтобы оптимизировать скорость сборки дока. Несколько `RUN` операций увеличивают эффективность кэширования, так как для каждой `RUN` инструкции создаются индивидуальные слои. Если идентичная инструкция уже выполнялась в другой операции сборки Dock, эта кэшированная операция (слой изображений) используется повторно, что приводит к уменьшению среды выполнения сборки.

В приведенном ниже примере пакеты Apache и Visual Studio, распространяемые вместе, загружаются, устанавливаются и удаляются путем удаления файлов, которые больше не нужны. Это делается с помощью одной `RUN` инструкции. Если какое – либо из этих действий будет Обновлено, все действия будут перезапущены.

```dockerfile
FROM windowsservercore

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

Полученное изображение состоит из двух слоев, одного для образа базовой ОС и из одной `RUN` инструкции, содержащей все операции.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

По сравнению с этим действием можно разделить на три `RUN` команды. В этом случае каждая `RUN` инструкция кэшируется на уровне изображения контейнера, и только те из них, которые были изменены, должны быть повторно запущены в последующих сборках доккерфиле.

```dockerfile
FROM windowsservercore

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

Полученное изображение состоит из четырех слоев. один слой для основного образа ОС и все три `RUN` инструкции. Так как `RUN` каждая инструкция выполнялась в отдельном слое, любые последующие запуски этого доккерфиле или идентичный набор инструкций в другом доккерфиле будут использовать кэшированные слои изображений, уменьшая время сборки.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Способ упорядочения инструкций важен при работе с кэшами изображений, как показано в следующем разделе.

### <a name="ordering-instructions"></a>Инструкции по заказу

Файл Dockerfile обрабатывается сверху вниз, при этом каждая инструкция сравнивается с кэшированными слоями. При обнаружении инструкции без кэшированного слоя она и все последующие инструкции обрабатываются в новых слоях образа контейнера. Поэтому порядок инструкций имеет важное значение. Инструкции, которые останутся постоянными, располагайте ближе к началу файла Dockerfile. Инструкции, которые могут изменяться, располагайте ближе к концу файла Dockerfile. Это снижает вероятность отмены существующего кэша.

В следующих примерах показано, как заказ инструкций Доккерфиле может влиять на эффективность кэширования. Этот простой пример Доккерфиле состоит из четырех пронумерованных папок.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

Получившийся рисунок имеет пять слоев, один — для образа базовой ОС и для каждой `RUN` инструкции.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Следующий Доккерфиле немного изменился, и третья `RUN` инструкция изменилась на новый файл. При запуске сборки Docker для этого файла Dockerfile три первых инструкции, которые идентичны инструкциям з прошлого примера, используют кэшированные слои образа. Тем не менее, поскольку `RUN` Инструкция по изменению не кэшируется, создается новый слой для инструкции Changed и всех последующих инструкций.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

При сравнении идентификаторов изображений нового изображения с тем, что в первом примере этого раздела, вы обратите внимание на то, что первые три слоя сверху вниз являются общими, но четвертые и пятого — уникальные.

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

### <a name="instruction-case"></a>Сценарий для инструкции

В доккерфиле не учитывается регистр, но для обозначения используются прописные буквы. Это улучшает читаемость за счет различий между вызовом инструкции и операцией с инструкцией. В двух приведенных ниже примерах сравниваются прописные и прописные Доккерфиле.

Ниже приведено использование прописных букв в Доккерфиле:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Ниже указаны те же Доккерфиле с прописными буквами.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Обтекание строки

Длинные и сложные операции можно разделить на несколько строк с помощью символа обратной косой черты `\` . Следующий файл Dockerfile устанавливает распространяемый пакет Visual Studio, удаляет файлы установщика и затем создает файл конфигурации. Все эти три операции указаны на одной строке.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

Команду можно разбить на обратные косые черты таким образом, чтобы каждая операция из `RUN` одной инструкции была указана в отдельной строке.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Дальнейшие чтение и ссылки

[Dockerfile в Windows](manage-windows-dockerfile.md)

[Рекомендации по составлению файлов Dockerfile на сайте Docker.com](https://docs.docker.com/engine/reference/builder/)
