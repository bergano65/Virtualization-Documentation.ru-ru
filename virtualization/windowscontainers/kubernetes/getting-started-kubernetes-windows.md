---
title: Kubernetes в Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Windows к кластеру Kubernetes с помощью v 1.14.
keywords: kubernetes, 1,14, Windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c380f5dc10430a94959718a5ce92f311603db733
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910364"
---
# <a name="kubernetes-on-windows"></a>Kubernetes в Windows

На этой странице представлены общие сведения о начале работы с Kubernetes в Windows путем присоединения узлов Windows к кластеру под управлением Linux. С выпуском Kubernetes 1,14 в Windows Server [версии 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)пользователи могут воспользоваться следующими функциями в Kubernetes в Windows:

- **наложение сетей**: использование фланнел в режиме вкслан для настройки сети виртуального оверлея
    - требуется либо Windows Server 2019 с установленным [KB4489899](https://support.microsoft.com/help/4489899) , либо [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Build 18317 +
    - требуется Kubernetes v 1.14 (или выше) с включенным шлюзом компонентов `WinOverlay`
    - требуется Фланнел v 0.11.0 (или более поздняя версия)
- **упрощенное управление сетью**. Используйте фланнел в режиме главного шлюза для автоматического управления маршрутами между узлами.
- **улучшения масштабируемости**: повышение скорости и более надежного запуска контейнеров благодаря [бесvnicным устройствам для контейнеров Windows Server](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716).
- **Изоляция Hyper-v (альфа)** : Управление [изоляцией Hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) с помощью изоляции в режиме ядра для усиления безопасности. Дополнительные сведения см. в [подтипах контейнеров Windows](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types).
    - требуется Kubernetes v 1.10 (или более поздней версии) с включенным шлюзом компонентов `HyperVContainer`.
- **подключаемые модули хранилища**. Используйте [подключаемый модуль хранилища ФЛЕКСВОЛУМЕ](https://github.com/Microsoft/K8s-Storage-Plugins) с поддержкой SMB и iSCSI для контейнеров Windows.

>[!TIP]
>Если вы хотите развернуть кластер в Azure, средство AKS-Engine с открытым кодом упрощает эту задачу. Дополнительные сведения см. в этом пошаговом [руководстве](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md).

## <a name="prerequisites"></a>Предварительные условия

### <a name="plan-ip-addressing-for-your-cluster"></a>Планирование назначения IP-адресов для кластера

<a name="definitions"></a>Так как кластеры Kubernetes предоставляют новые подсети для модулей Pod и служб, важно убедиться, что ни одна из них не конфликтует с другими существующими сетями в вашей среде. Ниже приведены все адресные пространства, которые необходимо освободить для успешного развертывания Kubernetes.

| Диапазон подсети/адреса | Описание | Значение по умолчанию |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Подсеть службы** | Виртуальная подсеть без поддержки маршрутизации, которая используется в модулях Pod для равномерного доступа к службам, не волнует о топологии сети. Она преобразуется в маршрутизируемое адресное пространство или из адресного пространства при выполнении `kube-proxy` на узлах. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Подсеть кластера** |  Это глобальная подсеть, используемая всеми модулями Pod в кластере. Для использования модулей Pod каждому узлу назначается меньшая или 24 подсеть. Оно должно быть достаточно большим для размещения всех модулей Pod, используемых в кластере. Вычисление *минимального* размера подсети: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Пример для кластера с 5 узлами для модулей 100 для каждого узла: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **IP-адрес службы DNS Kubernetes** | IP-адрес службы KUBE-DNS, которая будет использоваться для разрешения DNS & обнаружения службы кластеров. | "10.96.0.10" |

> [!NOTE]
> Существует другая сеть DOCKER (NAT), которая создается по умолчанию при установке DOCKER. Не требуется выполнять Kubernetes в Windows, так как мы назначаем IP-адреса из подсети кластера.

## <a name="what-you-will-accomplish"></a>Цели

В рамках этого руководства вы выполните следующие действия.

> [!div class="checklist"]
> * Создал [главный узел Kubernetes](./creating-a-linux-master.md) .  
> * Выбрано [сетевое решение](./network-topologies.md).  
> * К нему присоединен [Рабочий узел рабочей роли Windows](./joining-windows-workers.md) или [Рабочий узел Linux](./joining-linux-workers.md) .  
> * Развернут [образец ресурса Kubernetes](./deploying-resources.md).  
> * Рассмотрим [распространенные проблемы и ошибки](./common-problems.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом разделе мы говорили о важных предварительных требованиях &, необходимых для успешного развертывания Kubernetes в Windows на сегодняшний день. Продолжайте научиться настраивать Kubernetes Master:

>[!div class="nextstepaction"]
>[Создание главного Kubernetes](./creating-a-linux-master.md)