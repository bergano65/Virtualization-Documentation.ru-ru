---
title: Присоединение узла Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение к кластеру Kubernetes с v1.12 узла Linux.
keywords: kubernetes, 1.12, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 97c12d70db9679dbb85877f0985c6053f95fa500
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/15/2018
ms.locfileid: "6947983"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Присоединение к кластеру узла Linux

После [настройки главном узле Kubernetes](creating-a-linux-master.md) и [выбрать нужную сеть решение](network-topologies.md), вы готовы присоединение узла Linux в кластере. Это требуется некоторые [подготовки на узле Linux](joining-linux-workers.md#preparing-a-linux-node) , прежде чем присоединять.
> [!tip]
> Инструкции Linux подобранными по направлению к **Ubuntu 16.04**. Другие дистрибутивов Linux, сертифицированных для запуска Kubernetes также должны выдавать эквивалент команды, которые можно заменить. Они также будет успешно взаимодействовать с Windows.

## <a name="preparing-a-linux-node"></a>Подготовка узла Linux

> [!NOTE]
> Если явно не указаны в противном случае, выполните все команды в **оболочку с повышенными привилегиями, привилегированного пользователя**.

Сначала получите оболочки корневой:

```bash
sudo –s
```

Убедитесь, что компьютер находится в актуальном состоянии.

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Установка Docker

Чтобы иметь возможность использовать контейнеры, требуется модуль контейнера, например, Docker. Чтобы получить последнюю версию, можно использовать [следующие действия](https://docs.docker.com/install/linux/docker-ce/ubuntu/) для установки Docker. Вы можете проверить, что docker установлены правильно, выполнив `hello-world` изображения:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Установка kubeadm

Загрузите `kubeadm` двоичные файлы для дистрибутива Linux и инициализировать кластер.

> [!Important]  
> В зависимости от дистрибутива Linux, вам может потребоваться заменить `kubernetes-xenial` ниже с правильным [кодовое название](https://wiki.ubuntu.com/Releases).

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

Если вы выбрали Flannel как сетевые решения, рекомендуется включить в мост трафик IPv4 утилита iptables цепочки. Вы должны [уже сделали это для главного узла](network-topologies.md#flannel-in-host-gateway-mode) и теперь необходимо повторите ее объяснения присоединение узла Linux. Это можно сделать с помощью следующей команды:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Копирование сертификата Kubernetes

**Как обычный, пользователь (некорневой)**, выполните следующие шаги 3.

1. Создайте Kubernetes для Linux каталог:

```bash
mkdir -p $HOME/.kube
```

1. Скопируйте файл сертификата Kubernetes (`$HOME/.kube/config`) [от главного узла](./creating-a-linux-master.md#collect-cluster-information) и сохранять в виде `$HOME/.kube/config` рабочие.

1. Установите владение файла конфигурации скопированных следующим образом:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Присоединение узла

Наконец, для присоединения к кластеру, запустите `kubeadm join` команды [мы отмечалось вниз](./creating-a-linux-master.md#initialize-master) **как корень**:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

В случае успеха вы должны увидеть похожие вывод:

![текст](./media/node-join.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом разделе мы рассматривается как присоединить устройство Linux сотрудников к нашей кластера Kubernetes. Теперь вы готовы к шагу 6:
> [!div class="nextstepaction"]
> [Развертывание Kubernetes ресурсы](./deploying-resources.md)