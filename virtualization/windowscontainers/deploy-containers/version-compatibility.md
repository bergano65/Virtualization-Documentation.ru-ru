---
title: Совместимость версий контейнеров Windows
description: Сборка и запуск контейнеров в нескольких версиях Windows
keywords: метаданные, контейнеры, версия
author: taylorb-microsoft
ms.openlocfilehash: 1f068cd011b2172e75c240d566473ccab25d984a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910484"
---
# <a name="windows-container-version-compatibility"></a>Совместимость версий контейнера Windows

Windows Server 2016 и юбилейное обновление Windows 10 (обе версии 14393) были первыми выпусками Windows, которые могли создавать и запускать контейнеры Windows Server. Контейнеры, созданные с помощью этих версий, могут работать в более новых выпусках, но перед началом работы необходимо знать несколько моментов.

В ходе улучшения функций контейнеров Windows нам пришлось внести некоторые изменения, которые могут повлиять на совместимость. Более старые контейнеры будут выполняться на новых узлах с [изоляцией Hyper-V](../manage-containers/hyperv-container.md)и будут использовать ту же (более старую) версию ядра. Однако если вы хотите запустить контейнер на основе более новой сборки Windows, он может работать только в более новой сборке узла.

## <a name="windows-server-host-os-compatibility"></a>Совместимость ОС узла Windows Server

<!-- start tab view -->
# <a name="windows-server-version-1909tabwindows-server-1909"></a>[Windows Server, версия 1909](#tab/windows-server-1909)

|Версия ОС контейнера для базового образа|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server, версия 1909|&#10004;|&#10004;|
|Windows Server версии 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-version-1903tabwindows-server-1903"></a>[Windows Server, версия 1903](#tab/windows-server-1903)

|Версия ОС контейнера для базового образа|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server, версия 1909|&#10060;|&#10060;|
|Windows Server версии 1903|&#10004;|&#10004;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-2019tabwindows-server-2019"></a>[Windows Server 2019](#tab/windows-server-2019)

|Версия ОС контейнера для базового образа|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server, версия 1909|&#10060;|&#10060;|
|Windows Server версии 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10004;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-2016tabwindows-server-2016"></a>[Windows Server 2016](#tab/windows-server-2016)

|Версия ОС контейнера для базового образа|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server, версия 1909|&#10060;|&#10060;|
|Windows Server версии 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10060;|&#10060;|
|Windows Server 2016|&#10004;|&#10004;|

---
<!-- stop tab view -->

## <a name="windows-10-host-os-compatibility"></a>Совместимость ОС узла Windows 10

<!-- start tab view -->

# <a name="windows-10-version-1909tabwindows-10-1909"></a>[Windows 10, версия 1909](#tab/windows-10-1909)

|Версия ОС контейнера для базового образа|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server, версия 1909|&#10004;|&#10060;|
|Windows Server версии 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-10-version-1903tabwindows-10-1903"></a>[Windows 10, версия 1903](#tab/windows-10-1903)

|Версия ОС контейнера для базового образа|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server, версия 1909|&#10060;|&#10060;|
|Windows Server версии 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-10-version-1809tabwindows-10-1809"></a>[Windows 10, версия 1809](#tab/windows-10-1809)

|Версия ОС контейнера для базового образа|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server, версия 1909|&#10060;|&#10060;|
|Windows Server версии 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

---
<!-- stop tab view -->

## <a name="matching-container-host-version-with-container-image-versions"></a>Соответствующая версия узла контейнера с версиями образа контейнера

### <a name="windows-server-containers"></a>Контейнеры Windows Server

Так как контейнеры Windows Server и базовый узел совместно используют один ядро, версия ОС для базового образа контейнера должна совпадать с версией узла. Если версии различаются, контейнер может запуститься, но полная функциональная работа не гарантируется. Операционная система Windows имеет четыре уровня управления версиями: основная, дополнительная, сборка и редакция. Например, версия 10.0.14393.103 будет иметь основную версию 10, дополнительную версию 0, номер сборки 14393 и номер редакции 103. Номер сборки изменяется только при публикации новых версий ОС, таких как версия 1709, 1903 и т. д. Номер редакции меняется при установке обновлений Windows.

#### <a name="build-number-new-release-of-windows"></a>Номер сборки (новый выпуск Windows)

Запуск контейнеров Windows Server заблокирован, если номер сборки между узлом контейнера и образом контейнера отличается. Например, если узел контейнера имеет версию 10.0.14393. * (Windows Server 2016) и образ контейнера имеет версию 10.0.16299. * (Windows Server версии 1709) контейнер не запускается.  

#### <a name="revision-number-patching"></a>Номер редакции (установка исправлений)

Запуск контейнеров Windows Server не блокируется, если номера редакции узла контейнера и образа контейнера отличаются. Например, если узел контейнера имеет версию 10.0.14393.1914 (Windows Server 2016 с применением KB4051033), а образ контейнера имеет версию 10.0.14393.1944 (Windows Server 2016 с KB4053579), то образ будет запущен, даже если его редакция числа различаются.

Для узлов или образов на базе Windows Server 2016 версия образа контейнера должна соответствовать узлу, поддерживаемому конфигурацией. Однако для узлов или образов, использующих Windows Server версии 1709 и выше, это правило не применяется, и образ узла и контейнера не должен иметь соответствующих редакций. Мы рекомендуем обновлять системы с последними исправлениями и обновлениями.

#### <a name="practical-application"></a>Практичное приложение

Пример 1. узел контейнера работает под Windows Server 2016 с примененным KB4041691. Все контейнеры Windows Server, развернутые на этом узле, должны основываться на базовых образах 10.0.14393.1770 в контейнере версии. При применении KB4053579 к контейнеру узлов необходимо также обновить образы, чтобы убедиться, что контейнер узла поддерживает их.

Пример 2. узел контейнера работает под управлением Windows Server версии 1709 с примененным KB4043961. Все контейнеры Windows Server, развернутые на этом узле, должны быть основаны на базовом образе контейнера Windows Server версии 1709 (10.0.16299), но не должны совпадать с базой знаний узла. Если к узлу применяется KB4054517, образы контейнеров по-прежнему будут поддерживаться, но рекомендуется обновить их для решения любых потенциальных проблем безопасности.

#### <a name="querying-version"></a>Запрос версии

Метод 1. представленный в версии 1709 команда cmd Prompt и **ver** теперь возвращает сведения о редакции.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Метод 2. запрос следующего раздела реестра: HKEY_LOCAL_MACHINE \Софтваре\микрософт\виндовс Нт\куррентверсион

Например:

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Чтобы проверить версию, используемую базовым образом, просмотрите теги в центре DOCKER или хэш-таблицу образа, приведенную в описании образа. При выпуске каждой сборки и редакции отображается страница [Журнал обновлений Windows 10](https://support.microsoft.com/help/12387/windows-10-update-history) .

### <a name="hyper-v-isolation-for-containers"></a>Изоляция Hyper-V для контейнеров

Вы можете запускать контейнеры Windows с изоляцией Hyper-V или без нее. Изоляция Hyper-V создает границу безопасности вокруг контейнере с оптимизированной ВМ. В отличие от стандартных контейнеров Windows, которые совместно используют ядро между контейнерами и узлом, каждый изолированный контейнер Hyper-V имеет собственный экземпляр ядра Windows. Это означает, что в узле контейнера и на образе можно использовать разные версии ОС (Дополнительные сведения см. в следующей матрице совместимости).  

Для запуска контейнера с изоляцией Hyper-V просто добавьте тег `--isolation=hyperv` в вашу команду docker run.

## <a name="errors-from-mismatched-versions"></a>Ошибки из-за несовпадения версий

При попытке выполнить неподдерживаемую комбинацию возникнет следующая ошибка:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Устранить эту ошибку можно тремя способами:

- Перестройте контейнер на основе правильной версии `mcr.microsoft.com/windows/nanoserver` или `mcr.microsoft.com/windows/servercore`
- Если узел новее, выполните команду **DOCKER Run--Isolation = Hyperv...**
- Попробуйте запустить контейнер на другом узле с той же версией Windows

## <a name="choose-which-container-os-version-to-use"></a>Выберите используемую версию ОС контейнера

>[!NOTE]
>По состоянию на 16 апреля 2019, тег "latest" больше не публикуется или не поддерживается для [образов контейнеров базовой ОС Windows](https://hub.docker.com/_/microsoft-windows-base-os-images). Объявите конкретный тег при получении изображений из этих репозиториев или ссылок на них.

Необходимо знать, какую версию необходимо использовать для контейнера. Например, если требуется, чтобы в качестве ОС контейнера использовался Windows Server версии 1809 и для нее были установлены последние исправления, следует использовать тег `1809` при указании нужной версии основных образов контейнеров ОС, например:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

Тем не менее, если требуется определенное исправление для Windows Server версии 1809, можно указать номер статьи базы знаний в теге. Например, чтобы получить образ контейнера базовой ОС Nano Server из Windows Server версии 1809 с примененным к нему KB4493509, необходимо указать его следующим образом:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

Можно также указать необходимые исправления для схемы, которую мы использовали ранее, указав версию ОС в теге:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Базовые образы Server Core, основанные на Windows Server 2019 и Windows Server 2016, являются выпусками [долгосрочного канала обслуживания (LTSC)](https://docs.microsoft.com/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) . Если для экземпляра требуется, чтобы Windows Server 2019 была в качестве ОС контейнера образа Server Core, и для нее нужно установить последние исправления, можно указать LTSC выпуски следующим образом:

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Сопоставление версий с помощью Docker Swarm

В настоящее время DOCKER Swarm не имеет встроенного способа сопоставления с версией Windows, которую контейнер использует для размещения с той же версией. Если вы обновите службу для использования более нового контейнера, она будет выполнена успешно.

Если необходимо запустить несколько версий Windows в течение длительного периода времени, можно выполнить два способа: настроить узлы Windows так, чтобы они всегда использовали изоляцию Hyper-V, или использовать ограничения меток.

### <a name="finding-a-service-that-wont-start"></a>Поиск службы, которая не запускается

Если служба не запускается, вы увидите, что `MODE` `replicated`, но `REPLICAS` будет задержана на 0. Чтобы узнать, является ли проблема версией ОС, выполните следующие команды:

Запустите **службу DOCKER Ls** , чтобы найти имя службы:

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Запустите **службу DOCKER PS (Service Name)** , чтобы получить состояние и последние попытки:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Если вы видите `starting container failed: ...`, можно увидеть полную ошибку в **службе DOCKER PS--No-TRUNC (имя контейнера)** :

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Это та же ошибка, что и `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Исправление: обновите службу, чтобы использовать соответствующую версию

При работе с Docker Swarm следует учитывать два момента. Если у вас есть файл создания, в котором есть служба, использующая создаваемый образ, необходимо соответствующим образом обновить ссылку. Например:

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

Кроме того, следует учитывать, что вы указали, что образ, на который вы указываете, создан самостоятельно (например, Contoso/MyImage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

В этом случае следует использовать метод, описанный в разделе [ошибки из несовпадающих версий](#errors-from-mismatched-versions) , чтобы изменить это dockerfile вместо строки DOCKER-компоновки.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Решение: использование изоляции Hyper-V с Docker Swarm

Существует предложение, поддерживающее использование изоляции Hyper-V для каждого контейнера, но код еще не выполнен. Ход разработки можно отслеживать на портале [GitHub](https://github.com/moby/moby/issues/31616). До завершения работы узлы необходимо настроить так, чтобы они всегда выполнялись с изоляцией Hyper-V.

Для этого необходимо изменить конфигурацию службы Docker, а затем перезапустить модуль Docker.

1. Измените файл `C:\ProgramData\docker\config\daemon.json`.
2. Добавление строки с `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >Файл daemon. JSON по умолчанию не существует. Если вы столкнулись с такой ситуацией при изучении каталога, вам необходимо создать файл. Затем нужно скопировать следующие фрагменты:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Закройте и сохраните файл, а затем перезапустите подсистему DOCKER, выполнив следующие командлеты в PowerShell:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. После перезапуска службы запустите контейнеры. После запуска можно проверить уровень изоляции контейнера, проверив контейнер с помощью следующего командлета:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Функция вернет значение "process" или "hyperv". Если вы изменили файл daemon.json, как описано выше, вы должны получить последнее значение ("hyperv").

### <a name="mitigation---use-labels-and-constraints"></a>Решение: использование меток и ограничений

Ниже показано, как использовать метки и ограничения для сопоставления версий.

1. Добавьте метки для каждого узла.

    На каждом узле добавьте две метки: `OS` и `OsVersion`. Предполагается, что контейнеры выполняются локально, но их также можно настроить для использования удаленного узла.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    Затем вы можете проверить их, запустив команду " **проверить узел" DOCKER** , которая должна показывать только что добавленные Метки:

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

    Теперь, после обозначения каждого узла, можно обновить ограничения, определяющие размещение служб. В следующем примере замените "contoso_service" именем реальной службы:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    После этого будут применены ограничения для запуска узла.

Дополнительные сведения об использовании ограничений службы см. в [справочнике по созданию службы](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Сопоставление версий с помощью Kubernetes

Эта же ситуация, описанная в [соответствующих версиях с помощью DOCKER Swarm](#matching-versions-using-docker-swarm) , может произойти при планировании модулей Pod в Kubernetes. Эту ошибку можно избежать с помощью аналогичных стратегий:

- Перестройте контейнер на основе той же версии ОС в разработке и рабочей среде. См. Дополнительные сведения о [выборе используемой версии ОС контейнера](#choose-which-container-os-version-to-use).
- Используйте метки узлов и Нодеселекторс, чтобы убедиться, что модули Pod запланированы на совместимых узлах, если узлы Windows Server 2016 и Windows Server версии 1709 находятся в одном кластере.
- Используйте отдельные кластеры с разными версиями ОС.

### <a name="finding-pods-failed-on-os-mismatch"></a>Ошибка поиска модулей pod при несоответствии версий ОС

В этом случае развертывание включало модуль Pod, который был запланирован на узле с несоответствующей версией ОС и не включен режим изоляции Hyper-V.

Та же ошибка отображается в событиях, которые возвращает команда `kubectl describe pod <podname>`. После нескольких попыток состояние Pod, вероятно, будет `CrashLoopBackOff`.

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

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Устранение рисков. Использование меток узлов и Нодеселектор

Запустите **kubectl Get node** , чтобы получить список всех узлов. После этого можно запустить **kubectl описать узел (имя узла)** , чтобы получить дополнительные сведения.

В следующем примере на двух узлах Windows выполняются разные версии:

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

Давайте рассмотрим этот пример, чтобы продемонстрировать, как сопоставить версии:

1. Запишите имя каждого узла и `Kernel Version` из сведений о системе.

    В нашем примере сведения будут выглядеть следующим образом:

    Имя         | Версия
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Добавьте метку `beta.kubernetes.io/osbuild` для каждого узла. Для Windows Server 2016 требуется поддержка основной и вспомогательной версий (14393,1715 в этом примере) без изоляции Hyper-V. В Windows Server версии 1709 для сопоставления требуется только основная версия (16299 в этом примере).

    В этом примере команда для добавления меток выглядит следующим образом:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Проверьте наличие меток, выполнив **kubectl Get Nodes--показывать-Labels**.

    В этом примере выходные данные будут выглядеть следующим образом:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Добавьте в развертывания селекторы узлов. В этом примере мы добавим `nodeSelector` в спецификацию контейнера с `beta.kubernetes.io/os` = Windows и `beta.kubernetes.io/osbuild` = 14393. * или 16299 в соответствии с базовой ОС, используемой контейнером.

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

    Теперь можно запустить модуль pod в обновленном развертывании. Селекторы узлов также отображаются в `kubectl describe pod <podname>`, поэтому можно выполнить эту команду, чтобы убедиться, что они были добавлены.

    Результат для нашего примера выглядит следующим образом:

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
