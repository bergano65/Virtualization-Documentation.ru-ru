---
title: Kubernetes в Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Windows к кластеру Kubernetes с v1.12.
keywords: kubernetes, 1.12, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e43b2ac5b19d16721c1ba0dd1f34e339223bdaf
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178907"
---
# <a name="kubernetes-on-windows"></a>Kubernetes в Windows #
Эта страница служит Обзор для начало работы с Kubernetes в Windows, присоединение узла Windows к кластеру на основе Linux. В выпуске Kubernetes 1.12 в бета-версии Windows Server [версии 1803](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1803#kubernetes) пользователи могут воспользоваться [новейшие возможности](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) в Kubernetes в Windows:

  - **упрощенное управления сетями**: используйте Flannel в режим узел шлюз для управления автоматического маршрут между узлами
  - **улучшения масштабируемости**: хотят раз при запуске контейнера быстрее и надежнее благодаря [без устройства виртуальные сетевые карты для контейнеров Windows Server](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)
  - **Изоляция Hyper-v (альфа)**: координации таких действий [контейнеры hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) с изоляцией в режиме ядра для повышения безопасности ([см. в разделе типы контейнеров Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types))
  - **подключаемые модули хранилища**: использовать [подключаемый модуль FlexVolume хранилища](https://github.com/Microsoft/K8s-Storage-Plugins) с поддержкой iSCSI и SMB для контейнеров Windows

> [!TIP] 
> Если вы хотите развернуть кластер в Azure, используйте средство с открытым исходным кодом ACS-Engine. Доступно [пошаговое руководство](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md).

## <a name="prerequisites"></a>Необходимые условия ##

### <a name="plan-ip-addressing-for-your-cluster"></a>Планирование IP-адресов для кластера ###
<a name="definitions"></a>Кластеры Kubernetes со Представляем новый подсетей для модулей POD и служб, важно убедиться, что ни один из них не исключены конфликты находящиеся с любой существующей сети в вашей среде. Ниже приведены адресными пространствами, которые должны быть освобождение успешного развертывания Kubernetes требуются.

| Подсеть / Address диапазона | Описание | Значение по умолчанию |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Подсеть службы** | Не являющихся чисто виртуальная подсеть является, которую модули POD немаршрутизируемая доступа к службам независимо от топологии сети. Она преобразуется в маршрутизируемое адресное пространство или из адресного пространства при выполнении `kube-proxy` на узлах. | «10.96.0.0/12» |
| <a name="cluster-subnet-def"></a>**Подсеть кластера** |  Это глобальные подсети, используемую все модули POD в кластере. Каждый узлов назначается меньше /24 подсети из этого используйте модулей POD. Он должен быть достаточно большим, чтобы разместить все модули, используемые в кластере. Для расчета *Минимальный* размер подсети: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Пример для 5 узла кластера для 100 модули POD на каждом узле: `(5) + (5 *  100) = 505`.  | «10.244.0.0/16» |
| **IP-адрес службы Kubernetes DNS** | IP-адрес службы «с помощью kube-dns», будут использоваться для обнаружение служб DNS разрешение & кластера. | «10.96.0.10» |
> [!NOTE]
> Существует другой сети Docker (NAT), который создается по умолчанию при установке Docker. Он не требуется для работы Kubernetes в Windows, как мы назначение IP-адреса из подсети кластера вместо.

### <a name="disable-anti-spoofing-protection"></a>Отключить защиту в защиту от подделок ###
> [!Important] 
> Внимательно прочитайте этот раздел, так как она необходима для успешного использования виртуальных машин для развертывания Kubernetes в ОС Windows сегодня любым пользователем.

Убедитесь, спуфинг MAC-адресов и виртуализация включена для узла контейнера Windows виртуальных машин (гостевые ОС). Чтобы добиться этого, необходимо выполните следующую команду от имени администратора на компьютере, где размещаются виртуальные машины (пример, приведенный для Hyper-V):

```powershell
Set-VMProcessor -VMName "<name>" -ExposeVirtualizationExtensions $true 
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```
> [!TIP]
> Если вы используете продуктов на основе VMware в соответствии с потребностями виртуализации, можно найти в Включение [неизбирательный режим](https://kb.vmware.com/s/article/1004099) для требования подмена MAC.

>[!TIP]
> Если вы развертываете Kubernetes на виртуальных машинах Azure IaaS самостоятельно, можно найти в виртуальные машины, которые поддерживают [вложенную виртуализацию](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/) для этого требования.

## <a name="what-you-will-accomplish"></a>Цели ##

В рамках этого руководства вы выполните следующие действия.

> [!div class="checklist"]
> * Создан узел [главного узла Kubernetes](./creating-a-linux-master.md) .  
> * Выбор [решения сети](./network-topologies.md).  
> * Присоединенные к [рабочий узел Windows](./joining-windows-workers.md) или [Linux рабочий узел](./joining-linux-workers.md) к нему.  
> * Развернуть [Пример Kubernetes ресурсов](./deploying-resources.md).  
> * Рассмотрим [распространенные проблемы и ошибки](./common-problems.md).

## <a name="next-steps"></a>Дальнейшие действия ##
В этом разделе мы говорили о важных предварительные условия и предположения, необходимая для успешного сегодня развертывания Kubernetes в Windows. Дальше узнать, как для настройки главного узла Kubernetes:

> [!div class="nextstepaction"]
> [Создание главного узла Kubernetes](./creating-a-linux-master.md)