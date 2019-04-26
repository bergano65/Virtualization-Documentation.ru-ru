---
title: Создание главного узла Kubernetes с нуля
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Создание главного узла кластера Kubernetes.
keywords: kubernetes, 1,13, главный узел, linux
ms.openlocfilehash: 8a3fb073616d115ab84e6cc36f0fb6cedbcf1f7d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578255"
---
# <a name="creating-a-kubernetes-master"></a>Создание главного узла Kubernetes #
> [!NOTE]
> В этом руководстве были проверены на Kubernetes v1.13. Из-за изменчивости Kubernetes от версии к версии в этом разделе могут привести к предположения, которые не содержат true для всех будущих версий. Официальной документации по инициализации образцов Kubernetes с помощью kubeadm можно найти [здесь](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Просто поверх включите [планирования раздел смешанных ОС](#enable-mixed-os-scheduling) .

> [!NOTE]  
> Недавно обновленный компьютер под управлением Linux для этой операции требуется вдоль; Kubernetes основных и подробных данных ресурсов, как [помощью kube-dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [помощью kube заданий](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)и [помощью kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) не были перенесены в Windows еще. 

> [!tip]
> Инструкции Linux подобранными по направлению к **Ubuntu 16.04**. Другие дистрибутивов Linux, сертифицированных для запуска Kubernetes также должны выдавать эквивалент команды, которые можно заменить. Они также будут успешно взаимодействовать с Windows.


## <a name="initialization-using-kubeadm"></a>С помощью kubeadm инициализации ##
Если явно не указаны в противном случае, выполните все команды ниже как **корень**.

Во-первых размещать в оболочке корневой с повышенными привилегиями:

```bash
sudo –s
```

Убедитесь, что компьютер находится в актуальном состоянии.

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Установка Docker ###
Чтобы иметь возможность использовать контейнеры, требуется механизм контейнера, например, Docker. Чтобы получить последнюю версию, можно использовать [следующие действия](https://docs.docker.com/install/linux/docker-ce/ubuntu/) для установки Docker. Вы можете проверить, что docker установлены правильно, выполнив `hello-world` контейнера:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Установка kubeadm ###
Скачать `kubeadm` двоичные файлы для дистрибутива Linux и инициализировать кластер.

> [!Important]  
> В зависимости от дистрибутива Linux, вам может потребоваться заменить `kubernetes-xenial` ниже с правильным [Название](https://wiki.ubuntu.com/Releases).

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>Подготовка главный узел ###
Kubernetes в Linux требует места буферов выключить:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Инициализация главного узла ###
Запишите подсети кластера (например, 10.244.0.0/16) и подсеть службы (например, 10.96.0.0/12) и инициализировать с помощью kubeadm образце:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Это может занять несколько минут. После завершения вы увидите экран, как это подтверждения того, что был инициализирован образце:

![текст](media/kubeadm-init.png)

> [!tip]
> Следует отметить этой команды kubeadm соединения. Срок действия маркера kubeadm блоков, вы можете использовать `kubeadm token create --print-join-command` создать новый маркер.

> [!tip]
> Если у вас есть нужную версию Kubernetes вы хотите использовать, вы можете передать `--kubernetes-version` флаг kubeadm.

Мы не остановимся еще. Чтобы использовать `kubectl` как обычный пользователь, выполняются следующие __**в оболочке unelevated, некорневых пользователя**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Теперь можно использовать kubectl для редактирования или просмотреть сведения о кластере.

### <a name="enable-mixed-os-scheduling"></a>Включить планирование смешанных ОС ###
По умолчанию определенных ресурсов Kubernetes записываются таким образом, что они все запланированные на всех узлах. Тем не менее в среде рабочие мы не хотим Linux ресурсы, которые влияют на или быть совместимым с двойной планируется на узлах Windows и наоборот. По этой причине необходимо применить [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) метки. 

Таким образом, мы будем исправление linux помощью kube прокси [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) только целевой объект Linux.

Во-первых создадим каталог для хранения файлов манифеста .yaml:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Убедитесь, что стратегия обновления `kube-proxy` DaemonSet задано значение [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Затем исправление DaemonSet, скачав [Этот nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) и применить его только интересующих Linux:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

После успешной, вы должны увидеть «Селекторы узла» из `kube-proxy` и любые другие DaemonSets значение `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Сбор сведений о кластера ###
Для успешного присоединения к будущих узлов в образце, вам следует хранить список следующие сведения:
  1. `kubeadm join` Команда выходных данных ([ниже](#initialize-master))
    * Пример. `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Определенные во время подсеть кластера `kubeadm init` ([ниже](#initialize-master))
    * Пример. `10.244.0.0/16`
  3. Служба подсетью, заданной во время `kubeadm init` ([ниже](#initialize-master))
    * Пример. `10.96.0.0/12`
    * Также можно найти с помощью `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. IP-адрес службы помощью Kube-dns 
    * Пример. `10.96.0.10`
    * Можно найти в поле, используя «IP-адрес для кластера» `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` файла, созданного после `kubeadm init` ([ниже](#initialize-master)). Если вы следовали инструкциям, это можно найти в следующих путях:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Проверка главного узла ##
Через несколько минут система должна быть в следующем состоянии:

  - В разделе `kubectl get pods -n kube-system`, будет присутствовать модули для [Kubernetes основных и подробных данных компонентов](https://kubernetes.io/docs/concepts/overview/components/#master-components) в `Running` состояния.
  - Вызов `kubectl cluster-info` отобразит информацию о главном сервере API Kubernetes в дополнение к надстройкам DNS.
  
> [!tip]
> Так как kubeadm не настройки сети, модули DNS по-прежнему могут быть в `ContainerCreating` или `Pending` состояния. Они будут переключаться на `Running` состояние после [выбора решением для сети](./network-topologies.md).

## <a name="next-steps"></a>Дальнейшие действия ## 
В этом разделе мы описаны способы настройки главного узла Kubernetes с помощью kubeadm. Теперь все готово для шага 3:

> [!div class="nextstepaction"]
> [Выбор сетевого решения](./network-topologies.md)