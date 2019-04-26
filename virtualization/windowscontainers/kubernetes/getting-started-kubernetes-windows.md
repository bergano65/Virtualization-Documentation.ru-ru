---
title: Kubernetes в Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Windows к кластеру Kubernetes с v1.13.
keywords: kubernetes, 1.13, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 7c3a0111b3d19ae1b513a84665f870bba24ae33d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576993"
---
# <a name="kubernetes-on-windows"></a>Kubernetes в Windows

Эта страница служит Обзор для Приступая к работе с Kubernetes в Windows, присоединение узла Windows к кластеру на основе Linux. С выпуском Kubernetes 1.14 в Windows Server [версия 1809](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)пользователи могут воспользоваться следующие функции в Kubernetes в Windows:

- **наложение сеть**: использование Flannel в режиме инкапсуляция для настройки сети виртуальный слой
    - требуется либо Windows Server 2019 с [KB4489899](https://support.microsoft.com/en-us/help/4489899) установлена или [Windows Server vNext предварительные](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) сборки 18317 +
    - требуется Kubernetes v1.14 (или выше) с помощью `WinOverlay` включена функция шлюза
    - требуется Flannel v0.11.0 (или выше)
- **упрощенное управления сетями**: используйте Flannel в режим узел шлюз управления автоматического маршрут между узлами.
- **улучшения масштабируемости**: хотят время запуска контейнера быстрее и надежнее благодаря [без устройства виртуальные сетевые карты для контейнеров Windows Server](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/).
- **Изоляция Hyper-V (альфа)**: планировать [изоляции Hyper-V](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) с изоляцией в режиме ядра для повышения безопасности. Дополнительные сведения, [типы контейнеров Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types).
    - требуется Kubernetes v1.10 (или выше) с помощью `HyperVContainer` шлюз функция включена.
- **подключаемые модули хранилища**: использовать [подключаемый модуль FlexVolume хранилища](https://github.com/Microsoft/K8s-Storage-Plugins) с поддержкой iSCSI и SMB для контейнеров Windows.

>[!TIP]
>Если вы хотите развернуть кластер в Azure, средство с открытым исходным кодом AKS модуля упрощает этот процесс. Дополнительные сведения см. в разделе наших пошаговое [пошагового руководства](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md).

## <a name="prerequisites"></a>Что вам понадобится

### <a name="plan-ip-addressing-for-your-cluster"></a>Планирование IP-адресов для кластера

<a name="definitions"></a>Кластеры Kubernetes представляться новых подсетей для модулей POD и служб, важно убедиться, что ни один из них не исключены конфликты находящиеся с помощью других существующих сетей в вашей среде. Ниже приведены адресными пространствами, которые должны быть освобождение успешного развертывания Kubernetes требуются.

| Подсеть / Address диапазона | Описание | Значение по умолчанию |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Подсеть службы** | Не являющихся чисто виртуальная подсеть, используемую модули немаршрутизируемая доступа к службам независимо от топологии сети. Она преобразуется в маршрутизируемое адресное пространство или из адресного пространства при выполнении `kube-proxy` на узлах. | «10.96.0.0/12» |
| <a name="cluster-subnet-def"></a>**Подсеть кластера** |  Это глобальные подсети, используемую все модули POD в кластере. Каждый узлов назначается меньше /24 подсети из этого используйте модулей POD. Он должен быть достаточно большим, чтобы разместить все модули, используемые в кластере. Чтобы вычислить *Минимальный* размер подсети: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Пример 5 узла кластера для 100 модули POD на каждом узле: `(5) + (5 *  100) = 505`.  | «10.244.0.0/16» |
| **IP-адрес службы Kubernetes DNS** | IP-адрес службы «помощью kube-dns», будут использоваться для обнаружение служб DNS разрешение & кластера. | «10.96.0.10» |

> [!NOTE]
> Существует другой сети Docker (NAT), который создается по умолчанию при установке Docker. Он не требуется для работы Kubernetes в Windows, как мы назначение IP-адреса из подсети кластера вместо.

## <a name="what-you-will-accomplish"></a>Цели

В рамках этого руководства вы выполните следующие действия.

> [!div class="checklist"]
> * Создать узел [главного узла Kubernetes](./creating-a-linux-master.md) .  
> * Выбор [решения сети](./network-topologies.md).  
> * Присоединенные к [рабочий узел Windows](./joining-windows-workers.md) или [Linux рабочий узел](./joining-linux-workers.md) к нему.  
> * Развернуть [Пример Kubernetes ресурсов](./deploying-resources.md).  
> * Рассмотрим [распространенные проблемы и ошибки](./common-problems.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом разделе мы говорили о важные предварительные условия & допущений, необходимые для успешного сегодня развертывания Kubernetes в Windows. Дальше узнать, как настроить главного узла Kubernetes:

>[!div class="nextstepaction"]
>[Создание главного узла Kubernetes](./creating-a-linux-master.md)