---
title: Присоединение узлов Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Windows к кластеру Kubernetes с помощью v 1.14.
keywords: kubernetes, 1,14, Windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910334"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Присоединение узлов Windows Server к кластеру #
После [настройки главного узла Kubernetes](./creating-a-linux-master.md) и [выбора нужного сетевого решения](./network-topologies.md)можно присоединить узлы Windows Server к созданию кластера. Для этого требуется выполнить некоторые [подготовительные действия на узлах Windows](#preparing-a-windows-node) перед присоединением.

## <a name="preparing-a-windows-node"></a>Подготовка узла Windows ##
> [!NOTE]  
> Все фрагменты кода в разделах для Windows должны выполняться в сеансе PowerShell с _повышенными привилегиями_.

### <a name="install-docker-requires-reboot"></a>Установка DOCKER (требуется перезагрузка) ###
Kubernetes использует [DOCKER](https://www.docker.com/) в качестве подсистемы контейнеров, поэтому необходимо установить его. Вы можете следовать [официальным инструкциям в документах](../manage-docker/configure-docker-daemon.md#install-docker), [инструкциям Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) или выполнить следующие действия.

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Если вы находитесь за прокси-сервером, необходимо определить следующие переменные среды PowerShell.
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

Если после перезагрузки появится следующая ошибка:

![текст](media/docker-svc-error.png)

Затем запустите службу DOCKER вручную:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Создание образа "Пауза" (инфраструктуры) ###
> [!Important]
> Важно соблюдать осторожность при конфликте образов контейнеров; отсутствие ожидаемого тега может вызвать `docker pull` несовместимого образа контейнера, что вызывает [проблемы развертывания](./common-problems.md#when-deploying-docker-containers-keep-restarting) , такие как неопределенное `ContainerCreating` состояния.

После установки `docker` необходимо подготовить образ «Пауза», используемый Kubernetes для подготовки модулей pod инфраструктуры. Вот три шага: 
  1. [Извлечение образа](#pull-the-image)
  2. [Добавление тегов](#tag-the-image) в качестве Microsoft/Server: Последняя версия
  3. и [запустите его](#run-the-container)


#### <a name="pull-the-image"></a>Извлечение образа ####     
 Извлекать образ для конкретного выпуска Windows. Например, при использовании Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Добавление тега к изображению ####
Файлы dockerfile, который будет использоваться далее в этом пошаговом окне, найдите тег образа `:latest`. Пометьте только что извлеченный образ сервера, как показано ниже.

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Запуск контейнера ####
Дважды проверьте, что контейнер действительно выполняется на компьютере:

```powershell
docker run microsoft/nanoserver:latest
```

Отобразится примерно следующее:

![текст](./media/docker-run-sample.png)

> [!tip]
> Если не удается запустить контейнер, см. раздел: [соответствующая версия узла контейнера с образом контейнера](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions) .


#### <a name="prepare-kubernetes-for-windows-directory"></a>Подготовка Kubernetes для каталога Windows ####
Создайте каталог "Kubernetes for Windows" для хранения двоичных файлов Kubernetes, а также все сценарии развертывания и файлы конфигурации.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Копирование сертификата Kubernetes #### 
Скопируйте файл сертификата Kubernetes (`$HOME/.kube/config`) [из Master](./creating-a-linux-master.md#collect-cluster-information) в этот новый каталог `C:\k`.

> [!tip]
> Для перемещения файла конфигурации между узлами можно использовать такие средства, как [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) или [WinSCP](https://winscp.net/eng/download.php) .

#### <a name="download-kubernetes-binaries"></a>Скачивание двоичных файлов Kubernetes ####
Чтобы иметь возможность запускать Kubernetes, сначала необходимо скачать `kubectl`, `kubelet`и `kube-proxy` двоичные файлы. Их можно загрузить по ссылкам в файле `CHANGELOG.md` [последних выпусках](https://github.com/kubernetes/kubernetes/releases/).
 - Например, ниже приведены [двоичные файлы узла 1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Чтобы извлечь архив и поместить двоичные файлы в `C:\k\`, используйте такой инструмент, как [expand-Archive](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) .

#### <a name="optional-setup-kubectl-on-windows"></a>Используемых Настройка kubectl в Windows ####
Если вы хотите управлять кластером из Windows, это можно сделать с помощью команды `kubectl`. Прежде всего, чтобы сделать `kubectl` доступным вне `C:\k\` каталога, измените переменную среды `PATH`:

```powershell
$env:Path += ";C:\k"
```

Чтобы сохранить это изменение, переменную следует изменить на целевом компьютере:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Далее мы проверим, является ли [сертификат кластера](#copy-kubernetes-certificate) действительным. Чтобы задать расположение, в котором `kubectl` ищет файл конфигурации, можно передать параметр `--kubeconfig` или изменить переменную среды `KUBECONFIG`. Например, если конфигурации находится в `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Чтобы сделать этот параметр постоянным в области текущего пользователя:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Наконец, чтобы проверить, правильно ли обнаружена конфигурация, можно использовать:

```powershell
kubectl config view
```

Если возникает ошибка подключения,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Необходимо дважды проверить расположение kubeconfig или повторить попытку копирования.

Если ошибки не отображаются, теперь узел готов к присоединению кластера.

## <a name="joining-the-windows-node"></a>Присоединение к узлу Windows ##
В зависимости от [выбранного сетевого решения](./network-topologies.md)можно:
1. [Присоединение узлов Windows Server к кластеру Фланнел (вкслан или Host-GW)](#joining-a-flannel-cluster)
2. [Присоединение узлов Windows Server к кластеру с помощью параметра ToR](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Присоединение к кластеру Фланнел ###
В [этом репозитории Майкрософт](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) имеется набор скриптов развертывания фланнел, которые помогают присоединить этот узел к кластеру.

Скачайте скрипт [Start. ps1 фланнел](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) , содержимое которого необходимо извлечь в `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Если вы [подготовили узел Windows](#preparing-a-windows-node), а `c:\k` Каталог выглядит следующим образом, вы можете присоединиться к узлу.

![текст](./media/flannel-directory.png)

#### <a name="join-node"></a>Соединение с узлом #### 
Чтобы упростить процесс присоединения к узлу Windows, достаточно запустить один сценарий Windows для запуска `kubelet`, `kube-proxy`, `flanneld`и присоединиться к узлу.

> [!Note]
> [Start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) ссылается на [install. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), который скачивает дополнительные файлы, такие как исполняемый файл `flanneld` и [модуль Dockerfile для инфраструктуры](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *, и устанавливает их*. Для наложения сетевого режима [брандмауэр](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) будет открыт для локального UDP-порта 4789. Может появиться несколько открытых или закрытых окон PowerShell, а также несколько секунд простоя сети, в то время как новый внешний vSwitch для сети Pod создается в первый раз.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# <a name="managementiptabmanagementip"></a>[манажементип](#tab/ManagementIP)
IP-адрес, назначенный узлу Windows. Чтобы найти это, можно использовать `ipconfig`.

|  |  | 
|---------|---------|
|Параметр     | `-ManagementIP`        |
|Значение по умолчанию    | н.а. **обязательный параметр**        |

# <a name="networkmodetabnetworkmode"></a>[нетворкмоде](#tab/NetworkMode)
Сетевой режим `l2bridge` (фланнел Host-GW) или `overlay` (фланнел вкслан) выбран в качестве [сетевого решения](./network-topologies.md).

> [!Important] 
> для `overlay` сетевого режима (фланнел вкслан) требуются двоичные файлы Kubernetes v 1.14 (или более поздней версии) и [KB4489899](https://support.microsoft.com/help/4489899).

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


# <a name="servicecidrtabservicecidr"></a>[сервицеЦидр](#tab/ServiceCIDR)
[Диапазон подсети службы](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  | 
|---------|---------|
|Параметр     | `-ServiceCIDR`        |
|Значение по умолчанию    | `10.96.0.0/12`        |


# <a name="kubednsserviceiptabkubednsserviceip"></a>[кубеднссервицеип](#tab/KubeDnsServiceIP)
[IP-адрес службы DNS Kubernetes](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|Параметр     | `-KubeDnsServiceIP`        |
|Значение по умолчанию    | `10.96.0.10`        |


# <a name="interfacenametabinterfacename"></a>[InterfaceName](#tab/InterfaceName)
Имя сетевого интерфейса узла Windows. Чтобы найти это, можно использовать `ipconfig`.

|  |  | 
|---------|---------|
|Параметр     | `-InterfaceName`        |
|Значение по умолчанию    | `Ethernet`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
Каталог, в котором журналы kubelet и KUBE-proxy перенаправляются в соответствующие выходные файлы.

|  |  | 
|---------|---------|
|Параметр     | `-LogDir`        |
|Значение по умолчанию    | `C:\k`        |


---

> [!tip]
> Вы уже заотмечали подсеть кластера, подсеть службы и IP-адрес KUBE-DNS из [главного мастера](./creating-a-linux-master.md#collect-cluster-information) Linux.

После выполнения этой команды вы сможете:
  * Просмотр присоединенных узлов Windows с помощью `kubectl get nodes`
  * См. 3 окна PowerShell Open, одно для `kubelet`, одно для `flanneld`и другое для `kube-proxy`
  * См. раздел процессы агента узла для `flanneld`, `kubelet`и `kube-proxy`, выполняющихся на узле.

В случае успеха перейдите к [следующим шагам](#next-steps).

## <a name="joining-a-tor-cluster"></a>Присоединение к кластеру ToR ##
> [!NOTE]
> Вы можете пропустить этот раздел, если [ранее](./network-topologies.md#flannel-in-host-gateway-mode)вы выбрали фланнел в качестве сетевого решения.

Для этого необходимо выполнить инструкции по [настройке контейнеров Windows Server в Kubernetes для вышестоящей топологии маршрутизации L3](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Это включает в себя настройку вышестоящего маршрутизатора таким путем, чтобы префикс CIDR для Pod, назначенный узлу, сопоставляется с соответствующим IP-адресом узла.

Если новый узел указан как "готовый" `kubectl get nodes`, kubelet + KUBE-proxy работает, и вы настроили вышестоящий маршрутизатор ToR, вы готовы к дальнейшим действиям.

## <a name="next-steps"></a>Дальнейшие действия ##
В этом разделе мы рассмотрели, как присоединить рабочие роли Windows к нашему кластеру Kubernetes. Теперь все готово к шагу 5.

> [!div class="nextstepaction"]
> [Присоединение к рабочим работникам Linux](./joining-linux-workers.md)

Кроме того, если у вас нет ни одной из рабочих ролей Linux, вы можете перейти к шагу 6:

> [!div class="nextstepaction"]
> [Развертывание ресурсов Kubernetes](./deploying-resources.md)
