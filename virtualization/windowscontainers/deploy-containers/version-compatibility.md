---
title: Совместимость версий контейнеров Windows
description: Сборка и запуск контейнеров в нескольких версиях Windows
keywords: метаданные, контейнеры, версия
author: taylorb-microsoft
ms.openlocfilehash: 64b6b400e12060b86594b90474fdedd73dfef45e
ms.sourcegitcommit: 561eaf94c0c0698d43228ebfcd316a7fcd835a59
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/09/2019
ms.locfileid: "9622789"
---
# <a name="windows-container-version-compatibility"></a>Совместимость версий контейнеров Windows

Windows Server 2016 и Юбилейное обновление Windows 10 (версии 14393) стали первыми выпусками Windows, которые могли выполнять сборку и запускать контейнеры Windows Server. Контейнеры, созданные с помощью этих версий, можно запускать в более поздних выпусках, таких как Windows Server версии 1709, но перед этим необходимо изучить некоторые подробности.

В ходе улучшения функций контейнеров Windows нам пришлось внести некоторые изменения, которые могут повлиять на совместимость. Старые контейнеры будут работать на тех же новых узлах с [изоляцией Hyper-V](../manage-containers/hyperv-container.md)и будет использоваться та же (старая) версия ядра. Тем не менее если вы хотите запустить контейнер в новой сборке Windows, его можно только работать на новой сборке узла.

|Версия ОС контейнера|Версия ОС узла|Совместимость|
|---|---|---|
|Windows Server 2016<br>Сборки: 14393.* |Windows Server 2016<br>Сборки: 14393.* |Поддерживает `process` или `hyperv` изоляции|
|Windows Server 2016<br>Сборки: 14393.* |Windows Server версии 1709<br>Сборки: 16299.* |Поддерживает только `hyperv` изоляции|
|Windows Server 2016<br>Сборки: 14393.* |Windows 10 Fall Creators Update<br>Сборки: 16299.* |Поддерживает только `hyperv` изоляции|
|Windows Server 2016<br>Сборки: 14393.* |Windows Server версии 1803<br>17134.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server 2016<br>Сборки: 14393.* |Windows10 версии1803<br>17134.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server 2016<br>Сборки: 14393.* |WindowsServer2019<br>17763.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server 2016<br>Сборки: 14393.* |Windows10 версии1809<br>17763.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1709<br>Сборки: 16299.* |Windows Server 2016<br>Сборки: 14393.* |Не поддерживается|
|Windows Server версии 1709<br>Сборки: 16299.* |Windows Server версии 1709<br>Сборки: 16299.* |Поддерживает `process` или `hyperv` изоляции|
|Windows Server версии 1709<br>Сборки: 16299.* |Windows 10 Fall Creators Update<br>Сборки: 16299.* |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1709<br>Сборки: 16299.* |Windows Server версии 1803<br>17134.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1709<br>Сборки: 16299.* |Windows10 версии1803<br>17134.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1709<br>Сборки: 16299.* |WindowsServer2019<br>17763.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1709<br>Сборки: 16299.* |Windows10 версии1809<br>17763.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1803<br>17134.* сборки |Windows Server 2016<br>Сборки: 14393.* |Не поддерживается|
|Windows Server версии 1803<br>17134.* сборки |Windows Server версии 1709<br>Сборки: 16299.* |Не поддерживается.|
|Windows Server версии 1803<br>17134.* сборки |Windows 10 Fall Creators Update<br>Сборки: 16299.* |Не поддерживается.|
|Windows Server версии 1803<br>17134.* сборки |Windows Server версии 1803<br>17134.* сборки |Поддерживает `process` или `hyperv` изоляции|
|Windows Server версии 1803<br>17134.* сборки |Windows10 версии1803<br>17134.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1803<br>17134.* сборки |WindowsServer2019<br>17763.* сборки |Поддерживает только `hyperv` изоляции|
|Windows Server версии 1803<br>17134.* сборки |Windows10 версии1809<br>17763.* сборки |Поддерживает только `hyperv` изоляции|
|WindowsServer2019<br>17763.* сборки |Windows Server 2016<br>Сборки: 14393.* |Не поддерживается|
|WindowsServer2019<br>17763.* сборки |Windows Server версии 1709<br>Сборки: 16299.* |Не поддерживается.
|WindowsServer2019<br>17763.* сборки |Windows 10 Fall Creators Update<br>Сборки: 16299.* |Не поддерживается.|
|WindowsServer2019<br>17763.* сборки |Windows Server версии 1803<br>17134.* сборки |Не поддерживается|
|WindowsServer2019<br>17763.* сборки |Windows10 версии1803<br>17134.* сборки |Не поддерживается|
|WindowsServer2019<br>17763.* сборки |WindowsServer2019<br>17763.* сборки |Поддерживает `process` или `hyperv` изоляции|
|WindowsServer2019<br>17763.* сборки |Windows10 версии1809<br>17763.* сборки |Поддерживает только `hyperv` изоляции|

## <a name="matching-container-host-version-with-container-image-versions"></a>Сопоставление версии узла контейнера с версиями образа контейнера

### <a name="windows-server-containers"></a>Контейнеры Windows Server

Так как контейнеры Windows Server и базовые узлы имеют одно ядро, базовые образы контейнера должны совпадать узла. Если версии не совпадают, контейнер может запуститься, но работа всех возможностей не гарантируется. Операционная система Windows имеет четыре уровня указания: основной, вспомогательный номера, сборки и номер редакции. Например версии: 10.0.14393.103 бы основной номер версии 10, дополнительный номер версии 0, номер сборки 14393 и номер версии 103. Номер сборки меняется только в том случае, когда новые версии операционной системы публикации, например версии 1709, 1803, Fall Creators Update и т. д. Номер редакции меняется при установке обновлений Windows.

#### <a name="build-number-new-release-of-windows"></a>Номер сборки (новая версия для Windows)

Контейнеры Windows Server блокируется Если номера сборки узла контейнера и образа контейнера отличаются. Например, когда узла контейнера —: 10.0.14393.* версии (Windows Server 2016) и образ контейнера — версия 10.0.16299.* (Windows Server версии 1709), начнется контейнера.  

#### <a name="revision-number-patching"></a>Номер редакции (исправления)

Контейнеры Windows Server не заблокирован, если номера редакции узла контейнера и образа контейнера отличаются. Например если узел контейнера — версия: 10.0.14393.1914.* (Windows Server 2016 с исправлением KB4051033) и образа контейнера — версия 10.0.14393.1944.* (Windows Server 2016 с исправлением KB4053579), изображение будет по-прежнему запустите даже если номер версии числа не совпадают.

Для узлов под управлением Windows Server 2016 или изображений редакция образа контейнера должна соответствовать узлу с поддерживаемой конфигурацией. Тем не менее для узлов или изображения с помощью Windows Server версии 1709 и более поздней версии, это правило не применяется и образов узла контейнера не обязаны совпадать. Мы рекомендуем оставить устанавливать последние исправления и обновления системы.

#### <a name="practical-application"></a>Практическое применение

Пример 1: Узел контейнера работает под управлением Windows Server 2016 с исправлением KB4041691. Все контейнеры Windows Server, развернутые на этом узле должны быть основаны на базовых образах версии 10.0.14393.1770. Если вы применяете KB4053579 для контейнера узла, необходимо также обновить изображения, чтобы убедиться, что узел контейнера поддерживает их.

Пример 2: Узел контейнера работает под управлением Windows Server версии 1709 с исправлением kb4043961. Все контейнеры Windows Server, развернутые на этом узле, должны быть основаны на Windows Server версии 1709 (10.0.16299) базовый образ контейнера, но не должны соответствовать KB узла. Если KB4054517 применяется к узлу, образы контейнеров по-прежнему будет поддерживаться, но мы рекомендуем обновить их для устранить все потенциальные проблемы безопасности.

#### <a name="querying-version"></a>Запрос версии

Метод 1: Появились в версии 1709, появились Командная строка и **ver** команда теперь возвращают сведений о версиях.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Метод 2: Запросите следующий раздел реестра: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

Пример.

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Проверить, какая версия использует ваше базового образа, просмотрите теги в центре Docker или хэш-таблицу образа в описании образа. На странице [Журнал обновлений Windows 10](https://support.microsoft.com/help/12387/windows-10-update-history) перечислены информацию о датах выпусках всех сборок и редакции.

### <a name="hyper-v-isolation-for-containers"></a>Изоляция Hyper-V для контейнеров

Контейнеры Windows можно запускать изоляцией Hyper-V и без нее. Изоляция Hyper-V создает границу безопасности вокруг контейнере с оптимизированной ВМ. В отличие от стандартных контейнеров Windows, которые используют контейнеров и узла каждый изолированный контейнер Hyper-V использует собственный экземпляр ядра Windows. Это означает, что могут быть разные версии операционной системы в узле контейнера и образа (Дополнительные сведения см. в разделе следующая Матрица совместимости).  

Для запуска контейнера с изоляцией Hyper-V просто добавьте тег `--isolation=hyperv` в вашу команду docker run.

## <a name="errors-from-mismatched-versions"></a>Ошибки из-за несовпадения версий

При попытке использования неподдерживаемого сочетания, вы получите следующую ошибку:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Вы можете устранить эту ошибку тремя способами:

- Заново создайте контейнер в зависимости от правильная версия `mcr.microsoft.com/windows/nanoserver` или `mcr.microsoft.com/windows/servercore`
- Если узел более новой версии, запустите **"run" docker--изоляции = … hyperv**
- Попробуйте запустить контейнер на другом узле с такой же версией Windows

## <a name="choose-which-container-os-version-to-use"></a>Выбор версии ОС контейнера

>[!NOTE]
>Начиная с 16 апреля 2019 г. тег» Latest "больше не публикации и не поддерживается для [Windows базовые образы ОС контейнера](https://hub.docker.com/_/microsoft-windows-base-os-images). Следует объявите конкретного тега при получении или ссылки на изображения из этих репозитории.

Необходимо знать, какая версия, необходимо использовать для контейнера. Например, если вы хотите Windows Server версия 1809 как контейнер ОС и хотите получить последние исправления для нее, следует использовать тег `1809` после указания, какую версию образов требуется, следующим образом:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

Тем не менее если вам требуется определенное исправление Windows Server версия 1809, можно указать номер статьи базы Знаний в теге. Например, чтобы получить базовый образ контейнера ОС Nano Server из Windows Server версия 1809 с KB4493509, примененным к ней, необходимо задать его следующим образом:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

Вы также можете указать точные исправления, необходимые в схеме, которую мы использовали ранее, задавая версию ОС в теге:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Базовые образы Server Core, на основе Windows Server 2019 и Windows Server 2016, освобождает [Long-Term Servicing Channel (LTSC)](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) . Если для экземпляра требуется использовать в качестве контейнера образ Server Core операционной системы Windows Server 2019 и хотите получить последние исправления для него, вы можете указать LTSC выпуски следующим образом:

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Сопоставление версий с помощью Docker Swarm

Мелких объектов docker в настоящее время не имеет встроенный способ сопоставления версии Windows контейнера используется на узел с той же версии. Если вы обновите службу для использования нового контейнера, она будет успешно запущена.

Если вам необходимо выполнять несколько версий Windows на продолжительный период времени, существует два подхода можно предпринять: либо настроить узлы Windows всегда использовали изоляцию Hyper-V или ограничения меток.

### <a name="finding-a-service-that-wont-start"></a>Поиск службы, которая не запускается

Если служба не запускается, вы увидите, что `MODE` — `replicated` , но `REPLICAS` будет равно 0. Чтобы узнать, ли проблема версией ОС, выполните следующие команды:

Запустите **ls службы docker** , чтобы найти имя службы:

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Запустите **службу docker ps (имя)** для получения состояния и последних попыток:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Если вы видите `starting container failed: ...`, вы видите полного описания ошибки с **docker ps службы — нет ОТБР (имя контейнера)**:

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Это та же ошибка как `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Исправление: обновите службу, чтобы использовать соответствующую версию

При работе с Docker Swarm следует учитывать два момента. В случае, когда у вас есть файл compose, содержащий службу, которая использует образ, который не был создан необходимо изменить ссылки соответствующим образом. Пример.

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

Вторая Если вы изображения — это уведомление, вы ссылаетесь, создан (например, contoso/myimage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

В этом случае следует использовать метод, описанный в [ошибки из-за несовпадения версий](#errors-from-mismatched-versions) для изменения этого файла dockerfile вместо строки docker-compose.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Решение: использование изоляции Hyper-V с Docker Swarm

Существует предложение для поддержки использования изоляции Hyper-V на основе каждого контейнера, но еще не готов, код. Ход разработки можно отслеживать на портале [GitHub](https://github.com/moby/moby/issues/31616). До завершения работы узлы необходимо настроить так, чтобы они всегда выполнялись с изоляцией Hyper-V.

Для этого необходимо изменить конфигурацию службы Docker, а затем перезапустить модуль Docker.

1. Измените `C:\ProgramData\docker\config\daemon.json`
2. Добавьте строку с `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >Файл daemon.json по умолчанию не существует. Если вы столкнулись с такой ситуацией при изучении каталога, вам необходимо создать файл. Затем необходимо скопировать в следующем примере:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Закройте и сохраните файл, а затем перезапустите модуль docker, выполнив указанные ниже командлеты в PowerShell:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. После того, как перезапуска службы запустите контейнеры. Они используют, вы можете проверить уровень изоляции контейнера, изучив его с помощью следующего командлета:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Функция вернет значение "process" или "hyperv". Если вы изменили файл daemon.json, как описано выше, вы должны получить последнее значение ("hyperv").

### <a name="mitigation---use-labels-and-constraints"></a>Решение: использование меток и ограничений

Вот как использовать меток и ограничений в соответствии с версиями.

1. Добавление метки для каждого узла.

    На каждом узле добавьте две метки: `OS` и `OsVersion`. Предполагается, что контейнеры выполняются локально, но их также можно настроить для использования удаленного узла.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    После этого вы сможете использовать, выполнив команду **Проверить узел docker** , что следует отображать только что добавленный метки:

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. Добавление ограничения службы.

    Теперь, когда помеченные каждого узла, вы можете обновить ограничения, которые определяют размещение служб. В следующем примере замените «contoso_service» на имя вашей службы:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    После этого будут применены ограничения для запуска узла.

Дополнительные сведения об использовании ограничений службы, ознакомьтесь с [Служба создать ссылку](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Сопоставление версий с помощью Kubernetes

Та же проблема, описанные в [соответствующие версий с помощью режима мелких объектов Docker](#matching-versions-using-docker-swarm) может произойти, если модули POD запланированы для выполнения в Kubernetes. Эту проблему можно избежать с помощью аналогичных методов.

- Заново создайте контейнер в зависимости от той же версии операционной системы в разработки и производственной среде. Чтобы узнать, как, см. в разделе [Выбор какие версии ОС контейнера](#choose-which-container-os-version-to-use).
- Используйте метки узла и nodeSelectors убедитесь, что модули POD запланированы совместимым узлам, если узлы Windows Server 2016 и Windows Server 1709 версии находятся в одном кластере
- Используйте отдельные кластеры с разными версиями ОС.

### <a name="finding-pods-failed-on-os-mismatch"></a>Ошибка поиска модулей pod при несоответствии версий ОС

В этом случае развертывание включены pod, который был назначен узлу с несовпадающей версией ОС и без изоляции Hyper-V.

Та же ошибка отображается в событиях, которые возвращает команда `kubectl describe pod <podname>`. После нескольких попыток pod, вероятно, будет в состоянии `CrashLoopBackOff`.

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Решение: использование меток узлов и nodeSelector

Запустите **kubectl получить узел** для получения списка всех узлов. После этого можно запустить **kubectl описания узла (имя узла)** для получения дополнительных сведений.

В следующем примере два узла Windows работают с различными версиями:

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

Мы будем использовать этот пример для отображения как в соответствии с версии:

1. Запишите имя каждого узла и `Kernel Version` из сведений о системе.

    В нашем примере информация будет выглядеть следующим образом:

    Имя         | Версия
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Добавьте метку `beta.kubernetes.io/osbuild` для каждого узла. Windows Server 2016 требуется поддержка без изоляции Hyper-V основной и дополнительный номер версии (14393.1715 в этом примере). Windows Server версии 1709 нужно только основной версии (16299 в этом примере) совпадают.

    В этом примере команда для добавления подписи выглядит следующим образом:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Проверьте существовании меток, выполнив **kubectl получить узлы--Показывать подписи**.

    В этом примере выходных данных будет выглядеть следующим образом:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Добавьте селекторы узла в развертывания. В этом примере мы добавим `nodeSelector` в спецификации контейнера с `beta.kubernetes.io/os` = windows и `beta.kubernetes.io/osbuild` = 14393.* или 16299 в соответствии с базовой ОС, используемой контейнером.

    Ниже приведен полный пример для запуска контейнера, созданного для Windows Server 2016:

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    Теперь можно запустить модуль pod в обновленном развертывании. Селекторы узла также отображаются в `kubectl describe pod <podname>`, поэтому можно запустить эту команду, чтобы проверить их добавление.

    Выходные данные для нашего примера выглядит следующим образом:

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
