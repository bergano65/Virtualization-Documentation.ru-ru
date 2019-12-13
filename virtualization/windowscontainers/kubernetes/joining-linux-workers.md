---
title: Присоединение узлов Linux
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Linux к кластеру Kubernetes с помощью v 1.14.
keywords: kubernetes, 1,14, Windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 88207939c82bfe8ffa0b088cfd61cf4ab22cb10a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909954"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Присоединение узлов Linux к кластеру

После [настройки главного узла Kubernetes](creating-a-linux-master.md) и [выбора нужного сетевого решения](network-topologies.md)вы можете присоединить узлы Linux к кластеру. Для этого требуется выполнить некоторые [подготовительные действия на узле Linux](joining-linux-workers.md#preparing-a-linux-node) перед присоединением.
> [!tip]
> Инструкции для Linux адаптированы к установке **Ubuntu 16,04**. Другие дистрибутивы Linux, сертифицированные для запуска Kubernetes, также должны предоставлять эквивалентные команды, которые можно заменить. Они также будут успешно взаимодействовать с Windows.

## <a name="preparing-a-linux-node"></a>Подготовка узла Linux

> [!NOTE]
> Если явно не указано иное, выполните все команды в **пользовательской оболочке с повышенными привилегиями**.

Сначала Приготовьтесь к корневой оболочке:

```bash
sudo –s
```

Убедитесь, что ваша виртуальная машина обновлена:

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Установка Docker

Чтобы иметь возможность использовать контейнеры, необходим обработчик контейнеров, например DOCKER. Чтобы получить последнюю версию, можно использовать [эти инструкции](https://docs.docker.com/install/linux/docker-ce/ubuntu/) для установки DOCKER. Чтобы убедиться, что DOCKER установлен правильно, запустите `hello-world` образ:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Установка кубеадм

Скачайте `kubeadm` двоичные файлы для дистрибутива Linux и инициализируйте кластер.

> [!Important]  
> В зависимости от дистрибутива Linux может потребоваться заменить `kubernetes-xenial` ниже правильным [кодовым названием](https://wiki.ubuntu.com/Releases).

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>Отключить переключение

Для Kubernetes в Linux требуется отключить пространство подкачки.

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Только фланнел) Включение передачи трафика IPv4 через мост в iptables

Если вы выбрали Фланнел в качестве сетевого решения, рекомендуется включить передачу трафика IPv4 через мост в цепочки iptables. [Это уже необходимо сделать для главного](network-topologies.md#flannel-in-host-gateway-mode) сервера, и теперь его нужно повторить для узла Linux, который планируется присоединить. Это можно сделать с помощью следующей команды:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Копирование сертификата Kubernetes

**Как обычный (не корневой) пользователь**, выполните следующие три шага.

1. Создайте Kubernetes для каталога Linux:

```bash
mkdir -p $HOME/.kube
```

2. Скопируйте файл сертификата Kubernetes (`$HOME/.kube/config`) [из главного](./creating-a-linux-master.md#collect-cluster-information) сервера и сохраните его как `$HOME/.kube/config` в рабочей роли.

> [!tip]
> Для перемещения файла конфигурации между узлами можно использовать средства на основе SCP, такие как [WinSCP](https://winscp.net/eng/download.php) .

3. Задайте для параметра владение файлом скопированный файл конфигурации следующее:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Присоединение к узлу

Наконец, чтобы присоединиться к кластеру, выполните команду `kubeadm join`, которая [была записана ранее](./creating-a-linux-master.md#initialize-master) **в качестве корневого**:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

В случае успешного выполнения вы увидите аналогичные выходные данные:

![текст](./media/node-join.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом разделе мы рассмотрели, как присоединить рабочие роли Linux к нашему кластеру Kubernetes. Теперь вы готовы к шагу 6.
> [!div class="nextstepaction"]
> [Развертывание ресурсов Kubernetes](./deploying-resources.md)