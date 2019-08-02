---
title: Сетевые топологии
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Поддерживаемые сетевые топологии в Windows и Linux.
keywords: кубернетес, 1,14, Windows, Приступая к работе
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884985"
---
# <a name="network-solutions"></a>Сетевые решения #

После [настройки основного узла кубернетес](./creating-a-linux-master.md) вы можете выбрать сетевое решение. Существует несколько способов создания маршрутизируемой [подсети кластера](./getting-started-kubernetes-windows.md#cluster-subnet-def) на разных узлах. Выберите один из следующих вариантов для Кубернетес в Windows сегодня:

1. Используйте подключаемый модуль CNI, например [фланнел](#flannel-in-vxlan-mode) , для настройки сети оверлея.
2. Использование подключаемого модуля CNI, например [фланнел](#flannel-in-host-gateway-mode) , к маршрутам программы (используется l2bridge сетевой режим).
3. Настройка переключателя Smart [Top of Rack (Тор)](#configuring-a-tor-switch) для маршрутизации подсети.

> [!tip]  
> В Windows есть четвертое сетевое решение, использующее открытую vSwitch (ОВС) и открытую виртуальную сеть (ОВН). Задокументируйте этот документ за пределами области для этого документа, но вы можете прочитать [эти инструкции](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) , чтобы настроить его.

## <a name="flannel-in-vxlan-mode"></a>Фланнел в режиме вкслан

Фланнел в режиме вкслан можно использовать для настройки настраиваемой виртуальной сети оверлея, в которой используется ВКСЛАН туннелирование, чтобы маршрутизировать пакеты между узлами.

### <a name="prepare-kubernetes-master-for-flannel"></a>Подготовка образца Кубернетес для Фланнел
Некоторые небольшие подготовки рекомендуются на [кубернетес образце](./creating-a-linux-master.md) в нашем кластере. Рекомендуется включить трафик IPv4 в мост для иптаблесных цепочек при использовании Фланнел. Это можно сделать с помощью следующей команды:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Скачать & Настройка Фланнел ###
Скачайте последнюю версию манифеста Фланнел:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Для включения сетевого сервера вкслан необходимо изменить два раздела:

1. В `net-conf.json` разделе `kube-flannel.yml`проверки дважды сделайте следующее:
 * Подсеть кластера (например, "10.244.0.0/16") задана нужным образом.
 * ВНИ 4096 установлен в серверной части
 * Порт 4789 установлен в серверной части
2. В `cni-conf.json` разделе `kube-flannel.yml`"мой" измените сетевое имя на `"vxlan0"`.

После применения описанных выше действий вы `net-conf.json` должны выглядеть следующим образом:
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
> Для взаимодействия с Фланнел в Windows вни необходимо установить значение 4096 и порт 4789 для Фланнел в Linux. Поддержка других Внис ожидается в ближайшее время. Подробнее об этих полях смотрите в [вкслан](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Вы `cni-conf.json` должны выглядеть следующим образом:
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
> Дополнительные сведения о перечисленных выше параметрах можно найти в разделе Официальные CNI [фланнел](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)и мосты для подключаемого модуля [моста](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) для Linux.

### <a name="launch-flannel--validate"></a>Запуск Фланнел & проверка ###
Запустить Фланнел с помощью:

```bash
kubectl apply -f kube-flannel.yml
```

Далее, так как Фланнелные среды предназначены для Linux, примените исправление Linux [нодеселектор](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) к `kube-flannel-ds` демону только на целевую версию Linux (мы запустим процесс агента хоста фланнел "фланнелд" в Windows позднее при присоединении):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Если какие-либо узлы на базе x86 не работают `-amd64` , замените их архитектурой процессора.

Через несколько минут вы должны увидеть все обыкновенные записи, как запущенные, если была развернута сеть Pod Фланнел.

```bash
kubectl get pods --all-namespaces
```

![текст](media/kube-master.png)

Фланнел демон должен также применять Нодеселектор `beta.kubernetes.io/os=linux` .

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-daemonset.png)

> [!tip]  
> Для остальных фланнел-DS-* Даемонсетс эти данные можно игнорировать или удалить, так как они не будут планироваться, если в них нет узлов, соответствующих этой архитектуре процессора.

> [!tip]  
> Заблуждени? Ниже приведен полный [Пример Кубе-фланнел. ИМЛ](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) для фланнел v 0.11.0 с предварительно примененными шагами для подсети `10.244.0.0/16`кластера по умолчанию.

После успешного выполнения переходите к [следующим действиям](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Фланнел в режиме основного шлюза

Наряду с [фланнел вкслан](#flannel-in-vxlan-mode)другим вариантом для фланнел Network является *режим Host Gateway* (Host-GW), который включает в себя программирование статических маршрутов на каждом узле подсетями Pod другого узла, используя адрес узла целевого узла в качестве следующего прыжка.

### <a name="prepare-kubernetes-master-for-flannel"></a>Подготовка образца Кубернетес для Фланнел

Некоторые небольшие подготовки рекомендуются на [кубернетес образце](./creating-a-linux-master.md) в нашем кластере. Рекомендуется включить трафик IPv4 в мост для иптаблесных цепочек при использовании Фланнел. Это можно сделать с помощью следующей команды:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Скачать & Настройка Фланнел ###
Скачайте последнюю версию манифеста Фланнел:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Для включения сети Host-GW в Windows и Linux необходимо изменить один из файлов

В `net-conf.json` разделе Кубе-фланнел. ИМЛ дважды убедитесь, что:
1. Для `host-gw` используемого типа сетевой серверной части вместо него устанавливается значение " `vxlan`".
2. Подсеть кластера (например, "10.244.0.0/16") задана нужным образом.

После применения двух шагов вы `net-conf.json` должны выглядеть следующим образом:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Запуск Фланнел & проверка ###
Запустить Фланнел с помощью:

```bash
kubectl apply -f kube-flannel.yml
```

Далее, так как Фланнелные среды предназначены для Linux, примените исправление [](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) для Linux нодеселектор `kube-flannel-ds` к демону только на целевую версию Linux (мы запустим процесс агента узла фланнелд для Windows при присоединении):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Если какие-либо узлы на базе x86 не работают `-amd64` , замените выше на нужную архитектуру процессора.

Через несколько минут вы должны увидеть все обыкновенные записи, как запущенные, если была развернута сеть Pod Фланнел.

```bash
kubectl get pods --all-namespaces
```

![текст](media/kube-master.png)

Фланнел демон должен также применять Нодеселектор.

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-daemonset.png)

> [!tip]  
> Для остальных фланнел-DS-* Даемонсетс эти данные можно игнорировать или удалить, так как они не будут планироваться, если в них нет узлов, соответствующих этой архитектуре процессора.

> [!tip]  
> Заблуждени? Ниже приведен полный [Пример Кубе-фланнел. ИМЛ](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) для фланнел v 0.11.0 со следующими 2 шагами, которые предварительно применены к `10.244.0.0/16`подсети кластера по умолчанию.

После успешного выполнения переходите к [следующим действиям](#next-steps).

## <a name="configuring-a-tor-switch"></a>Настройка переключателя Тор ##
> [!NOTE]
> Вы можете пропустить этот раздел, если вы выбрали [фланнел в качестве сетевого решения](#flannel-in-host-gateway-mode).
Настройка переключателя Тор выполняется за пределами реальных узлов. Подробнее об этом можно узнать в разделе [официальные документы кубернетес](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Дальнейшие действия ## 
В этом разделе мы рассмотрели, как выбрать и настроить сетевое решение. Теперь вы готовы к шагу 4:

> [!div class="nextstepaction"]
> [Присоединение к сотрудникам Windows](./joining-windows-workers.md)