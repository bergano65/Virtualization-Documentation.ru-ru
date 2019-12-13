---
title: Создание главного узла Kubernetes с нуля
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Создание главного кластера Kubernetes.
keywords: kubernetes, 1,14, Master, Linux
ms.openlocfilehash: b1ec23b039ce6f5c42859452ecf3a8a5b35e006c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910424"
---
# <a name="creating-a-kubernetes-master"></a>Создание главного Kubernetes #
> [!NOTE]
> Это пошаговое руководством было проверено на Kubernetes v 1.14. Из-за изменчивости Kubernetes от версии до версии этот раздел может сделать предположения, которые не имеют значения true для всех будущих версий. Официальную документацию по инициализации Kubernetes хозяев с помощью кубеадм можно найти [здесь](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Просто включите [раздел "планирование смешанной ОС](#enable-mixed-os-scheduling) ".

> [!NOTE]  
> Для работы требуется недавно обновленный компьютер Linux. Kubernetes главные ресурсы, такие как [KUBE-DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [KUBE-Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)и [KUBE-аписервер](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) , еще не были перенесены в Windows. 

> [!tip]
> Инструкции для Linux адаптированы к установке **Ubuntu 16,04**. Другие дистрибутивы Linux, сертифицированные для запуска Kubernetes, также должны предоставлять эквивалентные команды, которые можно заменить. Они также будут успешно взаимодействовать с Windows.


## <a name="initialization-using-kubeadm"></a>Инициализация с помощью кубеадм ##
Если явно не указано иное, выполните следующие команды в качестве **корневого**.

Сначала Приготовьтесь к корневой оболочке с повышенными правами:

```bash
sudo –s
```

Убедитесь, что ваша виртуальная машина обновлена:

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Установка Docker ###
Чтобы иметь возможность использовать контейнеры, необходим обработчик контейнеров, например DOCKER. Чтобы получить последнюю версию, можно использовать [эти инструкции](https://docs.docker.com/install/linux/docker-ce/ubuntu/) для установки DOCKER. Чтобы убедиться, что DOCKER установлен правильно, запустите `hello-world` контейнер:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Установка кубеадм ###
Скачайте `kubeadm` двоичные файлы для дистрибутива Linux и инициализируйте кластер.

> [!Important]  
> В зависимости от дистрибутива Linux может потребоваться заменить `kubernetes-xenial` ниже правильным [кодовым названием](https://wiki.ubuntu.com/Releases).

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>Подготовка главного узла ###
Для Kubernetes в Linux требуется отключить пространство подкачки.

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Инициализация главного сервера ###
Запишите подсеть кластера (например, 10.244.0.0/16) и подсеть службы (например, 10.96.0.0/12) и инициализируйте свой главный компьютер с помощью кубеадм:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Это может занять несколько минут. После завершения вы увидите экран, подобный подтверждению того, что ваш образец был инициализирован:

![текст](media/kubeadm-init.png)

> [!tip]
> Обратите внимание на эту команду кубеадм JOIN. Должно срок действия маркера кубеадм, можно использовать `kubeadm token create --print-join-command` для создания нового маркера.

> [!tip]
> Если вы хотите использовать требуемую версию Kubernetes, можно передать флаг `--kubernetes-version` в кубеадм.

Мы еще не сделали этого. Чтобы использовать `kubectl` в качестве обычного пользователя, выполните следующую команду в непривилегированной  __**пользовательской оболочке**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Теперь вы можете использовать kubectl для изменения или просмотра сведений о кластере.

### <a name="enable-mixed-os-scheduling"></a>Включить планирование смешанных ОС ###
По умолчанию некоторые Kubernetes ресурсы записываются таким образом, чтобы они были запланированы на всех узлах. Однако в среде с несколькими операционными системами не требуется, чтобы ресурсы Linux перевлияли на узлы Windows и не перепланирулись на них, и наоборот. По этой причине необходимо применить метки [нодеселектор](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) . 

В связи с этим мы собираемся исправить версию [управляющей](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) программы Linux KUBE-proxy только для Linux.

Сначала создадим каталог для хранения файлов манифеста YAML:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Убедитесь, что для стратегии обновления `kube-proxy` управляющей программы задано значение [роллингупдате](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Затем установите исправление для демона, загрузив [этот нодеселектор](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) и примените его только к Linux:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

После успешного выполнения вы увидите "селекторы узлов" `kube-proxy` и все остальные Daemonset установлены в `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![текст](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Получение сведений о кластере ###
Чтобы успешно присоединить будущие узлы к базе данных master, следует отследить следующие сведения:
  1. `kubeadm join` команды из выходных данных ([здесь](#initialize-master))
    * Пример: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Подсеть кластера, определенная во время `kubeadm init` ([здесь](#initialize-master))
    * Пример: `10.244.0.0/16`
  3. Подсеть службы, определенная во время `kubeadm init` ([здесь](#initialize-master))
    * Пример: `10.96.0.0/12`
    * Можно также найти с помощью `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. KUBE — IP-адрес службы DNS 
    * Пример: `10.96.0.10`
    * Можно найти в поле "IP-адрес кластера" с помощью `kubectl get svc/kube-dns -n kube-system`
  5. Файл `config` Kubernetes, созданный после `kubeadm init` ([здесь](#initialize-master)). Если следовать инструкциям, это можно найти по следующим путям:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Проверка главного ##
Через несколько минут система должна быть в следующем состоянии:

  - В разделе `kubectl get pods -n kube-system`будут использоваться модули [Kubernetes для главных компонентов](https://kubernetes.io/docs/concepts/overview/components/#master-components) в `Running` состоянии.
  - При вызове `kubectl cluster-info` будут показаны сведения о сервере Kubernetes главного API в дополнение к надстройкам DNS.
  
> [!tip]
> Так как кубеадм не настраивает сеть, модули DNS могут по-прежнему находиться в `ContainerCreating` или `Pending` состоянии. После [выбора сетевого решения](./network-topologies.md)они перейдут в состояние `Running`.

## <a name="next-steps"></a>Дальнейшие действия ## 
В этом разделе мы рассмотрели, как настроить Kubernetes Master с помощью кубеадм. Теперь вы готовы к этапу 3.

> [!div class="nextstepaction"]
> [Выбор сетевого решения](./network-topologies.md)