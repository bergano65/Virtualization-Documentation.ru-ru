---
title: Запуск Kubernetes в качестве службы Windows
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Как выполнять Kubernetes компоненты службы Windows.
keywords: kubernetes, 1.13, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120599"
---
# <a name="kubernetes-components-as-windows-services"></a>Компоненты Kubernetes как службы Windows 

Некоторым пользователям может потребоваться настроить процессы, такие как flanneld.exe, kubelet.exe, помощью kube-proxy.exe или другие для запуска в качестве службы Windows. Это обеспечивает преимущества дополнительных отказоустойчивость, например процессы, автоматический перезапуск при непредвиденных сбоев процесс или узла.


## <a name="prerequisites"></a>Необходимые условия
1. Вы загрузили [nssm.exe](https://nssm.cc/download) в `c:\k` каталога
2. Вы к домену узла в кластере и запустите его [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) или [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) на вашем узле ранее

## <a name="registering-windows-services"></a>Регистрация службы Windows
Можно запустить [пример скрипта](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) какие использует nssm.exe, будет регистрироваться `kubelet`, `kube-proxy`, и `flanneld.exe` для запуска в качестве службы Windows в фоновом режиме:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
IP-адрес узла Windows. Вы можете использовать `ipconfig` найти это.

|  |  | 
|---------|---------|
|Параметр     | `-ManagementIP`        |
|Значение по умолчанию    | n.A.        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
Сетевой режим `l2bridge` (flannel gw узла) или `overlay` (flannel инкапсуляция) выбран в качестве [решения сети](./network-topologies.md).

> [!Important] 
> `overlay` режим сети (flannel инкапсуляция) требует двоичных файлов Kubernetes v1.14 или выше.

|  |  | 
|---------|---------|
|Параметр     | `-NetworkMode`        |
|Значение по умолчанию    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[Диапазону адресов подсети кластера](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Параметр     | `-ClusterCIDR`        |
|Значение по умолчанию    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
, [IP-адрес службы Kubernetes DNS](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Параметр     | `-KubeDnsServiceIP`        |
|Значение по умолчанию    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Каталог, где перенаправления журналы kubelet и помощью kube прокси в их соответствующие выходные файлы.

|  |  | 
|---------|---------|
|Параметр     | `-LogDir`        |
|Значение по умолчанию    | `C:\k`        |

---


> [!TIP] 
> Что-то пойдет не так, изучите [раздел по устранению неполадок](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>Подход с реализацией вручную
Следует [выше указанные сценарии](#registering-windows-services) не работ по, в этом разделе предоставляет некоторые *Примеры команд* которого можно зарегистрировать этих служб вручную пошаговые.

> [!TIP] 
> См. в разделе [Kubelet и помощью kube прокси теперь можно запускать как службы Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) Дополнительные сведения о том, как настроить `kubelet` и `kube-proxy` для запуска в качестве собственной службы Windows через `sc`.

### <a name="register-flanneldexe"></a>Регистрация flanneld.exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Регистрация kubelet.exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Регистрация помощью kube-proxy.exe (l2bridge / gw узла)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Регистрация помощью kube-proxy.exe (наложение / инкапсуляция)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```