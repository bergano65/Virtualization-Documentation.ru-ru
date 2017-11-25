---
title: "Контейнеры Hyper-V"
description: "Развертывание контейнеров Hyper-V."
keywords: "docker, контейнеры"
author: enderb-ms
ms.date: 09/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: ea131dfede51ee36f7dc703511357612430ccca9
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/08/2017
---
# <a name="hyper-v-containers"></a>Контейнеры Hyper-V

**Это предварительное содержимое. Возможны изменения.** 

Технология контейнеров Windows включает два различных типа контейнеров: Windows Server и Hyper-V. Оба типа создаются, управляются и работают одинаково. Они также создают и используют одни и те же образы контейнера. Отличие заключается в уровне изоляции контейнера, операционной системы узла и других запущенных на этом узле контейнеров.

**Контейнеры Windows Server**. Несколько экземпляров контейнеров могут одновременно работать на одном узле, а изоляция обеспечивается с помощью пространств имен, управления ресурсами и технологий изоляции процессов.  Контейнеры Windows Server и узел используют одно и то же ядро.

**Контейнеры Hyper-V**. На узле может быть одновременно запущено несколько экземпляров контейнера, однако каждый контейнер запускается в специальной виртуальной машине. Это обеспечивает изоляцию на уровне ядра между каждым контейнером Hyper-V и узлом контейнера.

## <a name="hyper-v-container"></a>Контейнер Hyper-V

### <a name="create-container"></a>Создание контейнера

Управление контейнерами Hyper-V и Windows Server с помощью Docker почти ничем не отличается. При создании контейнера Hyper-V с помощью Docker используется параметр `--isolation=hyperv`.

```
docker run -it --isolation=hyperv microsoft/nanoserver cmd
```

### <a name="isolation-explanation"></a>Пояснения по изоляции

В этом примере рассматриваются различия возможностей изоляции для контейнеров Windows Server и Hyper-V. 

Здесь развертываются контейнеры Windows Server и производится долгосрочный процесс проверки связи ping.

```
docker run -d microsoft/windowsservercore ping localhost -t
```

С помощью команды `docker top` процесс ping возвращается в том виде, который он имеет внутри контейнера. В этом примере процесс имеет идентификатор 3964.

```
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

На узле контейнера команду `get-process` можно использовать для возврата любых выполняющихся там процессов ping. В этом примере присутствует один такой процесс, идентификатор которого совпадает с идентификатором процесса из контейнера. Из контейнера и с узла виден один и тот же процесс.

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

В отличие от предыдущего примера, здесь запускается контейнер Hyper-V с процессом ping. 

```
docker run -d --isolation=hyperv microsoft/nanoserver ping -t localhost
```

Как и прежде, команда `docker top` позволяет вернуть выполняемые процессы из контейнера.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Однако при поиске процесса на узле контейнера процесс ping отсутствует и выдается ошибка.

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

При этом на узле виден процесс `vmwp`, который является виртуальной машиной, инкапсулирующей запущенный контейнер и защищающей выполняющиеся процессы от операционной системы сервера виртуальных машин.

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
