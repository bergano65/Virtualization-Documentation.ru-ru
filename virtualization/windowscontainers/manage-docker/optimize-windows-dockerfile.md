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
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610344"
---
# <a name="optimize-windows-dockerfiles"></a>Оптимизация файлов Dockerfile в Windows

Существует множество способов, чтобы оптимизировать процесс сборки Docker и получаемые образы Docker. В этой статье объясняется, как работает процесс сборки Docker и оптимально создание образов для контейнеров Windows.

## <a name="image-layers-in-docker-build"></a>Слои изображения в сборке Docker

Прежде чем вы можете оптимизировать Docker build, необходимо знать, как сборка работает. В процессе сборки Docker используется файл Dockerfile, а также поочередно выполняются все активные инструкции, каждая в своем собственном временном контейнере. В результате для каждой активной инструкции создается новый слой образа.

Например, в следующем примере используются Dockerfile `windowsservercore` базового образа ОС, установка служб IIS, а затем создает простой веб-сайт.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Можно предположить, что этого файла Dockerfile создает образ с помощью два слоя: один для образа ОС контейнера и второй, включающий IIS и веб-сайта. Тем не менее самого изображения имеет множества слоев, и каждый слой зависит от того, прежде чем его.

Чтобы пояснить, запустим `docker history` команды для образа наш пример внесенные Dockerfile.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Выходные данные показывает, что это изображение имеет четыре уровня: уровень базового и трех дополнительных — по, сопоставленные с каждой инструкции в файле Dockerfile. Нижний слой (в этом примере— `6801d964fda5`) представляет базовый образ ОС. Один уровень вверх — это установка служб IIS. Следующий слой включает новый веб-сайт ит.д.

Файлы Dockerfile можно свести свести к минимуму число слоев в образе, оптимизировать производительность сборки и оптимизировать специальными возможностями удобочитаемость. По сути, существует множество способов выполнить одну и ту же задачу сборки образа. Общие сведения о том, как формат файла Dockerfile влияет на время сборки и изображение, которое создает расширяет возможности автоматизации.

## <a name="optimize-image-size"></a>Оптимизация размера образа

В зависимости от потребностей пространства размер изображения может быть важным фактором при создании образов контейнеров Docker. Образы контейнеров перемещаются между реестрами и узлом, экспортируются и импортируются и в конечном счете занимают определенное место. В этом разделе можно узнать, как свести к минимуму размер образа во время сборки Docker для контейнеров Windows.

Дополнительные сведения о рекомендациях Dockerfile см. в разделе [Советы и рекомендации по составлению файлов Dockerfile на сайте Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Группирование связанных действий

Так как каждый `RUN` инструкции создается новый слой в образе контейнера, группирование действий в одной `RUN` инструкция позволяет сократить число слоев в файле Dockerfile. Хотя минимизация числа слоев может не влиять на размер образа, группирование связанных действий влияет на него, что видно в приведенных ниже примерах.

В этом разделе мы будем Сравните двух файлов Dockerfile, выполните те же действия. Тем не менее один файл Dockerfile содержит одна инструкция каждого действия, а другой было его связанных действий, сгруппировать.

Приведенный ниже без группировки Dockerfile скачивает Python для Windows, Установка и удаляет скачанного файла установки после выполнения установки. В этом файле Dockerfile, каждое действие, которому присваивается собственный `RUN` инструкции.

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

Во втором примере — это файл Dockerfile, выполняющий точное одной операции. Тем не менее, все связанные действия были сгруппированы в один `RUN` инструкции. Каждый шаг в `RUN` инструкция — с новой строки в файле dockerfile, пока "\\" символ используется для переноса строки.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Полученный образ имеет только один дополнительный слой для `RUN` инструкции.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Удаление лишних файлов

Есть ли используется файл в файле Dockerfile, например установщик, ненужные после него, вы можете удалить его, чтобы уменьшить размер образа. Это следует делать на том же шаге, где данный файл был скопирован в слой образа. Это предотвращает сохранение файла в слое образа более низкого уровня.

В следующем примере Dockerfile пакет Python загрузки, выполняются, то удален. Все это выполняется в рамках одной операции `RUN` и создает всего один слой образа.

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

Можно разделить операции на несколько отдельных инструкций оптимизация скорости сборки Docker. Несколько `RUN` операции повышает эффективность кэширования, поскольку отдельные слои создаются для каждой `RUN` инструкции. Если так же, как инструкции уже был выполнен в другой операции сборки Docker, эта кэшированная операция (слой образа) используется повторно, что приводит к снижению среды выполнения сборки Docker.

В следующем примере Visual Studio распространяемые пакеты Apache и загружаются, установки и затем удаляются, удаляя файлы, которые больше не требуются. Все это выполняется с помощью одного `RUN` инструкции. При изменении любого из этих действий будет повторно выполнить все действия.

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

Полученный образ имеет два слоя, один для базового образа ОС и тот, который содержит все операции из одной `RUN` инструкции.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Для сравнения, вот же действия разбить на три `RUN` инструкции. В этом случае каждый `RUN` инструкция кэшируется в слое образа контейнера, а только те, которые имеют измененных нужно выполнить еще раз на последующих Dockerfile сборки.

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

Полученный образ имеет четыре слоя —; один слой для базового образа ОС и каждый из трех `RUN` инструкции. Так как каждый `RUN` инструкция был выполнен в отдельном слое, при последующих запусках этого файла dockerfile или аналогичного набора инструкций из другого Dockerfile будет использоваться кэшированные слои образа, сокращает время сборки.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Как порядок инструкций важно при работе с кэшем изображения, как будет видно в следующем разделе.

### <a name="ordering-instructions"></a>Порядок инструкций

Файл Dockerfile обрабатывается сверху вниз, при этом каждая инструкция сравнивается с кэшированными слоями. При обнаружении инструкции без кэшированного слоя она и все последующие инструкции обрабатываются в новых слоях образа контейнера. Поэтому порядок инструкций имеет важное значение. Инструкции, которые останутся постоянными, располагайте ближе к началу файла Dockerfile. Инструкции, которые могут изменяться, располагайте ближе к концу файла Dockerfile. Это снижает вероятность отмены существующего кэша.

В следующих примерах показано, как порядок инструкций Dockerfile влияет на эффективность кэширования. Этот простой пример файла Dockerfile имеет четыре нумерованных папки.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

Полученный образ имеет пять слоев — один для базового образа ОС и к каждому из `RUN` инструкции.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Это следующий файл Dockerfile теперь немного изменен, с помощью третьего `RUN` инструкция изменения в новый файл. При запуске сборки Docker для этого файла Dockerfile три первых инструкции, которые идентичны инструкциям з прошлого примера, используют кэшированные слои образа. Однако поскольку измененная `RUN` не кэшируются инструкцию, для измененные инструкции и всех последующих инструкций создается новый слой.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

При сравнении идентификаторы изображение нового образа в первом примере в этом разделе, вы заметите, что совместно используются первые три слоя снизу вверх, однако четвертый и пятый уникальны.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Косметическая оптимизация

### <a name="instruction-case"></a>Регистр инструкций

Инструкции Dockerfile не учитывается, но соответствует соглашению заключается в использовании верхний регистр. Это улучшает удобочитаемость, разделяя вызов инструкции и операцию инструкции. Следующие два примера сравнить Dockerfile не с прописных и прописными буквами.

Ниже приведена параметр Dockerfile.

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Ниже приведена же Dockerfile, с помощью верхний регистр.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Перенос строк

Длинные и сложные операции можно разделить на несколько строк, обратная косая черта `\` символов. Следующий файл Dockerfile устанавливает распространяемый пакет Visual Studio, удаляет файлы установщика и затем создает файл конфигурации. Все эти три операции указаны на одной строке.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

Команду можно выделить символа обратной косой черты таким образом, каждая операция из `RUN` инструкция задается в отдельную строку.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Дополнительные материалы и ссылки

[Dockerfile в Windows](manage-windows-dockerfile.md)

[Рекомендации по составлению файлов Dockerfile на сайте Docker.com](https://docs.docker.com/engine/reference/builder/)
