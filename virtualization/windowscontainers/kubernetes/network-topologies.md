---
title: Сетевые топологии
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Поддерживаемые сетевые топологии в Windows и Linux.
keywords: kubernetes, 1,14, Windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910314"
---
# <a name="network-solutions"></a>Network Solutions #

После [настройки главного узла Kubernetes](./creating-a-linux-master.md) вы можете выбрать сетевое решение. Существует несколько способов маршрутизации [подсети виртуального кластера](./getting-started-kubernetes-windows.md#cluster-subnet-def) между узлами. Выберите один из следующих вариантов для Kubernetes в Windows сегодня:

1. Используйте подключаемый модуль CNI, например [фланнел](#flannel-in-vxlan-mode) , чтобы настроить сеть наложения.
2. Используйте подключаемый модуль CNI, например [фланнел](#flannel-in-host-gateway-mode) , для вас (использует сетевой режим l2bridge).
3. Настройте [коммутатор верхнего уровня в стойке (Tor)](#configuring-a-tor-switch) для маршрутизации подсети.

> [!tip]  
> В Windows имеется четвертое сетевое решение, использующее Open vSwitch (OvS) и Open Virtual Network (ОВН). Документ не входит в область этого документа, но вы можете прочитать [эти инструкции](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) , чтобы настроить его.

## <a name="flannel-in-vxlan-mode"></a>Фланнел в режиме вкслан

Фланнел в режиме вкслан можно использовать для установки настраиваемой сети виртуального оверлея, которая использует туннелирование ВКСЛАН для маршрутизации пакетов между узлами.

### <a name="prepare-kubernetes-master-for-flannel"></a>Подготовка Kubernetes Master для Фланнел
На [главном Kubernetes](./creating-a-linux-master.md) в нашем кластере рекомендуется выполнить некоторые незначительные подготовительные процедуры. Рекомендуется включить передачу трафика IPv4 через мост в цепочки iptables при использовании Фланнел. Это можно сделать с помощью следующей команды:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Скачать & настроить Фланнел ###
Скачайте последний манифест Фланнел:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Чтобы включить серверную часть вкслан Networking, необходимо изменить два раздела:

1. В разделе `net-conf.json` `kube-flannel.yml`дважды проверьте следующее:
 * Подсеть кластера (например, "10.244.0.0/16") задается по желанию.
 * ВНИ 4096 задается в серверной части
 * Порт 4789 установлен в серверной части
2. В разделе `cni-conf.json` `kube-flannel.yml`измените имя сети на `"vxlan0"`.

После применения описанных выше действий `net-conf.json` должен выглядеть следующим образом:
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
> Для вни необходимо задать значение 4096 и порт 4789 для Фланнел в Linux, чтобы взаимодействовать с Фланнел в Windows. Поддержка других Внис ожидается в ближайшее время. Описание этих полей см. в разделе [вкслан](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Ваш `cni-conf.json` должен выглядеть следующим образом:
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
> Дополнительные сведения о перечисленных выше вариантах см. в официальных документах CNI [фланнел](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)и [Bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) plugin для Linux.

### <a name="launch-flannel--validate"></a>Запустить Фланнел & проверку ###
Запустите Фланнел с помощью:

```bash
kubectl apply -f kube-flannel.yml
```

Далее, так как модули Фланнел для Linux основаны на Windows, примените исправление [нодеселектор](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) для linux к `kube-flannel-ds`ому демону только для Linux (в дальнейшем мы запустили процесс агента узла фланнел "фланнелд" при присоединении):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Если какие-либо узлы не основаны на x86 64, замените `-amd64` выше архитектурой процессора.

Через несколько минут вы должны увидеть все модули Pod, как только они будут запущены, если была развернута сеть Фланнел-модулей.

```bash
kubectl get pods --all-namespaces
```

![текст](media/kube-master.png)

Фланнел управляющая программа должна также иметь `beta.kubernetes.io/os=linux` применения Нодеселектор.

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-daemonset.png)

> [!tip]  
> Для оставшихся фланнел-DS-* Daemonset их можно игнорировать или удалить, так как они не будут планироваться, если нет узлов, соответствующих этой архитектуре процессора.

> [!tip]  
> Странно? Ниже приведен полный [Пример Кубе-фланнел. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) для фланнел v 0.11.0 с предварительно примененными действиями для подсети кластера по умолчанию `10.244.0.0/16`.

После успешного выполнения перейдите к [следующим шагам](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Фланнел в режиме главного шлюза

Наряду с [фланнел вкслан](#flannel-in-vxlan-mode)другим вариантом для фланнел Network является *режим узла-шлюза* (Host-GW), который включает в себя программирование статических маршрутов на каждом узле в подсетях Pod другого узла с помощью адреса узла целевого узла в качестве следующего прыжка.

### <a name="prepare-kubernetes-master-for-flannel"></a>Подготовка Kubernetes Master для Фланнел

На [главном Kubernetes](./creating-a-linux-master.md) в нашем кластере рекомендуется выполнить некоторые незначительные подготовительные процедуры. Рекомендуется включить передачу трафика IPv4 через мост в цепочки iptables при использовании Фланнел. Это можно сделать с помощью следующей команды:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Скачать & настроить Фланнел ###
Скачайте последний манифест Фланнел:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Чтобы включить сеть Host-GW в Windows и Linux, необходимо изменить один из файлов.

В разделе `net-conf.json` Кубе-фланнел. yml дважды проверьте следующее:
1. Тип используемой сетевой серверной части задается `host-gw` вместо `vxlan`.
2. Подсеть кластера (например, "10.244.0.0/16") задается по желанию.

После применения 2 действий `net-conf.json` должен выглядеть следующим образом:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Запустить Фланнел & проверку ###
Запустите Фланнел с помощью:

```bash
kubectl apply -f kube-flannel.yml
```

Далее, так как модули Фланнел для Linux используются в ОС под управлением Windows, примените исправление [нодеселектор](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) для linux к `kube-flannel-ds`ому демону только для Linux (мы запустили процесс агента узла фланнел "фланнелд" в дальнейшем при присоединении):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Если какие-либо узлы не основаны на x86 64, замените `-amd64` выше на нужную архитектуру процессора.

Через несколько минут вы должны увидеть все модули Pod, как только они будут запущены, если была развернута сеть Фланнел-модулей.

```bash
kubectl get pods --all-namespaces
```

![текст](media/kube-master.png)

В Фланнел DAEMON должен быть также применен Нодеселектор.

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-daemonset.png)

> [!tip]  
> Для оставшихся фланнел-DS-* Daemonset их можно игнорировать или удалить, так как они не будут планироваться, если нет узлов, соответствующих этой архитектуре процессора.

> [!tip]  
> Странно? Ниже приведен полный [Пример Кубе-фланнел. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) для фланнел v 0.11.0 с этими 2 шагами, предварительно примененными к подсети кластера по умолчанию `10.244.0.0/16`.

После успешного выполнения перейдите к [следующим шагам](#next-steps).

## <a name="configuring-a-tor-switch"></a>Настройка параметра ToR ##
> [!NOTE]
> Этот раздел можно пропустить, если вы выбрали [фланнел в качестве сетевого решения](#flannel-in-host-gateway-mode).
Настройка параметра ToR выполняется за пределами реальных узлов. Дополнительные сведения об этом см. в [официальных документах Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Дальнейшие действия ## 
В этом разделе мы рассмотрели, как выбрать и настроить сетевое решение. Теперь вы готовы к шагу 4:

> [!div class="nextstepaction"]
> [Присоединение к рабочим процессам Windows](./joining-windows-workers.md)