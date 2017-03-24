---
title: "Оптимизация файлов Dockerfile в Windows"
description: "Оптимизация файлов Dockerfile для контейнеров Windows."
keywords: "docker, контейнеры"
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: 2077f7cf0428e08ce915470ac4cc3b0ccc9c6369
ms.lasthandoff: 01/25/2017

---
# Оптимизация файлов Dockerfile в Windows

Чтобы оптимизировать процесс сборки Docker и получаемые образы Docker, можно использовать несколько методов. Этот документ описывает работу процесса сборки Docker и предлагает несколько методик, позволяющих оптимизировать создание образов с помощью контейнеров Windows.

## Сборка Docker

### Слои образов

Прежде чем изучать оптимизацию сборки Docker, важно понять, как именно эта сборка работает. В процессе сборки Docker используется файл Dockerfile, а также поочередно выполняются все активные инструкции, каждая в своем собственном временном контейнере. В результате для каждой активной инструкции создается новый слой образа. 

Рассмотрим приведенный ниже файл Dockerfile. В этом примере используется базовый образ ОС `windowsservercore`, устанавливаются службы IIS и создается простой веб-сайт.

```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

При таком файле Dockerfile можно предположить, что итоговый образ содержит два слоя: один для образа ОС контейнера и второй, включающий службы IIS и веб-сайт. Однако это не так. Новый образ состоит из множества слоев, каждый из которых зависит от предыдущего. Чтобы показать это в наглядном виде, можно выполнить для нового образа команду `docker history`. При этом становится видно, что образ состоит из четырех слоев: одного базового и трех дополнительных — по одному для каждой инструкции в файле Dockerfile.

```none
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Каждый из этих слоев можно сопоставить с инструкцией из файла Dockerfile. Нижний слой (в этом примере — `6801d964fda5`) представляет базовый образ ОС. Одним слоем выше находится установка служб IIS. Следующий слой включает новый веб-сайт и т. д.

Файлы Dockerfile можно составлять таким образом, чтобы свести к минимуму число слоев в образе, оптимизировать производительность сборки, а также улучшить такие показатели, как удобочитаемость. По сути, существует множество способов выполнить одну и ту же задачу сборки образа. Понимание того, как формат файла Dockerfile влияет на время сборки и итоговый образ, расширяет возможности автоматизации. 

## Оптимизация размера образа

При создании образов контейнеров Docker важное значение может иметь их размер. Образы контейнеров перемещаются между реестрами и узлом, экспортируются и импортируются и в конечном счете занимают определенное место. Доступно несколько разных методик, позволяющих минимизировать размер образа во время сборки Docker. В этом разделе подробно описываются некоторые такие методики, относящиеся к контейнерам Windows. 

Дополнительные сведения о рекомендациях для файлов Dockerfile см. в [советах по составлению файлов Dockerfile на сайте Docker.com]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### Группирование связанных действий

Поскольку каждая инструкция `RUN` создает новый слой в образе контейнера, группирование действий в одной инструкции `RUN` позволяет сократить число слоев. Хотя минимизация числа слоев может не влиять на размер образа, группирование связанных действий влияет на него, что видно в приведенных ниже примерах.

Два следующих примера демонстрируют выполнение одной операции. При этом получаются образы контейнеров с идентичными возможностями, однако используются два разных файла Dockerfile. Итоговые образы также сравниваются.  

В первом примере Python для Windows скачивается, устанавливается и очищается путем удаления скачанного файла установки. Каждое из этих действий выполняется в своей собственной инструкции `RUN`.

```none
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

Полученный образ имеет три дополнительных слоя — по одному для каждой инструкции `RUN`.

```none
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Для сравнения здесь выполняется та же операция, однако все шаги сгруппированы в одну инструкцию `RUN`. Обратите внимание, что каждый шаг в инструкции `RUN` начинается с новой строки в файле Dockerfile, при этом символ '\' используется для переноса строки. 

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Полученный образ имеет один дополнительный слой для инструкции `RUN`.

```none
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### Удаление лишних файлов

Если после использования файл, например установщик, больше не требуется, удалите его, чтобы уменьшить размер образа. Это следует делать на том же шаге, где данный файл был скопирован в слой образа. Эта процедура предотвращает сохранение файла в слое образа более низкого уровня.

В этом примере скачивается и выполняется пакет Python, после чего исполняемый файл удаляется. Все это выполняется в рамках одной операции `RUN` и создает всего один слой образа.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## Оптимизация скорости сборки

### Несколько строк

При оптимизации для скорости сборки Docker бывает выгодно разделить операции на несколько отдельных инструкций. Наличие нескольких операций `RUN` повышает эффективность кэширования. Поскольку отдельные слои создаются для каждой инструкции `RUN`, если аналогичный шаг уже был выполнен в другой операции сборки Docker, эта кэшированная операция (слой образа) используется повторно. В результате уменьшается среда выполнения для сборки Docker.

В следующем примере загружаются и устанавливаются распространяемые пакеты Apache и Visual Studio, после чего ненужные файлы удаляются. Все это выполняется в рамках одной инструкции `RUN`. При изменении любого из этих действий все они перезапускаются.

```none
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredistexe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force
```

Итоговый образ состоит из двух слоев — один для базового образа ОС, второй содержит все операции из одной инструкции `RUN`.

```none
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

В отличие от предыдущего примера здесь те же действия поделены между тремя инструкциями `RUN`. В этом случае каждая инструкция `RUN` кэшируется в слое образа контейнера, а при последующих сборках Dockerfile повторно выполняются только измененные инструкции.

```none
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

Полученный образ имеет четыре слоя — один для базового образа ОС и по одному для каждой инструкции `RUN`. Так как каждая инструкция `RUN` выполняется в отдельном слое, при всех последующих запусках этого файла Dockerfile или аналогичного набора инструкций из другого Dockerfile будет использоваться кэшированный слой образа, что сокращает время сборки. При работе с кэшем образов важен порядок инструкций. Дополнительные сведения см. в следующем разделе этого документа.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### Порядок инструкций

Файл Dockerfile обрабатывается сверху вниз, при этом каждая инструкция сравнивается с кэшированными слоями. При обнаружении инструкции без кэшированного слоя она и все последующие инструкции обрабатываются в новых слоях образа контейнера. Поэтому порядок инструкций имеет важное значение. Инструкции, которые останутся постоянными, располагайте ближе к началу файла Dockerfile. Инструкции, которые могут изменяться, располагайте ближе к концу файла Dockerfile. Это снижает вероятность отмены существующего кэша.

Этот пример демонстрирует, как порядок инструкций в файле Dockerfile влияет на эффективность кэширования. В этом простом файле Dockerfile создаются четыре нумерованных папки.  

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
Полученный образ имеет пять слоев — один для базового образа ОС и по одному для каждой инструкции `RUN`.

```none
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

Файл Docker теперь немного изменен. Обратите внимание, что изменилась третья инструкция `RUN`. При запуске сборки Docker для этого файла Dockerfile три первых инструкции, которые идентичны инструкциям з прошлого примера, используют кэшированные слои образа. Однако поскольку измененная инструкция `RUN` не кэшировалась, для нее и всех последующих инструкций создается новый слой.

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Сравнив идентификатор нового образа и аналогичный идентификатор из прошлого примера, вы увидите, что совместно используются первые три слоя (снизу вверх), однако четвертый и пятый уникальны.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## Косметическая оптимизация

### Регистр инструкций

В инструкциях Dockerfile не учитывается регистр, однако принято использовать верхний регистр. Это улучшает удобочитаемость, разделяя вызов инструкции и операцию инструкции. Эту концепцию поясняют два приведенных ниже примера. 

Нижний регистр:
```none
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
Верхний регистр: 
```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### Перенос строк

Длинные и сложные операции можно разделить на несколько строк с помощью символа обратной косой черты `\`. Следующий файл Dockerfile устанавливает распространяемый пакет Visual Studio, удаляет файлы установщика и затем создает файл конфигурации. Все эти три операции указаны на одной строке.

```none
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
Команду можно переписать, чтобы каждая операция из инструкции `RUN` была указана на отдельной строке. 

```none
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## Дополнительные материалы и справочники

[Dockerfile в Windows] (manage_windows_dockerfile.md)

[Рекомендации по составлению файлов Dockerfile на сайте Docker.com](https://docs.docker.com/engine/reference/builder/)

