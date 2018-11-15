---
title: Сетевые топологии
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Сетевые топологии поддерживается в Windows и Linux.
keywords: kubernetes, 1.12, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948063"
---
# <a name="network-solutions"></a>Решения для сети #

После [настройки главном узле Kubernetes](./creating-a-linux-master.md) вы готовы выбрать сетевые решения. Существует несколько способов сделать виртуальной [подсети кластера](./getting-started-kubernetes-windows.md#cluster-subnet-def) маршрутизируемый между узлами. Выберите один из следующих вариантов для Kubernetes в ОС Windows сегодня:

1. Используйте сторонних CNI подключаемый модуль например [Flannel](network-topologies.md#flannel-in-host-gateway-mode) для настройки маршруты для вас.
1. Настройка смарт- [переключения верхнего (ToR)](network-topologies.md#configuring-a-tor-switch) для маршрутизации подсети.

> [!tip]  
> Существует третий сетевые решения для Windows, который использует открыть виртуальный коммутатор (OvS) и откройте виртуальной сети (OVN). Документирование это выходит за рамки данного документа, но вы можете прочитать [этим инструкциям](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) , чтобы настроить его.

## <a name="flannel-in-host-gateway-mode"></a>Flannel в режим узел шлюз

Один из доступных вариантов для сетей Flannel является *режим узел шлюз* (хост gw), который реализует конфигурацию статических маршрутов между подсетями модулей pod на всех узлах.
> [!NOTE]  
> Этот параметр отличается в режим сети *наложения* в Flannel, который использует ИНКАПСУЛЯЦИЮ vxlan и находится в разработке прямо сейчас. Посмотрите это пространство …

### <a name="prepare-kubernetes-master-for-flannel"></a>Подготовка главного узла Kubernetes для Flannel

Некоторые незначительные подготовки рекомендуется от [главного узла Kubernetes](./creating-a-linux-master.md) в наших кластера. Рекомендуется, чтобы при использовании Flannel с мостом трафик IPv4 утилита iptables цепочки. Это можно сделать с помощью следующей команды:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Скачать и настроить Flannel ###
Загрузите последний Flannel манифест:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Существует два аспекта, вам нужно сделать, чтобы включить хост gw сеть на обоих Windows и Linux.

В `net-conf.json` раздел из помощью kube-flannel.yml убедиться, что:
1. Тип используемого серверной части сети задано значение `host-gw` вместо `vxlan`.
2. Задайте подсети кластера (например, «10.244.0.0/16»), как нужный.

После применения 2 действия вашего `net-conf.json` должен выглядеть следующим образом:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Запуск Flannel & проверки ###
Запуск с использованием Flannel:

```bash
kubectl apply -f kube-flannel.yml
```

Затем, так как модули Flannel на основе Linux, исправление для наших Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) для `kube-flannel-ds` DaemonSet только интересующих Linux (при запуске Flannel «flanneld» узла агента процесс в Windows позже при присоединении):

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Через несколько минут вы увидите все модули, что приложение работает, если сеть модуля pod Flannel была развернута.

```bash
kubectl get pods --all-namespaces
```

![текст](media/kube-master.png)

Flannel DaemonSet также должны иметь NodeSelector применения.

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-daemonset.png)
> [!tip]  
> Путать? Вот v0.9.1 полный [Пример помощью kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) Flannel для этих 2 действий, предварительно применяется для подсети кластера по умолчанию `10.244.0.0/16`.

## <a name="configuring-a-tor-switch"></a>Настройка коммутатора верхнего уровня ##
> [!NOTE]
> Этот раздел можно пропустить, если вы выбрали [Flannel как сетевые решения](#flannel-in-host-gateway-mode).
Конфигурация коммутатора верхнего уровня происходит за пределами фактических узлов. Дополнительные сведения об этом см. в [официальной документации Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Дальнейшие действия ## 
В этом разделе мы рассматривается как выбрать сетевые решения. Теперь все готово для шага 4:

> [!div class="nextstepaction"]
> [Присоединение Windows работников](./joining-windows-workers.md)