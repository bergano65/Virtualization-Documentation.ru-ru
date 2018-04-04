---
title: Kubernetes в Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: Присоединение узла Windows к кластеру Kubernetes с бета-версией 1.9.
keywords: kubernetes, 1.9, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 124895e93cbaee50c66b6b5a7cc2c71c144dad67
ms.sourcegitcommit: 6e3c3b2ff125f949c03a342c3709a6e57c5f736c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/17/2018
---
# <a name="kubernetes-on-windows"></a>Kubernetes в Windows #

После выхода версии Kubernetes 1.9 и Windows Server [версии 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking) пользователи могут воспользоваться преимуществами самых новых сетевых компонентов Windows.

  - **Общие секции модулей pod**: модули pod инфраструктуры и рабочих узлов теперь совместно используют сетевую секцию (аналог пространства имен Linux).
  - **Оптимизация конечных точек**. Благодаря совместному использованию секций службам контейнеров требуется отслеживать в два раза меньше конечных точек, чем раньше.
  - **Оптимизация пути данных**. Улучшения платформа виртуальной фильтрации и сетевой службы узлов позволяют реализовать балансировку нагрузки на основе ядра.


Эта страница служит руководством по присоединению нового узла Windows к существующему кластеру на основе Linux. Чтобы полностью начать с нуля, посетите [эту страницу](./creating-a-linux-master.md) &mdash; один из многих ресурсов, доступных для развертывания кластера Kubernetes &mdash; чтобы узнать, как настроить мастер с нуля так же, как мы.

<a name="definitions"></a> Ниже перечислены определения некоторых терминов, упомянутых в этом руководстве.

  - **Внешняя сеть**— это сеть, в рамках которой взаимодействуют ваши узлы.
  - <a name="cluster-subnet-def"></a>**Подсеть кластера**— это маршрутизируемая виртуальная сеть. Узлам назначаются более мелкие подсети из модулей pod.
  - **Подсеть службы**— это немаршрутизируемая чисто виртуальная подсеть в сегменте 11.0/16, которую модули pod используют для доступа к службам независимо от топологии сети. Она преобразуется в маршрутизируемое адресное пространство или из адресного пространства при выполнении `kube-proxy` на узлах.

## <a name="what-you-will-accomplish"></a>Цели ##

В рамках этого руководства вы выполните следующие действия.

> [!div class="checklist"]  
> * Настроим [главный узел Linux](#preparing-the-linux-master).  
> * Присоединим к нему [рабочий узел Windows](#preparing-a-windows-node).  
> * Подготовим [топологию сети](#network-topology).  
> * Развернем [образец службы Windows](#running-a-sample-service).  
> * Рассмотрим [распространенные проблемы и ошибки](./common-problems.md).  

## <a name="preparing-the-linux-master"></a>Подготовка главного узла Linux ##

Независимо от того,выполнили ли вы [инструкции](./creating-a-linux-master.md) или уже используете существующий кластер, единственное, что требуется от главного узла Linux— конфигурация сертификата Kubernetes. Она может находится в папке `/etc/kubernetes/admin.conf`, `~/.kube/config`, или в другом месте, в зависимости от настроек.

## <a name="preparing-a-windows-node"></a>Подготовка узла Windows ##

> [!NOTE]  
> Все фрагменты кода в разделах для Windows должны выполняться в сеансе PowerShell с _повышенными привилегиями_.

Kubernetes использует [Docker](https://www.docker.com/) как оркестратор контейнера, поэтому нам необходимо его установить. Вы можете следовать [официальным инструкциям в документах](../manage-docker/configure-docker-daemon.md#install-docker), [инструкциям Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) или выполнить следующие действия.

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

[В этом репозитории Майкрософт](https://github.com/Microsoft/SDN) существует набор сценариев, которые помогут вам присоединить этот узел к кластеру. Скачать ZIP-файл напрямую можно [здесь](https://github.com/Microsoft/SDN/archive/master.zip). Единственное, что вам нужно— это папка `Kubernetes/windows`, содержимое которой должно быть перемещено в `C:\k\`.

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

Скопируйте файл сертификата, [определенный ранее](#preparing-the-linux-master), в этот новый каталог `C:\k`.

## <a name="network-topology"></a>Топология сети ##

Существует несколько способов сделать виртуальную [подсеть кластера](#cluster-subnet-def) маршрутизируемой. Можно сделать следующее.

  - Настроить [режим узел-шлюз](./configuring-host-gateway-mode.md), задав статические маршруты следующего прыжка между узлами для обеспечения взаимодействия модулей pod.
  - Настроить интеллектуальный коммутатор верхнего уровня (ToR) для маршрутизации подсети.
  - Использовать подключаемый модуль наложения стороннего поставщика, например [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (поддержка Flannel в Windows находится на стадии бета-версии).

### <a name="creating-the-pause-image"></a>Создание образа «Пауза» ###

После установки `docker` необходимо подготовить образ «Пауза», используемый Kubernetes для подготовки модулей pod инфраструктуры.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!NOTE]
> Добавляется тег `:latest`, так как пример службы, которую вы развернете позднее, зависит от него, хотя это может _быть_ не последний доступный образ Windows Server Core. Следует быть осторожным и не допускать конфликтующих образов контейнеров. При отсутствии ожидаемого тега выполнение операции `docker pull` с несовместимым образом контейнера может привести к [проблемам с развертыванием](./common-problems.md#when-deploying-docker-containers-keep-restarting). 


### <a name="downloading-binaries"></a>Скачивание двоичных файлов ###
Когда происходит операция `pull`, скачайте следующие клиентские двоичные файлы из Kubernetes:

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

Вы можете скачать их по ссылкам в файле `CHANGELOG.md` последнего выпуска 1.9. На момент написания этой статьи была доступа версия [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1), а двоичные файлы Windows доступны [здесь](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz). Используйте архиватор, например [7-Zip](http://www.7-zip.org/), чтобы извлечь содержимое архива и разместить двоичные файлы в `C:\k\`.

Чтобы команда `kubectl` была доступна за пределами каталога `C:\k\`, измените переменную среды `PATH`:

```powershell
$env:Path += ";C:\k"
```

Чтобы сохранить это изменение, переменную следует изменить на целевом компьютере:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

### <a name="joining-the-cluster"></a>Присоединение к кластеру ###
Убедитесь, что конфигурация кластера допустима, с помощью следующей команды:

```powershell
kubectl version
```

Если возникает ошибка подключения,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

убедитесь, что конфигурация обнаружена правильно:

```powershell
kubectl config view
```

Чтобы изменить расположение, где `kubectl` выполняет поиск файла конфигурации, можно передать параметр `--kubeconfig` или изменить переменную среды `KUBECONFIG`. Например, если конфигурации находится в `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Чтобы сделать этот параметр постоянным в области текущего пользователя:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Теперь узел готов для присоединения к кластеру. Выполните эти сценарии (в указанном порядке) в двух отдельных окнах PowerShell *с повышенными привилегиями*. Параметр `-ClusterCidr` в первом сценарии предназначен для настроенной [подсети кластера](#cluster-subnet-def); здесь ее адрес— `192.168.0.0/16`.

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

Узел Windows будет доступен с главного узла Linux в `kubectl get nodes` в течение минуты!


### <a name="validating-your-network-topology"></a>Проверка топологии сети ###

Существует несколько основных тестов для проверки правильности конфигурации сети.

  - **Подключение между узлами**. Проверяется связь между главным узлом и рабочими узлами Windows, тест должен завершиться успешно в обоих направлениях.

  - **Подключение подсети модулей pod к узлам**. Проверка связи между виртуальным интерфейсом pod и узлами. Поиск адреса шлюза в `route -n` и `ipconfig` в Linux и Windows соответственно, поиск интерфейса `cbr0`.

Если какие-либо из этих базовых тестов не работают, посетите [страницу устранения неполадок](./common-problems.md#common-networking-errors).


## <a name="running-a-sample-service"></a>Запуск образца службы ##

Вы развернете очень простую [веб-службу на основе PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) для проверки присоединения к кластеру и правильности настройки нашей сети.

На главном узле Linux скачайте и запустите службу:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

При этом создается развертывание и служба. Модули pod будут неопределенное время отслеживать их состояние— просто нажмите `Ctrl+C` для выхода из команды `watch`, когда завершите наблюдение.

Если все прошло нормально:

  - вы видите четыре контейнера при выполнении команды `docker ps` на стороне Windows;
  - `curl` IP-адреса *модуля pod* на порту 80 от главного узла Linux получают ответ от веб-сервера; это подтверждает взаимодействие узлов с модулем pod в сети;
  - `curl` IP-адрес *узла* на порту 4444 получает ответ веб-сервера; этот демонстрирует правильное сопоставление портов контейнера и узла;
  - проверка связи *между модулями pod* (в том числе между узлами, если у вас несколько узлов Windows) с помощью `docker exec` выполняется успешно; это подтверждает правильную настройку взаимодействия модулей pod;
  - `curl` в разделе `kubectl get services` отображается виртуальный *IP-адрес службы* от главного узла Linux и отдельных модулей pod;
  - `curl` отображается *имя службы* с [DNS-суффиксом Kubernetes по умолчанию](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), что подтверждает работу DNS.

> [!WARNING]  
> У узлов Windows нет доступа к IP-адресу службы. Это [известное ограничение платформы](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip), которое будет устранено в будущем.
