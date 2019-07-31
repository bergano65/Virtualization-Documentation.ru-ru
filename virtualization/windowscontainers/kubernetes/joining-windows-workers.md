---
title: Присоединение к узлам Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Windows к кластеру Кубернетес с v 1.14.
keywords: кубернетес, 1,14, Windows, Приступая к работе
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882987"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Присоединение к кластеру узлов Windows Server #
После [настройки главного узла кубернетес](./creating-a-linux-master.md) и [выбора нужного сетевого решения](./network-topologies.md)вы можете присоединиться к узлу Windows Server для создания кластера. Для этого требуется предварительная [Подготовка на узлах Windows](#preparing-a-windows-node) перед присоединением.

## <a name="preparing-a-windows-node"></a>Подготовка узла Windows ##
> [!NOTE]  
> Все фрагменты кода в разделах для Windows должны выполняться в сеансе PowerShell с _повышенными привилегиями_.

### <a name="install-docker-requires-reboot"></a>Установка стыковочного узла (требуется перезагрузка) ###
Кубернетес использует [Dock](https://www.docker.com/) в качестве подсистемы контейнеров, поэтому нам нужно установить его. Вы можете следовать [официальным инструкциям в документах](../manage-docker/configure-docker-daemon.md#install-docker), [инструкциям Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) или выполнить следующие действия.

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

Если после перезагрузки появляется следующее сообщение об ошибке:

![текст](media/docker-svc-error.png)

Затем запустите службу Dock вручную:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Создание изображения "Пауза" (инфраструктуры) ###
> [!Important]
> Важно соблюдать осторожность при конфликте с изображениями контейнера; отсутствие ожидаемого тега может привести к созданию `docker pull` несовместимого изображения контейнера, которое вызывает [проблемы развертывания](./common-problems.md#when-deploying-docker-containers-keep-restarting) , такие как неопределенное `ContainerCreating` состояние.

После установки `docker` необходимо подготовить образ «Пауза», используемый Kubernetes для подготовки модулей pod инфраструктуры. Это можно сделать тремя шагами: 
  1. [Извлечение изображения](#pull-the-image)
  2. [расстановка тегов](#tag-the-image) в качестве Microsoft/сервера: последние
  3. и [запустите его](#run-the-container)


#### <a name="pull-the-image"></a>Извлечение изображения ####     
 Вытяните изображение для конкретного выпуска Windows. Например, если вы используете Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Добавление тега к изображению ####
Доккерфилес, который будет использоваться позже в этом руководстве, найдите тег `:latest` Image. Пометьте только что извлеченный образ сервера.

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Запуск контейнера ####
Убедитесь в том, что контейнер действительно выполняется на вашем компьютере:

```powershell
docker run microsoft/nanoserver:latest
```

Вы увидите нечто вроде:

![текст](./media/docker-run-sample.png)

> [!tip]
> Если вы не можете выполнить этот контейнер, ознакомьтесь со сведениями: [соответствие версии узла контейнера с изображением контейнера](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Подготовка Кубернетес к каталогу Windows ####
Создайте каталог "Кубернетес для Windows" для хранения двоичных файлов Кубернетес, а также любых сценариев развертывания и файлов конфигурации.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Копирование сертификата Кубернетес #### 
Скопируйте файл сертификата кубернетес (`$HOME/.kube/config`) [из образца](./creating-a-linux-master.md#collect-cluster-information) в этот новый `C:\k` каталог.

> [!tip]
> Вы можете использовать такие инструменты, как [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) или [винскп](https://winscp.net/eng/download.php) , для перемещения файла конфигурации между узлами.

#### <a name="download-kubernetes-binaries"></a>Скачивание двоичных файлов Кубернетес ####
Чтобы запустить кубернетес, сначала необходимо скачать `kubectl` `kubelet`и `kube-proxy` двоичные файлы. Вы можете скачать их из ссылок в `CHANGELOG.md` файле из последних выпусков. [](https://github.com/kubernetes/kubernetes/releases/)
 - Например, ниже приведены двоичные [файлы узла v 1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Чтобы извлечь архив и поместить в него двоичные файлы, используйте такие средства, как `C:\k\` [расширение-Архив](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) .

#### <a name="optional-setup-kubectl-on-windows"></a>Необязательно Настройка кубектл в Windows ####
Если вы хотите управлять кластером из Windows, это можно сделать с помощью `kubectl` команды. Прежде всего, чтобы `kubectl` сделать его доступным за `C:\k\` пределами каталога `PATH` , измените переменную Environment.

```powershell
$env:Path += ";C:\k"
```

Чтобы сохранить это изменение, переменную следует изменить на целевом компьютере:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Затем мы проверяем, что [сертификат кластера](#copy-kubernetes-certificate) действителен. Чтобы указать расположение для поиска `kubectl` файла конфигурации, можно передать `--kubeconfig` параметр или изменить переменную `KUBECONFIG` среды. Например, если конфигурации находится в `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Чтобы сделать этот параметр постоянным в области текущего пользователя:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Наконец, чтобы проверить, правильно ли обнаружены конфигурации, вы можете использовать следующие возможности.

```powershell
kubectl config view
```

Если возникает ошибка подключения,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Необходимо дважды проверить расположение кубеконфиг или повторить попытку копирования.

Если вы не видите никаких ошибок, узел будет готов к присоединению к кластеру.

## <a name="joining-the-windows-node"></a>Присоединение к узлу Windows ##
В зависимости от того, какое [сетевое решение вы выбрали](./network-topologies.md), вы можете:
1. [Присоединение к узлам Windows Server в кластере Фланнел (вкслан или Host-GW)](#joining-a-flannel-cluster)
2. [Присоединение к кластеру узлов Windows Server с помощью переключателя Тор](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Присоединение к кластеру Фланнел ###
В [этом репозитории Майкрософт](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) есть набор сценариев развертывания фланнел, которые помогают присоединить этот узел к кластеру.

Скачайте сценарий [Start. ps1 фланнел](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) , содержимое которого нужно извлечь в `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Если вы [подготовили ваш узел Windows](#preparing-a-windows-node), `c:\k` и ваш каталог выглядит так, как показано ниже, вы можете присоединиться к этому узлу.

![текст](./media/flannel-directory.png)

#### <a name="join-node"></a>Присоединение к узлу #### 
Чтобы упростить процесс присоединения к узлу Windows, необходимо выполнить только один сценарий Windows для запуска `kubelet`, `kube-proxy` `flanneld`а также присоединиться к этому узлу.

> [!Note]
> файл [Start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) содержит ссылки [install. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), в котором загружаются дополнительные `flanneld` файлы, такие как исполняемый объект и [модуль доккерфиле для инфраструктуры](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *, и устанавливаются нужные данные*. Для режима перекрытия сети [брандмауэр](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) будет открыт для ЛОКАЛЬНОГО порта UDP 4789. В некоторых случаях, когда новая внешняя система vSwitch для сети Pod создается в первый раз, может быть открыто и закрыто несколько окон PowerShell, а также несколько секунд сбоя сети.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>Манажементип](#tab/ManagementIP)
IP-адрес, назначенный узлу Windows. Вы можете использовать `ipconfig` для поиска.

|  |  | 
|---------|---------|
|Параметр     | `-ManagementIP`        |
|Значение по умолчанию    | Н.а. **Обязательный параметр**        |

# [<a name="networkmode"></a>Нетворкмоде](#tab/NetworkMode)
Сетевой режим `l2bridge` (фланнел Host-GW) или `overlay` (фланнел вкслан), выбранный в качестве [сетевого решения](./network-topologies.md).

> [!Important] 
> `overlay` режим сети (фланнел вкслан) требует Кубернетес v 1.14 двоичных файлов (или выше) и [KB4489899](https://support.microsoft.com/help/4489899).

|  |  | 
|---------|---------|
|Параметр     | `-NetworkMode`        |
|Значение по умолчанию    | `l2bridge`        |


# [<a name="clustercidr"></a>КлустерЦидр](#tab/ClusterCIDR)
[Диапазон подсети кластера](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Параметр     | `-ClusterCIDR`        |
|Значение по умолчанию    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>СервицеЦидр](#tab/ServiceCIDR)
[Диапазон подсети службы](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  | 
|---------|---------|
|Параметр     | `-ServiceCIDR`        |
|Значение по умолчанию    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>Кубеднссервицеип](#tab/KubeDnsServiceIP)
[IP-адрес службы DNS кубернетес](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|Параметр     | `-KubeDnsServiceIP`        |
|Значение по умолчанию    | `10.96.0.10`        |


# [<a name="interfacename"></a>Интерфаценаме](#tab/InterfaceName)
Имя сетевого интерфейса узла Windows. Вы можете использовать `ipconfig` для поиска.

|  |  | 
|---------|---------|
|Параметр     | `-InterfaceName`        |
|Значение по умолчанию    | `Ethernet`        |


# [<a name="logdir"></a>Логдир](#tab/LogDir)
Каталог, в котором журналы кубелет и Кубе-proxy, перенаправляется в соответствующие выходные файлы.

|  |  | 
|---------|---------|
|Параметр     | `-LogDir`        |
|Значение по умолчанию    | `C:\k`        |


---

> [!tip]
> Вы уже заговорили подсеть кластера, подсеть службы и IP-адрес Кубе-DNS из образца Linux [более ранней версии](./creating-a-linux-master.md#collect-cluster-information)

После выполнения этой команды вы можете:
  * Просмотр подключенных узлов Windows с помощью `kubectl get nodes`
  * Ознакомьтесь с тремя окнами PowerShell Open, One `kubelet`для них `flanneld`, а также другим для `kube-proxy`
  * Просмотр процессов `flanneld` `kubelet`агента узла и `kube-proxy` выполнения на нем

При успешном завершении переходите к [следующим действиям](#next-steps).

## <a name="joining-a-tor-cluster"></a>Присоединение к кластеру Тор ##
> [!NOTE]
> Вы можете пропустить этот раздел, если вы [ранее](./network-topologies.md#flannel-in-host-gateway-mode)выбрали фланнел в качестве сетевого решения.

Для этого вам потребуется выполнить инструкции по [настройке контейнеров Windows Server на кубернетес для восходящего топологии маршрутизации на L3](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Сюда входит настройка маршрутизатора для восходящего потока, так что префикс CIDR для Pod, назначаемый узлу, сопоставлен IP-адресу соответствующего узла.

Если новый узел указан как "Готово" `kubectl get nodes`, кубелет + Кубе-proxy запущен, и вы настроили вышестоящий маршрутизатор Тор, вы можете приступать к следующим действиям.

## <a name="next-steps"></a>Дальнейшие действия ##
В этом разделе мы посмотрели, как присоединиться к сотрудникам Windows в нашем кластере Кубернетес. Теперь вы готовы к действию 5.

> [!div class="nextstepaction"]
> [Присоединение к сотрудникам Linux](./joining-linux-workers.md)

Кроме того, если у вас нет ни одного сотрудника Linux, вы можете перейти к действию 6.

> [!div class="nextstepaction"]
> [Развертывание ресурсов Кубернетес](./deploying-resources.md)
