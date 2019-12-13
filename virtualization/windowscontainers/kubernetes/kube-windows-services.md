---
title: Запуск Kubernetes как службы Windows
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Как запускать компоненты Kubernetes в качестве служб Windows.
keywords: kubernetes, 1,14, Windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: cd5026a244b57b5c70d4abfe076839130315a4f5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909804"
---
# <a name="kubernetes-components-as-windows-services"></a>Компоненты Kubernetes как службы Windows 

Некоторым пользователям может потребоваться настроить такие процессы, как фланнелд. exe, kubelet. exe, Кубе-прокси. exe или другие, для запуска в качестве служб Windows. Это приводит к дополнительным преимуществам отказоустойчивости, таким как процессы, которые автоматически перезапускаются при непредвиденном сбое процесса или узла.


## <a name="prerequisites"></a>Предварительные условия
1. Вы загрузили [файл НССМ. exe](https://nssm.cc/download) в каталог `c:\k`
2. Вы присоединили узел к кластеру и запускаете скрипт [install. ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) или [Start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) на узле ранее

## <a name="registering-windows-services"></a>Регистрация служб Windows
Можно запустить [Пример скрипта](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) , который использует НССМ. exe, который будет регистрировать `kubelet`, `kube-proxy`и `flanneld.exe` для запуска в качестве служб Windows в фоновом режиме:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementiptabmanagementip"></a>[манажементип](#tab/ManagementIP)
IP-адрес, назначенный узлу Windows. Чтобы найти это, можно использовать `ipconfig`.

|  |  | 
|---------|---------|
|Параметр     | `-ManagementIP`        |
|Значение по умолчанию    | н.а.        |


# <a name="networkmodetabnetworkmode"></a>[нетворкмоде](#tab/NetworkMode)
Сетевой режим `l2bridge` (фланнел Host-GW) или `overlay` (фланнел вкслан) выбран в качестве [сетевого решения](./network-topologies.md).

> [!Important] 
> `overlay` сетевой режим (фланнел вкслан) требует наличия двоичных файлов Kubernetes v 1.14 или более поздней версии.

|  |  | 
|---------|---------|
|Параметр     | `-NetworkMode`        |
|Значение по умолчанию    | `l2bridge`        |


# <a name="clustercidrtabclustercidr"></a>[клустерЦидр](#tab/ClusterCIDR)
[Диапазон подсети кластера](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Параметр     | `-ClusterCIDR`        |
|Значение по умолчанию    | `10.244.0.0/16`        |


# <a name="kubednsserviceiptabkubednsserviceip"></a>[кубеднссервицеип](#tab/KubeDnsServiceIP)
[IP-адрес службы DNS Kubernetes](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Параметр     | `-KubeDnsServiceIP`        |
|Значение по умолчанию    | `10.96.0.10`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
Каталог, в котором журналы kubelet и KUBE-proxy перенаправляются в соответствующие выходные файлы.

|  |  | 
|---------|---------|
|Параметр     | `-LogDir`        |
|Значение по умолчанию    | `C:\k`        |

---


> [!TIP] 
> Если что-то пошло не так, ознакомьтесь с [разделом устранение неполадок](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>Ручной подход
Если указанный [выше скрипт](#registering-windows-services) не подходит для вас, в этом разделе приводится несколько *примеров команд* , которые можно использовать для регистрации этих служб вручную с пошаговыми действиями.

> [!TIP] 
> Дополнительные сведения о настройке `kubelet` и `kube-proxy` для запуска в качестве собственных служб Windows с помощью `sc`см. в разделе [Kubelet и KUBE-proxy](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) .

### <a name="register-flanneldexe"></a>Регистрация фланнелд. exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Регистрация kubelet. exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Регистрация Кубе-прокси. exe (l2bridge/Host-GW)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Регистрация Кубе-прокси. exe (Overlay/вкслан)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```