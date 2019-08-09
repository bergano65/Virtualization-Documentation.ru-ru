---
title: Совместимость версий контейнеров Windows
description: Сборка и запуск контейнеров в нескольких версиях Windows
keywords: метаданные, контейнеры, версия
author: taylorb-microsoft
ms.openlocfilehash: 84c78947284e18dac347bc04b1ea5fcd96e3a814
ms.sourcegitcommit: c9062b2c75838fcac64e8cd9bcc75d2f1a324d76
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/09/2019
ms.locfileid: "10008660"
---
# <a name="windows-container-version-compatibility"></a>Совместимость версий контейнера Windows

Обновления годовщины для Windows Server 2016 и Windows 10 (обе версии 14393) были первыми выпусками Windows, которые могут создавать и запускать контейнеры Windows Server. Контейнеры, созданные с помощью этих версий, можно запускать в более поздних выпусках, таких как Windows Server версии 1709, но перед этим необходимо изучить некоторые подробности.

В ходе улучшения функций контейнеров Windows нам пришлось внести некоторые изменения, которые могут повлиять на совместимость. Старые контейнеры будут выполняться одинаково на более новых узлах с [изоляцией Hyper-V](../manage-containers/hyperv-container.md)и будут использовать одну и ту же (более старую) версию ядра. Тем не менее, если вы хотите выполнить контейнер на основе более новой сборки Windows, он может выполняться только в новой сборке узла.

>[!NOTE]
> \ * Windows Server версии 1709 больше не поддерживается. Дополнительные сведения можно найти в разделе [жизненный цикл обслуживания образов](base-image-lifecycle.md).

## <a name="windows-server-version-1903-host-os-compatibility"></a>Совместимость операционной системы Windows Server версии 1903

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server версии 1903|Да|Да|
|WindowsServer2019|Да|Нет|
|Windows Server версии 1803|Да|Нет|
|Windows Server версии 1709 *|Да|Нет|
|Windows Server 2016|Да|Нет|

## <a name="windows-server-2019-host-os-compatibility"></a>Совместимость с ОС Windows Server 2019 Host

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server версии 1903|Нет|Нет|
|WindowsServer2019|Да|Да|
|Windows Server версии 1803|Да|Нет|
|Windows Server версии 1709 *|Да|Нет|
|Windows Server 2016|Да|Нет|

## <a name="windows-server-version-1803-host-os-compatibility"></a>Совместимость операционной системы Windows Server версии 1803

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server версии 1903|Нет|Нет|
|WindowsServer2019|Нет|Нет|
|Windows Server версии 1803|Да|Да|
|Windows Server версии 1709 *|Да|Нет|
|Windows Server 2016|Да|Нет|

## <a name="windows-server-version-1709-host-os-compatibility"></a>Windows Server, версия 1709, совместимость с ОС Host *

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server версии 1903|Нет|Нет|
|WindowsServer2019|Нет|Нет|
|Windows Server версии 1803|Нет|Нет|
|Windows Server версии 1709 *|Да|Да|
|Windows Server 2016|Да|Нет|

## <a name="windows-server-2016-host-os-compatibility"></a>Совместимость с ОС Windows Server 2016 Host

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server 2019, версия 1903|Нет|Нет|
|WindowsServer2019|Нет|Нет|
|Windows Server версии 1803|Нет|Нет|
|Windows Server версии 1709 *|Нет|Нет|
|Windows Server 2016|Да|Да|

## <a name="windows-10-version-1903-host-os-compatibility"></a>Совместимость операционной системы Windows 10 версии 1903

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server версии 1903|Нет|Нет|
|WindowsServer2019|Нет|Нет|
|Windows Server версии 1803|Нет|Нет|
|Windows Server версии 1709 *|Нет|Нет|
|Windows Server 2016|Да|Да|

## <a name="windows-10-version-1809-host-os-compatibility"></a>Совместимость операционной системы Windows 10 версии 1809

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server версии 1903|Нет|Нет|
|WindowsServer2019|Да|Нет|
|Windows Server версии 1803|Да|Нет|
|Windows Server версии 1709 *|Да|Нет|
|Windows Server 2016|Да|Нет|

## <a name="windows-10-version-1803-host-os-compatibility"></a>Совместимость операционной системы Windows 10 версии 1803

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows обслуживает версию 1903|Нет|Нет|
|WindowsServer2019|Нет|Нет|
|Windows Server версии 1803|Да|Нет||
|Windows Server версии 1709 *|Да|Нет|
|Windows Server 2016|Да|Нет|

## <a name="windows-10-fall-creators-update-host-os-compatibility"></a>Windows 10 — создатели обновлений для обеспечения совместимости с ОС узла

|ОС контейнера|Поддержка изоляции Hyper-V|Поддержка изоляции процессов|
|---|:---:|:---:|
|Windows Server версии 1903|Нет|Нет|
|WindowsServer2019|Нет|Нет|
|Windows Server версии 1803|Нет|Нет|
|Windows Server версии 1709 *|Да|Нет|
|Windows Server 2016|Да|Нет|

## <a name="matching-container-host-version-with-container-image-versions"></a>Соответствующая хостная версия контейнера с версиями изображений контейнера

### <a name="windows-server-containers"></a>Контейнеры Windows Server

Поскольку контейнеры Windows Server и базовый узел имеют один и тот же ядро, базовое изображение контейнера должно совпадать с тем, которое является узлом. Если версии различаются, контейнер может начинаться, но полная функция не гарантирована. В операционной системе Windows есть четыре уровня управления версиями: основная, младшая, сборка и редакция. Например, версия 10.0.14393.103 имеет старшую версию 10, младшую версию 0, номер сборки 14393 и номер редакции 103. Номер сборки изменяется только при публикации новых версий ОС, таких как версия 1709, 1803, источник обновлений и т. д. Номер редакции меняется при установке обновлений Windows.

#### <a name="build-number-new-release-of-windows"></a>Номер сборки (новый выпуск Windows)

Заблокировано выполнение контейнеров Windows Server, если номер сборки между узлом контейнера и изображением контейнера не совпадают. Например, если узел контейнера имеет версию 10.0.14393. * (Windows Server 2016), а контейнерный рисунок — версия 10.0.16299. * (Windows Server версии 1709) контейнер не запускается.  

#### <a name="revision-number-patching"></a>Номер редакции (исправление)

Контейнеры Windows Server не заблокируются, когда номера редакции узла контейнера и изображения-контейнера различаются. Например, если узел контейнера имеет версию 10.0.14393.1914 (Windows Server 2016 с KB4051033), а изображение контейнера — версия 10.0.14393.1944 (Windows Server 2016 с KB4053579ами), после этого изображение будет запускаться несмотря на то, что его редакция Номера различаются.

Для узлов и изображений на базе Windows Server 2016 версия изображения контейнера должна совпадать с узлом в поддерживаемой конфигурации. Тем не менее, для узлов или изображений, использующих Windows Server версии 1709 и выше, это правило не применяется, а для изображения узла и контейнера не требуется совпадение исправлений. Мы рекомендуем вам своевременно обновлять системы с помощью последних исправлений и обновлений.

#### <a name="practical-application"></a>Практичное приложение

Пример 1: узел контейнера работает под управлением Windows Server 2016, в котором KB4041691 применен. Любые контейнеры сервера Windows, развернутые на этом узле, должны основываться на базовых изображениях контейнера Version 10.0.14393.1770. Если вы примените KB4053579 к контейнеру узла, необходимо также обновить изображения, чтобы убедиться, что они поддерживаются контейнером узла.

Пример 2: узел контейнера работает под управлением операционной системы Windows Server версии 1709, в которой KB4043961 применен. Любой контейнер сервера Windows, развернутый на этом узле, должен быть основан на базовом образе контейнера Windows Server версии 1709 (10.0.16299), но не должен совпадать с KB узла. Если к узлу применен KB4054517, изображения-контейнеры будут поддерживаться, но мы рекомендуем вам обновить их, чтобы устранить любые потенциальные проблемы безопасности.

#### <a name="querying-version"></a>Запрос версии

Способ 1: введенный в версии 1709 команда cmd и **ver** теперь возвращают сведения о редакции.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Способ 2: запросите следующий раздел реестра: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows Нт\куррентверсион

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

Чтобы узнать, какая версия используется в базовом образе, просмотрите теги в центре Dock или хэш-таблицу изображений, указанную в описании изображения. На странице " [Журнал обновлений" для Windows 10](https://support.microsoft.com/help/12387/windows-10-update-history) перечислены все сборки и редакции, которые были выпущены.

### <a name="hyper-v-isolation-for-containers"></a>Изоляция Hyper-V для контейнеров

Вы можете запускать контейнеры Windows как с изоляцией Hyper-V, так и без нее. Изоляция Hyper-V создает границу безопасности вокруг контейнере с оптимизированной ВМ. В отличие от стандартных контейнеров Windows, использующих ядро между контейнерами и ведущим узлом, каждый изолированный контейнер Hyper-V имеет собственный экземпляр ядра Windows. Это означает, что вы можете использовать различные версии операционной системы на узле и на изображении контейнера (Дополнительные сведения можно найти в приведенной ниже матрице совместимости).  

Для запуска контейнера с изоляцией Hyper-V просто добавьте тег `--isolation=hyperv` в вашу команду docker run.

## <a name="errors-from-mismatched-versions"></a>Ошибки из-за несовпадения версий

Если вы попытаетесь запустить неподдерживаемую комбинацию, появится следующее сообщение об ошибке:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Вы можете устранить эту ошибку тремя способами:

- Перестройте контейнер, исходя из нужной `mcr.microsoft.com/windows/nanoserver` версии или `mcr.microsoft.com/windows/servercore`
- Если узел является более новым, запустите **Прикрепление Dock-Isolation = Hyperv...**
- Попробуйте запустить контейнер на другом узле с одной и той же версией Windows

## <a name="choose-which-container-os-version-to-use"></a>Выбор используемой версии контейнера ОС

>[!NOTE]
>По состоянию на 16 апреля 2019, тег "последние" больше не публикуется и не поддерживается для [изображений контейнера операционной системы Windows](https://hub.docker.com/_/microsoft-windows-base-os-images). Объявите конкретный тег для создания изображений из этих Репос или ссылок на них.

Вам необходимо узнать, какую версию вам нужно использовать для вашего контейнера. Например, если вы хотите, чтобы операционная система Windows Server версии 1809 выглядела как ОС контейнера и вы хотите использовать для нее последние исправления, вы `1809` должны воспользоваться этим тегом при указании нужной версии основных изображений контейнера операционной системы, например:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

Тем не менее, если вы хотите установить конкретный патч для Windows Server версии 1809, вы можете указать номер KB в теге. Например, чтобы получить изображение контейнера базовой операционной системы Nano Server из Windows Server версии 1809, к которому применен KB4493509, необходимо указать следующее:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

Вы также можете указать нужные исправления для схемы, которая использовалась ранее, указав версию ОС в теге.

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Базовые образы ядра сервера, основанные на Windows Server 2019 и Windows Server 2016, являются выпусками [долгосрочного канала обслуживания (лтск)](https://docs.microsoft.com/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) . Если вы хотите, чтобы в качестве версии Windows Server 2019 в качестве операционной системы контейнера вашего сервера имелись самые последние исправления, вы можете указать такие выпуски ЛТСК, как показано ниже.

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Сопоставление версий с помощью Docker Swarm

В настоящее время стыковочный Сварм не имеет встроенной функции для соответствия версии Windows, используемой контейнером, с узлом, имеющим такую же версию. Если вы обновите службу для использования более нового контейнера, она будет выполнена успешно.

Если вы хотите использовать несколько версий Windows в течение длительного промежутка времени, можно воспользоваться одним из двух способов: настройте компьютеры Windows так, чтобы всегда использовалась изоляция Hyper-V, или используйте ограничения меток.

### <a name="finding-a-service-that-wont-start"></a>Поиск службы, которая не запускается

Если служба не запускается, вы увидите, что она `MODE` `replicated` `REPLICAS` будет заблокирована на 0. Чтобы узнать, является ли проблема версией ОС, выполните следующие команды:

Чтобы найти имя службы, запустите **службу Dock Ls** .

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Запустите **службу Dock PS (имя службы)** , чтобы получить состояние и последние попытки:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Если вы видите `starting container failed: ...`сообщение об ошибке, вы можете увидеть полную ошибку в **службе Dock-No-ОТБР (имя контейнера)**:

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Такое же сообщение об ошибке `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Исправление: обновите службу, чтобы использовать соответствующую версию

При работе с Docker Swarm следует учитывать два момента. В том случае, если у вас есть файл создания и служба, использующая изображение, которое вы не создали, вам нужно будет обновить ссылку соответствующим образом. Пример.

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

Кроме того, вы можете указать, на какое изображение вы указываете, что вы создали (например, Contoso/MyImage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

В этом случае вы должны использовать метод, описанный в разделе [ошибки из несовпадающих версий](#errors-from-mismatched-versions) , чтобы изменить доккерфиле вместо строки создания дока.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Решение: использование изоляции Hyper-V с Docker Swarm

Существует предложение, поддерживающее использование изоляции Hyper-V на отдельных контейнерах, но код пока еще не выполнен. Ход разработки можно отслеживать на портале [GitHub](https://github.com/moby/moby/issues/31616). До завершения работы узлы необходимо настроить так, чтобы они всегда выполнялись с изоляцией Hyper-V.

Для этого необходимо изменить конфигурацию службы Docker, а затем перезапустить модуль Docker.

1. Измените `C:\ProgramData\docker\config\daemon.json`
2. Добавьте строку с `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >Файл daemon. JSON по умолчанию не существует. Если вы столкнулись с такой ситуацией при изучении каталога, вам необходимо создать файл. После этого вы захотите скопировать следующие фрагменты:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Закройте и сохраните файл, а затем перезапустите подписку, выполнив следующие командлеты в PowerShell:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. После перезапуска службы запустите контейнеры. После того как они запущены, вы можете проверить уровень изоляции контейнера, проверив его с помощью следующего командлета:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Функция вернет значение "process" или "hyperv". Если вы изменили файл daemon.json, как описано выше, вы должны получить последнее значение ("hyperv").

### <a name="mitigation---use-labels-and-constraints"></a>Решение: использование меток и ограничений

Ниже приведены инструкции по использованию меток и ограничений для сопоставления версий.

1. Добавление подписей к каждому узлу.

    На каждом узле добавьте две метки: `OS` и. `OsVersion` Предполагается, что контейнеры выполняются локально, но их также можно настроить для использования удаленного узла.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    Затем вы можете проверить эти данные, выполнив команду " **Dock node** Check", в которой должны отображаться вновь добавленные Метки:

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

    Теперь, когда вы подписаны каждым узлом, вы можете обновлять ограничения, определяющие размещение служб. В следующем примере замените слово "contoso_service" на имя реальной услуги:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    После этого будут применены ограничения для запуска узла.

Чтобы узнать больше о том, как использовать ограничения службы, ознакомьтесь со статьей [Создание ссылки на службу](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Сопоставление версий с помощью Kubernetes

Такая же неполадка, описанная в [соответствующих версиях с помощью Dock Сварм](#matching-versions-using-docker-swarm) , может возникнуть, если в кубернетес запланируются обыкновенные версии. Этот вопрос можно избежать с помощью подобных стратегий.

- Перестройте контейнер на основе той же версии ОС в разработке и производстве. Чтобы узнать, как это сделать, ознакомьтесь с разописанием [выберите, какую версию ОС контейнера использовать](#choose-which-container-os-version-to-use).
- Использование меток узлов и Нодеселекторс, чтобы убедиться, что они запланированы на совместимых узлах, если узлы Windows Server 2016 и Windows Server версии 1709 находятся в одном и том же кластере
- Используйте отдельные кластеры с разными версиями ОС.

### <a name="finding-pods-failed-on-os-mismatch"></a>Ошибка поиска модулей pod при несоответствии версий ОС

В этом случае развертывание включало модуль Pod, запланированный на узле с несовпадающей версией ОС, и не включена изоляция Hyper-V.

Та же ошибка отображается в событиях, которые возвращает команда `kubectl describe pod <podname>`. После нескольких попыток может возникнуть `CrashLoopBackOff`состояние Pod.

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

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Устранение проблем — использование меток узлов и Нодеселектор

Запустите **кубектл Get node** , чтобы получить список всех узлов. После этого вы можете запустить **кубектл описания узла (имя узла)** , чтобы получить дополнительные сведения.

В приведенном ниже примере запущены разные версии двух узлов Windows.

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

Рассмотрим этот пример, чтобы показать, как сопоставить версии.

1. Обратите внимание на каждое имя узла `Kernel Version` и сведения о системе.

    В нашем примере информация будет выглядеть примерно так:

    Имя         | Версия
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Добавьте метку `beta.kubernetes.io/osbuild` для каждого узла. Для операционной системы Windows Server 2016 необходимо, чтобы в этом примере в качестве основных и дополнительных версий (14393,1715) поддерживалась поддержка изоляции Hyper-V. Для Windows Server версии 1709 нужна только основная версия (в данном примере — 16299).

    В этом примере команда для добавления меток выглядит так:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Убедитесь в том, что метки есть в разделе **Get Nodes — Show-Labels ("кубектл")**.

    В этом примере результат будет выглядеть следующим образом:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Добавьте в развертывания узлы выбора. В этом примере мы добавим `nodeSelector` к спецификации контейнера спецификацию `beta.kubernetes.io/os` = Windows и `beta.kubernetes.io/osbuild` = 14393. * или 16299 в соответствии с базовой ОС, используемой контейнером.

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

    Теперь можно запустить модуль pod в обновленном развертывании. Селекторы узлов также отображаются в `kubectl describe pod <podname>`, поэтому вы можете выполнить эту команду, чтобы убедиться, что они были добавлены.

    Ниже приведен пример выходных данных.

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
