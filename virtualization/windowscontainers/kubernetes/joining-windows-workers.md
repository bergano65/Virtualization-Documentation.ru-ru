---
title: Присоединение узла Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Windows к кластеру Kubernetes с v1.14.
keywords: kubernetes, 1.14, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c1781a6ce48ebaa8433f5649a34ac79b852beae6
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622969"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Присоединение к кластеру узлов Windows Server #
После [настройки Kubernetes главном узле](./creating-a-linux-master.md) и [выбрать нужную сеть решение](./network-topologies.md), вы готовы присоединиться к узлам Windows Server для формирования кластера. Это требуется некоторая [подготовки на узлах Windows](#preparing-a-windows-node) до присоединения.

## <a name="preparing-a-windows-node"></a>Подготовка узла Windows ##
> [!NOTE]  
> Все фрагменты кода в разделах для Windows должны выполняться в сеансе PowerShell с _повышенными привилегиями_.

### <a name="install-docker-requires-reboot"></a>Установка Docker (требуется перезагрузка) ###
Kubernetes использует [Docker](https://www.docker.com/) в качестве обработчика его контейнера, поэтому нам необходимо установить его. Вы можете следовать [официальным инструкциям в документах](../manage-docker/configure-docker-daemon.md#install-docker), [инструкциям Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) или выполнить следующие действия.

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

Если после перезагрузки см. следующую ошибку:

![текст](media/docker-svc-error.png)

Затем вручную запустите службу docker:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Создание образа «Пауза» (инфраструктура) ###
> [!Important]
> Это следует быть осторожным конфликтующих образов контейнеров; не отсутствии ожидаемого тега может привести к `docker pull` несовместимым образом контейнера изображения, что приводит к [проблемам с развертыванием](./common-problems.md#when-deploying-docker-containers-keep-restarting) таких как долгую `ContainerCreating` состояние.

После установки `docker` необходимо подготовить образ «Пауза», используемый Kubernetes для подготовки модулей pod инфраструктуры. Существует три шага: 
  1. [Получение изображения](#pull-the-image)
  2. [теги его](#tag-the-image) как microsoft / nanoserver:latest
  3. и [его выполнения](#run-the-container)


#### <a name="pull-the-image"></a>Извлечь образ ####     
 Извлечь образ для вашего конкретного выпуска Windows. Например, если вы используете Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Тег изображения ####
Поиск файлов Dockerfile, вы будете использовать позже в этом руководстве `:latest` изображение тега. Тег nanoserver изображение, которое вы только что полученное следующим образом:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Запуск контейнера ####
Еще раз проверьте, что контейнер фактически выполняется на компьютере:

```powershell
docker run microsoft/nanoserver:latest
```

Вы увидите примерно так:

![текст](./media/docker-run-sample.png)

> [!tip]
> Если вы не можете запустить контейнер сведения см. в разделе: [соответствующей версии узла контейнера в образ контейнера](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Подготовка каталога Kubernetes для Windows ####
Создайте каталог «Kubernetes для Windows» для хранения двоичных файлов Kubernetes, а также все сценарии развертывания и файлы конфигурации.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Копирование сертификата Kubernetes #### 
Скопируйте файл сертификата Kubernetes (`$HOME/.kube/config`) [от главного узла](./creating-a-linux-master.md#collect-cluster-information) в новый метод `C:\k` каталога.

> [!tip]
> Средства, такие как [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) или [WinSCP](https://winscp.net/eng/download.php) можно использовать для перемещения между узлами в файле конфигурации.

#### <a name="download-kubernetes-binaries"></a>Скачать двоичных файлов Kubernetes ####
Чтобы иметь возможность выполнить Kubernetes, сначала необходимо загрузить `kubectl`, `kubelet`, и `kube-proxy` двоичных файлов. Вы можете скачать их по ссылкам в `CHANGELOG.md` файл [последних выпусках](https://github.com/kubernetes/kubernetes/releases/).
 - Например вот [v1.14 двоичные файлы узла](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Используйте средство, например [Expand архива](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) извлечь содержимое архива и разместить двоичные файлы в `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>(Необязательно) Настройка kubectl в Windows ####
Следует требуется управлять кластера с Windows, можно сделать с помощью `kubectl` команды. Во-первых, чтобы сделать `kubectl` доступна за пределами `C:\k\` каталога, измените `PATH` переменной среды:

```powershell
$env:Path += ";C:\k"
```

Чтобы сохранить это изменение, переменную следует изменить на целевом компьютере:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Затем мы позволит убедиться, что [кластер сертификат](#copy-kubernetes-certificate) действителен. Чтобы задать местоположение где `kubectl` выполняет поиск файла конфигурации, которые можно передавать `--kubeconfig` параметра или изменить `KUBECONFIG` переменной среды. Например, если конфигурации находится в `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Чтобы сделать этот параметр постоянным в области текущего пользователя:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Наконец Чтобы проверить, если конфигурация обнаружена правильно, можно использовать:

```powershell
kubectl config view
```

Если возникает ошибка подключения,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Следует еще раз проверьте расположение kubeconfig или попытайтесь скопировать его еще раз.

Если вы видите без ошибок узел готов для присоединения к кластеру.

## <a name="joining-the-windows-node"></a>Присоединение узла Windows ##
В зависимости от [сетевых решение, которое вы выбрали](./network-topologies.md)вы можете:
1. [Присоединитесь к узлам Windows Server в кластере Flannel (инкапсуляция или gw узла)](#joining-a-flannel-cluster)
2. [Присоединиться к Windows Server узлы в кластер с помощью коммутатора верхнего уровня](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Присоединение к кластеру Flannel ###
Существует набор сценариев развертывания Flannel в [этом репозитории Майкрософт](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) , помогут вам присоединить этот узел к кластеру.

Загрузите сценарий [Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) , содержимое которой Извлеките в `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Допустим, [подготовленные вашего узла Windows](#preparing-a-windows-node)и `c:\k` каталога выглядит следующим образом, вы будете готовы присоединение узла.

![текст](./media/flannel-directory.png)

#### <a name="join-node"></a>Присоединение узла #### 
Чтобы упростить процесс присоединение узла Windows, вам нужно выполнить один сценарий Windows для запуска `kubelet`, `kube-proxy`, `flanneld`и ее узел.

> [!Note]
> [Start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) ссылается на [install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), который будет загрузить дополнительные файлы, такие как `flanneld` исполняемый файл и [файл Dockerfile для pod инфраструктуры](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *и установить их для вас*. Для режим сети наложения будет открыт [брандмауэра](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) для локального порта UDP 4789. Могут существовать несколько windows powershell, возможность открытия и закрытия а также несколько секунд отказа сети, когда создается новый внешний виртуальный коммутатор для сети pod, впервые.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
IP-адрес узла Windows. Вы можете использовать `ipconfig` найти это.

|  |  | 
|---------|---------|
|Параметр     | `-ManagementIP`        |
|Значение по умолчанию    | n.A. **Обязательный параметр**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
Сетевой режим `l2bridge` (flannel gw узла) или `overlay` (flannel инкапсуляция) выбран в качестве [решения сети](./network-topologies.md).

> [!Important] 
> `overlay` режим сети (flannel инкапсуляция) требует двоичных файлов Kubernetes v1.14 (или выше) и [KB4489899](https://support.microsoft.com/help/4489899).

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


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
[Служба диапазону адресов подсети](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  | 
|---------|---------|
|Параметр     | `-ServiceCIDR`        |
|Значение по умолчанию    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
, [IP-адрес службы Kubernetes DNS](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|Параметр     | `-KubeDnsServiceIP`        |
|Значение по умолчанию    | `10.96.0.10`        |


# [<a name="interfacename"></a>Имя интерфейса](#tab/InterfaceName)
Имя сетевого интерфейса узла Windows. Вы можете использовать `ipconfig` найти это.

|  |  | 
|---------|---------|
|Параметр     | `-InterfaceName`        |
|Значение по умолчанию    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Каталог, где перенаправления журналы kubelet и помощью kube прокси в их соответствующие выходные файлы.

|  |  | 
|---------|---------|
|Параметр     | `-LogDir`        |
|Значение по умолчанию    | `C:\k`        |


---

> [!tip]
> Вы уже отмечалось вниз подсети кластера, подсеть службы и IP-адрес помощью kube-DNS от главного узла Linux [более ранних версий](./creating-a-linux-master.md#collect-cluster-information)

После выполнения этого вы сможете:
  * Представления к домену с помощью узлов Windows `kubectl get nodes`
  * См. в разделе 3 windows powershell открыть, один для `kubelet`, один для `flanneld`, а другая — для `kube-proxy`
  * См. в разделе процессы узла агента `flanneld`, `kubelet`, и `kube-proxy` на узле

В случае успеха, переходите к [дальнейшие действия](#next-steps).

## <a name="joining-a-tor-cluster"></a>Присоединение к кластеру верхнего уровня ##
> [!NOTE]
> Этот раздел можно пропустить, если вы выбрали Flannel как вашей сети решение [ранее](./network-topologies.md#flannel-in-host-gateway-mode).

Для этого необходимо следуйте инструкциям по [настройке контейнеры Windows Server в Kubernetes для топологии маршрутизации вышестоящего уровня 3](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Сюда входят, чтобы убедиться, что вы настроите маршрутизатор несоответствий так, чтобы префикс модуля CIDR назначены узла соответствует его IP-адрес соответствующего узла.

При условии нового узла отображается как «Готово» `kubectl get nodes`, выполняется kubelet + помощью kube прокси и вы настроили маршрутизатор несоответствий верхнего уровня, вы будете готовы для выполнения следующих действий.

## <a name="next-steps"></a>Дальнейшие действия ##
В этом разделе мы рассматривается как присоединить устройство Windows сотрудников к нашей кластера Kubernetes. Теперь все готово для шага 5:

> [!div class="nextstepaction"]
> [Присоединение работников Linux](./joining-linux-workers.md)

Кроме того Если у вас нет все сотрудники Linux вы можете пропустить шаг 6:

> [!div class="nextstepaction"]
> [Развертывание ресурсов Kubernetes](./deploying-resources.md)
