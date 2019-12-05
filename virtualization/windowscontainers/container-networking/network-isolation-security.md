---
title: Сетевые подключения контейнеров Windows
description: Сетевая изоляция и безопасность в контейнерах Windows.
keywords: docker, контейнеры
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: d5081104f1614a91d6441a5e879a439f1df1bf77
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998541"
---
# <a name="network-isolation-and-security"></a>Сетевая изоляция и безопасность

## <a name="isolation-with-network-namespaces"></a>Изоляция с помощью сетевых пространств имен

Каждый контейнер размещается в собственном __пространстве имен сети__. Виртуальная сетевая карта узла управления и сетевой стек узла размещаются в пространстве имен сети по умолчанию. Чтобы обеспечить сетевую изоляцию между контейнерами на одном и том же узле, создается сетевое пространство имен для каждого контейнера и контейнеров Windows Server, в которых установлен сетевой адаптер для контейнера. Для подключения к виртуальному коммутатору контейнеры Windows Server используют виртуальный сетевой адаптер узла. Изоляция Hyper-V использует искусственный сетевой адаптер виртуальной машины (не предоставленный для виртуальной машины служебной программы), чтобы присоединиться к виртуальному коммутатору.

![текст](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>Сетевая безопасность

В зависимости от используемого контейнера и сетевого драйвера, списки управления доступом портов применяются с помощью сочетания брандмауэра Windows и [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/).

### <a name="windows-server-containers"></a>Контейнеры Windows Server

Эти контейнеры используют брандмауэр узла Windows (использующий пространства имен сети) и VFP

* Исходящий трафик по умолчанию: ПОЛНОСТЬЮ РАЗРЕШИТЬ
* Входящий трафик по умолчанию: ПОЛНОСТЬЮ РАЗРЕШИТЬ (TCP, UDP, ICMP, IGMP) незапрошенный сетевой трафик
  * ПОЛНОСТЬЮ ЗАПРЕТИТЬ другой сетевой трафик, поступающий через прочие протоколы

  >[!NOTE]
  >До Windows Server версии 1709 и Windows 10 не обновляются, правило для входящего трафика по умолчанию было отклоняться. Пользователи, которые работают с более старыми выпусками ``docker run -p`` , могут создавать правила для входящих подключений с помощью (Переадресация порта).

### <a name="hyper-v-isolation"></a>Изоляция Hyper-V

Контейнеры, работающие в изоляции Hyper-V, имеют собственный изолированный ядро и, следовательно, выполняют собственный экземпляр брандмауэра Windows со следующей конфигурацией.

* Правило по умолчанию "ПОЛНОСТЬЮ РАЗРЕШИТЬ" в брандмауэре Windows (запущенном в служебной виртуальной машине) и VFP

![текст](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Кубернетес обыкновенных модулей

В [кубернетес Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)сначала создается контейнер инфраструктуры, к которому присоединена конечная точка. Контейнеры, которые принадлежат одному и тому же модулю, включая контейнеры инфраструктуры и рабочих процессов, совместно использовать общее сетевое пространство имен (тот же IP-адрес и пространство портов).

![текст](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Настройка списков управления доступом портов по умолчанию

Если вы хотите изменить списки ACL для порта по умолчанию, ознакомьтесь с документацией по сетевой службе сети (ссылка для добавления в ближайшее время). Вы должны обновить политики в следующих компонентах:

>[!NOTE]
>В режиме прозрачности и режима NAT вы не можете перепрограммировать ACL порта по умолчанию для изоляции Hyper-V. В таблице это отражено пометкой "X".

| Сетевой драйвер | Контейнеры Windows Server | Изоляция Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Прозрачный режим | Брандмауэр Windows | X |
| NAT | Брандмауэр Windows | X |
| L2Bridge | Оба типа | VFP |
| L2Tunnel | Оба типа | VFP |
| Наложение  | Оба типа | VFP |