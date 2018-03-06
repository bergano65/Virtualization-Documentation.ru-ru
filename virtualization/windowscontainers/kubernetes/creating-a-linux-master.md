---
title: "Создание главного узла Kubernetes с нуля"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Создание главного узла кластера Kubernetes с нуля."
keywords: "kubernetes, 1.9, главный узел, linux"
ms.openlocfilehash: 3ea338f7af3dd921731fce0ec5a8b2cf8c4fef0c
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/01/2018
---
# <a name="kubernetes-master--from-scratch"></a>Создание главного узла Kubernetes с нуля #
На этой странице описывается развертывание главного узла Kubernetes вручную от начала до конца.

Для этой операции требуется недавно обновленный компьютер под управлением Linux, например Ubuntu. ОС Windows не включена в контекст, все двоичные файлы компилируются в Linux.


> [!Warning]  
> Из-за изменчивости Kubernetes от версии к версии в этом руководстве могут применяться предположения, которые в будущем не будут справедливы.


## <a name="preparing-the-master"></a>Подготовка главного узла ##
Сначала установите все необходимые компоненты.

```bash
sudo apt-get install curl git build-essential docker.io conntrack python2.7
```

Если вы находитесь за прокси-сервером, определите переменные среды для текущего сеанса:
```bash
HTTP_PROXY=http://proxy.example.com:80/
HTTPS_PROXY=http://proxy.example.com:443/
http_proxy=http://proxy.example.com:80/
https_proxy=http://proxy.example.com:443/
```
Или, если вы хотите сделать этот параметр постоянным, добавьте переменные в раздел /etc/environment (для применения изменений необходимо выйти из системы и снова войти).

В [этом репозитории](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux) существует коллекция сценариев, которые помогают с процессом установки. Извлеките их в `~/kube/`. Этот каталог будет подключаться ко многим контейнерами Docker на последующих этапах, поэтому сохраните его структуру такой, как описано в этом руководстве.

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>Установка двоичных файлов Linux ###

> [!Note]  
> Чтобы включить исправления или использовать новый код Kubernetes вместо скачивания предварительно созданных двоичных файлов, посетите [эту страницу](./compiling-kubernetes-binaries.md).

Скачайте и установите официальные двоичные файлы Linux из [основной линии Kubernetes](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1) и установите их следующим образом.

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

Добавьте двоичные файлы в `$PATH`, чтобы их можно было запустить из любой точки. Обратите внимание, что при этом только устанавливается путь для сеанса. Добавьте эту строку в `~/.profile` для применения постоянного параметра.

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>Установка подключаемых модулей CNI ###
Для использования сети Kubernetes необходимы базовые подключаемые модули CNI. Скачайте их [здесь](https://github.com/containernetworking/plugins/releases) и извлеките в папку `/opt/cni/bin/`:

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>Сертификаты ###
Во-первых, получите локальный IP-адрес с помощью `ifconfig` или:

```bash
$ ip addr show dev eth0
```

, если имя интерфейса известно. Оно будет часто упоминаться во время этого процесса, если задать в его качестве значение переменной среды, все станет проще. Следующий фрагмент кода задает его временно. Если сеанс будет завершен или оболочка закроется, его потребуется задать снова.

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

Подготовьте сертификаты, которые будут использоваться для обмена данными в кластере:

```bash
cd ~/kube/certs
chmod u+x generate-certs.sh
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>Подготовка манифестов и надстроек ###
Создайте набор файлов YAML, указывающих системные модули pod Kubernetes, передав основной IP-адрес и *полный* CIDR кластера в сценарии Python в папке `manifest`:

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

Удалите сценарий Python, чтобы система Kubernetes не перепутала его с манифестом (это может привести к проблемам позднее).

> [!Important]  
> Если версия Kubernetes отличается от указанной в этом руководстве, используйте различные флаги управления версиями в сценарии (например, `--api-version`) для [настройки образа](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64), который развертывает модули pod. Не все манифесты используют один образ и имеют различные схемы управления версиями (в частности, `etcd` и диспетчер надстроек).


#### <a name="manifest-customization"></a>Настройка манифеста ####
На этом этапе может потребоваться внести изменения, связанные с установкой. Например, можно назначить подсети узлам вручную, а не использовать автоматическое управление Kubernetes. Для такой конфигурации доступен параметр в сценарии (см. описание параметра `--im-sure` см. в разделе `--help`):

```bash
./generate.py $MASTER_IP --im-sure
```

Для всех других параметров конфигурации потребуется вручную изменить созданные манифесты.


### <a name="configure--run-kubernetes"></a>Настройка и выполнение Kubernetes ###
Настройте Kubernetes для использования созданных сертификатов. Это приведет к созданию конфигурацию в каталоге `~/.kube/config`:

```bash
cd ~/kube
./configure-kubectl.sh $MASTER_IP
```

Теперь скопируйте файл, где его будут ожидать модули pod:

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

«Клиент» Kubernetes, `kubelet`, готов к запуску. Следующие сценарии обоих выполняются неопределенно долго. Откройте другой сеанс терминала после выполнения каждого из них, чтобы продолжить работу:

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

Выполните сценарий Kubeproxy, передав частичный CIDR кластера:

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> Это будет *полный* ожидаемый /16 CIDR, в который войдут узлы, *даже при наличии на этом CIDR трафика не от Kubernetes.* Kubeproxy применяется *только* к трафику Kubernetes для подсети *службы*, поэтому он не должен взаимодействовать с трафиком других узлов.

> [!Note]  
> Эти сценарии можно настроить для использования управляющей программы. В этом руководстве описывается только выполнение их вручную, так как это оптимальный вариант для перехвата ошибок во время установки.


## <a name="verifying-the-master"></a>Проверка главного узла ##
Через несколько минут система должна быть в следующем состоянии:

  - В разделе `docker ps` будет примерно 23 рабочих узлов и контейнеров pod.
  - Вызов `kubectl cluster-info` позволит отобразить сведения о сервере API главного узла Kubernetes в дополнение к надстройкам DNS и Heapster.
  - `ifconfig` отображает новый интерфейс `cbr0` с выбранного CIDR кластера.

