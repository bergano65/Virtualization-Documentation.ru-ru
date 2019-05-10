---
title: Сетевые топологии
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Поддерживаемые топологии сети для Windows и Linux.
keywords: kubernetes, 1.14, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6a2b7021efa0d90b69a88e1b498cddeadb3af80e
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622919"
---
# <a name="network-solutions"></a>Решения для сети #

После [настройки главном узле Kubernetes](./creating-a-linux-master.md) вы готовы выбрать сетевые решения. Существует несколько способов сделать виртуальной [подсети кластера](./getting-started-kubernetes-windows.md#cluster-subnet-def) маршрутизируемый между узлами. Выберите один из следующих вариантов для Kubernetes в ОС Windows сегодня:

1. Используйте подключаемый модуль CNI например [Flannel](#flannel-in-vxlan-mode) для настройки сети наложения.
2. Используйте подключаемый модуль CNI например [Flannel](#flannel-in-host-gateway-mode) в программе маршруты для вас.
3. Настройка смарт- [переключения верхнего (ToR)](#configuring-a-tor-switch) для маршрутизации подсети.

> [!tip]  
> Существует четвертый сетевые решения для Windows, который использует открыть виртуальный коммутатор (OvS) и откройте виртуальной сети (OVN). Документирование это выходит за рамки данного документа, но вы можете прочитать [этим инструкциям](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) , чтобы настроить его.

## <a name="flannel-in-vxlan-mode"></a>Flannel в режиме инкапсуляция

Flannel в режиме инкапсуляция может использоваться для настройки сети настраиваемой виртуальный слой, который использует ИНКАПСУЛЯЦИЯ туннелирования для маршрутизации пакетов между узлами.

### <a name="prepare-kubernetes-master-for-flannel"></a>Подготовка главного узла Kubernetes для Flannel
Некоторые незначительные подготовки рекомендуется [главного узла Kubernetes](./creating-a-linux-master.md) в наших кластера. Рекомендуется, чтобы при использовании Flannel с мостом трафик IPv4 утилита iptables цепочки. Это можно сделать с помощью следующей команды:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Скачать & Настройка Flannel ###
Скачайте последние манифест Flannel:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Существует два раздела, следует изменить для включения на внутреннем сервере инкапсуляция сети.

1. В `net-conf.json` части вашего `kube-flannel.yml`, еще раз проверьте:
 * Задайте подсети кластера (например, «10.244.0.0/16»), как нужные.
 * VNI 4096 устанавливается на внутреннем сервере
 * Порт 4789 устанавливается на внутреннем сервере
2. В `cni-conf.json` части вашего `kube-flannel.yml`, измените имя сети на `"vxlan0"`.

После применения указанные выше действия вашего `net-conf.json` должен выглядеть следующим образом:
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]  
> VNI должно быть присвоено 4096 и 4789 для Flannel в Linux для взаимодействия с Flannel в Windows. Поддержка для других VNIs ожидается в ближайшее время. См. в разделе [ИНКАПСУЛЯЦИЯ](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) описание этих полей.

Ваше `cni-conf.json` должен выглядеть следующим образом:
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]  
> Дополнительные сведения о перечисленных выше параметров обратитесь к официальной CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [сопоставления портов](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)и [мост](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) подключаемый модуль документы для Linux.

### <a name="launch-flannel--validate"></a>Запуск проверки & Flannel ###
Запуск с использованием Flannel:

```bash
kubectl apply -f kube-flannel.yml
```

Затем, так как модули Flannel под управлением Linux, исправление для Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) для `kube-flannel-ds` DaemonSet только интересующих Linux (при запуске Flannel «flanneld» узла — процесс агента в Windows позже при присоединении):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Если узлы не x86-64-разрядных, замените `-amd64` выше с архитектуру процессора.

Через несколько минут вы увидите все модули, что приложение работает, если сеть модуля pod Flannel была развернута.

```bash
kubectl get pods --all-namespaces
```

![текст](media/kube-master.png)

Flannel DaemonSet также должны иметь NodeSelector `beta.kubernetes.io/os=linux` применения.

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-daemonset.png)

> [!tip]  
> Для остальных flannel - ds-* DaemonSets, их можно будет игнорироваться или удалить, так как они не запланированы, если нет узла, сопоставление этой архитектуры процессора.

> [!tip]  
> Путать? Ниже приведен полный [Пример помощью kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) для v0.11.0 Flannel этих действий, предварительно применяется для подсети кластера по умолчанию `10.244.0.0/16`.

После успешного выполнения продолжайте [дальнейшие действия](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel в режим узел шлюз

Вместе с [Flannel инкапсуляция](#flannel-in-vxlan-mode)другой вариант для сетей Flannel — *режим узел шлюз* (хост gw), который включает в себя программирования статических маршрутов на каждом узле в другой узел подсетями модулей pod, используя адрес узла целевому узлу в качестве следующего перехода.

### <a name="prepare-kubernetes-master-for-flannel"></a>Подготовка главного узла Kubernetes для Flannel

Некоторые незначительные подготовки рекомендуется [главного узла Kubernetes](./creating-a-linux-master.md) в наших кластера. Рекомендуется, чтобы при использовании Flannel с мостом трафик IPv4 утилита iptables цепочки. Это можно сделать с помощью следующей команды:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Скачать & Настройка Flannel ###
Скачайте последние манифест Flannel:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Существует один файл, который нужно изменить для включения хост gw сеть на обоих Windows и Linux.

В `net-conf.json` раздел из помощью kube-flannel.yml убедиться, что:
1. Тип используемого серверной части сети задано значение `host-gw` вместо `vxlan`.
2. Задайте подсети кластера (например, «10.244.0.0/16»), как нужные.

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

### <a name="launch-flannel--validate"></a>Запуск проверки & Flannel ###
Запуск с использованием Flannel:

```bash
kubectl apply -f kube-flannel.yml
```

Затем, так как модули Flannel под управлением Linux, исправление наших Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) для `kube-flannel-ds` DaemonSet только интересующих Linux (при запуске Flannel «flanneld» узла — процесс агента в Windows позже при присоединении):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Если узлы не x86-64-разрядных, замените `-amd64` выше с архитектурой процессора нужные.

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
> Для остальных flannel - ds-* DaemonSets, их можно будет игнорироваться или удалить, так как они не запланированы, если нет узла, сопоставление этой архитектуры процессора.

> [!tip]  
> Путать? Ниже приведен полный [Пример помощью kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) v0.11.0 Flannel для этих 2 действий, предварительно применяется для подсети кластера по умолчанию `10.244.0.0/16`.

После успешного выполнения продолжайте [дальнейшие действия](#next-steps).

## <a name="configuring-a-tor-switch"></a>Настройка коммутатора верхнего уровня ##
> [!NOTE]
> Этот раздел можно пропустить, если вы выбрали [Flannel как сетевые решения](#flannel-in-host-gateway-mode).
Конфигурация коммутатора верхнего уровня происходит за пределами фактических узлов. Дополнительные сведения об этом см. в [официальной документации Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Дальнейшие действия ## 
В этом разделе мы рассматривается как выбрать и настроить сетевые решения. Теперь все готово для шага 4:

> [!div class="nextstepaction"]
> [Присоединение Windows работников](./joining-windows-workers.md)