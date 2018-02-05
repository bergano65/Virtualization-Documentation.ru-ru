---
title: "Компиляция двоичных файлов Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Компиляция и кросс-компиляции двоичных файлов Kubernetes из источника."
keywords: "kubernetes, 1.9, linux, компиляция"
ms.openlocfilehash: c9b0146202d7e9e5d857ca88faa43282bd504dfa
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2018
---
# <a name="compiling-kubernetes-binaries"></a>Компиляция двоичных файлов Kubernetes #
Для компиляции двоичных файлов Kubernetes требуется рабочая среда Go. На этой странице описывается несколько способов компиляции двоичных файлов Linux и кросс-компиляции двоичных файлов Windows.

## <a name="installing-go"></a>Установка Go ##
Для простоты установка Go выполняется во временной настраиваемой папке:

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz -O go1.9.2.tar.gz
tar -vxzf go1.9.2.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]  
> При этом настраиваются переменные среды для сеанса. Добавьте операции `export`, чтобы сделать настройки `~/.profile` постоянными.

Запустите `go env`, чтобы правильно настроить пути. Существует несколько вариантов создания двоичных файлов Kubernetes:

  - [локальная](#build-locally) сборка;
  - создание двоичных файлов с помощью [Vagrant](#build-with-vagrant);
  - использование [стандартных контейнерных сценариев построения](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts) в проекте Kubernetes. Для этого следуйте инструкциям по [локальному созданию](#build-locally) до шага `make`, а затем используйте инструкции по ссылке.

Чтобы скопировать двоичные файлы Windows в соответствующие им узлы, используйте визуальное средство, например [WinSCP](https://winscp.net/eng/download.php), или средство командной строки, например [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), для их переноса в каталог `C:\k`.


## <a name="building-locally"></a>Локальное построение ##
> [!Tip]  
> При возникновении ошибок «Отказано в разрешении» их можно избежать, выполнив сначала построение Linux `kubelet`, как описано в примечании в [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176):
>  
> _Из-за того, что кажется ошибкой в системе сборки Kubernetes для Windows, необходимо сначала выполнить построение двоичного файла Linux для создания `_output/bin/deepcopy-gen`. Сборка в Windows без этого действия приведет к созданию пустого объекта `deepcopy-gen`._

Сначала получите репозиторий Kubernetes:

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about 
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

Теперь извлеките ветвь для создания двоичного файла Linux `kubelet`. Это необходимо, чтобы избежать ошибок построения Windows, указанных выше. Здесь мы используем `v1.9.1`. После выполнения операции `git checkout` вы можете применить ожидающие PR, исправления или внести другие изменения в пользовательские двоичные файлы.

```bash
git checkout tags/v1.9.1
make clean && make WHAT=cmd/kubelet
```

Наконец создайте необходимые клиентские двоичные файлы Windows (последний этап может отличаться в зависимости откуда потребуется извлечь двоичные файлы Windows в дальнейшем):

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Действия по созданию двоичных файлов Linux ничем не отличаются, просто уберите префикс `KUBE_BUILD_PLATFORMS=windows/amd64` из команд. Выходным каталогом будет `_output/.../linux/amd64`.


## <a name="build-with-vagrant"></a>Построение с помощью Vagrant ##
Программа установки Vagrant доступна [здесь](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant). Используйте ее для подготовки виртуальной машины Vagrant, а затем выполните эти команды внутри нее:

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.9.1
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet 
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy 
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```

