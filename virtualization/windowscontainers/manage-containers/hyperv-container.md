---
title: Изоляция Hyper-V
description: Объясняется, как изоляция Hyper-V отличается от изолированных контейнеров процесса.
keywords: docker, контейнеры
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 092312848173102bec5a791f2c48fe8166e70d5f
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998331"
---
# <a name="hyper-v-isolation"></a>Изоляция Hyper-V

Технология контейнеров Windows включает два различных уровня изоляции для контейнеров, процессов и изоляции Hyper-V. Оба типа создаются, управляются и работают одинаково. Они также создают и используют одни и те же образы контейнера. Отличие заключается в уровне изоляции контейнера, операционной системы узла и других запущенных на этом узле контейнеров.

**Изоляция процесса** — несколько экземпляров контейнера могут одновременно выполняться на узле с изоляцией, обеспечиваемой с помощью технологий пространства имен, элемента управления ресурсами и изоляции процессов.  Контейнеры имеют один и тот же ядро с узлом, а также друг друга.  Это примерно то же самое, что и контейнеры в среде Linux.

**Изоляция Hyper-V** — несколько экземпляров контейнера могут одновременно выполняться на узле, но каждый контейнер выполняется внутри специальной виртуальной машины. Это обеспечивает изоляцию уровня ядра между контейнером и узлом контейнера.

## <a name="hyper-v-isolation-examples"></a>Примеры изоляции Hyper-V

### <a name="create-container"></a>Создание контейнера

Управление изолированными контейнерами Hyper-V с помощью дока практически одинаково для управления контейнерами Windows Server. Чтобы создать контейнер с подробным стыковочным окном для изоляции Hyper-V `--isolation` , используйте параметр `--isolation=hyperv`для задания.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>Пояснения по изоляции

В этом примере показано различие между возможностями изоляции между Windows Server и Hyper-V.

Здесь изолированный контейнер процесса развертывается, и на нем будет выполняться длительный процесс проверки связи.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
```

С помощью команды `docker top` процесс ping возвращается в том виде, который он имеет внутри контейнера. В этом примере процесс имеет идентификатор 3964.

``` cmd
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

В этом примере также запускается изолированный контейнер Hyper-V с процедурой ping.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
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
