---
title: Присоединение узла Linux
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение к кластеру Kubernetes с v1.14 узла Linux.
keywords: kubernetes, 1.14, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 88207939c82bfe8ffa0b088cfd61cf4ab22cb10a
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622949"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Присоединение к кластеру узла Linux

После [настройки Kubernetes главном узле](creating-a-linux-master.md) и [выбрать нужную сеть решение](network-topologies.md), можно приступать к присоединение узла Linux в кластере. Прежде чем присоединять этого некоторые [подготовки на узле Linux](joining-linux-workers.md#preparing-a-linux-node) .
> [!tip]
> Инструкции Linux подобранными по направлению к **Ubuntu 16.04**. Другие дистрибутивов Linux, сертифицированных для запуска Kubernetes также должны выдавать эквивалент команды, которые можно заменить. Они также будут успешно взаимодействовать с Windows.

## <a name="preparing-a-linux-node"></a>Подготовка узла Linux

> [!NOTE]
> Если явно не указаны в противном случае, выполните все команды в **оболочку с повышенными привилегиями, привилегированного пользователя**.

Во-первых получите оболочки корневой:

```bash
sudo –s
```

Убедитесь, что компьютер находится в актуальном состоянии.

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Установка Docker

Чтобы иметь возможность использовать контейнеры, требуется механизм контейнера, например, Docker. Чтобы получить последнюю версию, можно использовать [следующие действия](https://docs.docker.com/install/linux/docker-ce/ubuntu/) для установки Docker. Вы можете проверить, что docker установлены правильно, выполнив `hello-world` изображения:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Установка kubeadm

Скачать `kubeadm` двоичные файлы для дистрибутива Linux и инициализировать кластер.

> [!Important]  
> В зависимости от дистрибутива Linux, вам может потребоваться заменить `kubernetes-xenial` ниже с правильным [Название](https://wiki.ubuntu.com/Releases).

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>Отключить буферов

Kubernetes в Linux требует места буферов выключить:

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Flannel) С мостом трафик IPv4 утилита iptables

Если вы выбрали Flannel как сетевые решения, рекомендуется включить мост трафик IPv4 утилита iptables цепочки. Вы должны [сделано для главного узла](network-topologies.md#flannel-in-host-gateway-mode) и теперь необходимо повторить для объяснения присоединение узла Linux. Это можно сделать с помощью следующей команды:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Копирование сертификата Kubernetes

**Обычный, лица (некорневой)**, выполните следующие шаги 3.

1. Создание Kubernetes для Linux каталог:

```bash
mkdir -p $HOME/.kube
```

2. Скопируйте файл сертификата Kubernetes (`$HOME/.kube/config`) [от главного узла](./creating-a-linux-master.md#collect-cluster-information) и сохранять в виде `$HOME/.kube/config` рабочие.

> [!tip]
> На основе scp средства, такие как [WinSCP](https://winscp.net/eng/download.php) можно использовать для перемещения между узлами в файле конфигурации.

3. Установите владение файла конфигурации скопированных следующим образом:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Присоединение узла

Наконец, для присоединения к кластеру, запустите `kubeadm join` команды [мы отмечалось вниз](./creating-a-linux-master.md#initialize-master) **как корень**:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

В случае успеха вы должны увидеть похожие выходные данные для этого:

![текст](./media/node-join.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом разделе мы рассматривается как присоединить устройство Linux сотрудников к нашей кластера Kubernetes. Теперь вы готовы к шагу 6:
> [!div class="nextstepaction"]
> [Развертывание ресурсов Kubernetes](./deploying-resources.md)