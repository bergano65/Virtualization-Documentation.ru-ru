---
title: "Сетевые подключения контейнеров Windows"
description: "Настройка сетевого взаимодействия для контейнеров Windows."
keywords: "docker, контейнеры"
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 70e662e73693d9fef9d36635289ec9affb8d0331
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/01/2018
---
# <a name="windows-container-networking"></a>Сетевые подключения контейнеров Windows
> ***Сведения об основных сетевых командах, параметрах и синтаксисе Docker см. в разделе [Сетевые подключения контейнеров Docker](https://docs.docker.com/engine/userguide/networking/).*** За исключением случаев, описанных в этом документе, выполнение всех сетевых команд Docker поддерживается в Windows с помощью того же синтаксиса, что и в Linux. Однако обратите внимание, что сетевые стеки Windows и Linux отличаются друг от друга, и таким образом вы обнаружите, что некоторые сетевые команды Linux (например ifconfig) не поддерживаются в Windows.

## <a name="basic-networking-architecture"></a>Базовая сетевая архитектура
В этом разделе приведено описание того, каким образом Docker создает и администрирует сети в Windows. В отношении сетевых подключений контейнеры Windows функционируют аналогично виртуальным машинам. У каждого контейнера есть виртуальный сетевой адаптер (vNIC), который подключен к виртуальному коммутатору Hyper-V (vSwitch). Операционная система Windows поддерживает пять различных сетевых драйверов (или по-другому "режимов"), которые можно создать через Docker: *nat*, *overlay*, *transparent*, *l2bridge*, и *l2tunnel*. В зависимости от особенностей физической сетевой инфраструктуры и от того, должна ли сеть быть одно- или многоузловой, необходимо выбрать сетевой драйвер, который наилучшим образом соответствует вашим потребностям.

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

При первом запуске подсистемы Docker происходит создание сети NAT по умолчанию ("nat"), которая использует внутренний виртуальный коммутатор и компонент Windows под названием `WinNAT`. При наличии на узле других внешних виртуальных коммутаторов, созданных с помощью оболочки командной строки PowerShell или диспетчера Hyper-V, они также будут доступны подсистеме Docker посредством сетевого драйвера *transparent* и будут отображаться при запуске команды ``docker network ls``.  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - ***Внутренний*** виртуальный коммутатор— это коммутатор, который не подключен напрямую к сетевому адаптеру на узле контейнера 

> - ***Внешний*** виртуальный коммутатор— это коммутатор, который _подключен_ напрямую к сетевому адаптеру на узле контейнера  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

Сеть "nat"— это сеть по умолчанию для контейнеров под управлением Windows. Все контейнеры, которые работают под управлением Windows без каких-либо флагов или аргументов, использующихся для реализации конкретной сетевой конфигурации, будут подключены к сети "nat" по умолчанию и им будет автоматически присвоен IP-адрес из диапазона IP-адресов внутреннего префикса сети "nat". Для "nat" по умолчанию используется следующий внутренний префикс IP-адреса: 172.16.0.0/16. 


## <a name="windows-container-network-drivers"></a>Сетевые драйверы контейнеров Windows  

Помимо использования сети "nat" по умолчанию, созданной подсистемой Docker в Windows, пользователи могут определять собственные сети контейнеров. Пользовательские сети можно создавать в интерфейсе командной строки Docker (команда [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/)). В Windows доступны следующие типы сетевых драйверов.

- **nat**— контейнеры, подключенные к сети, которая была создана с помощью драйвера "nat", будут получать IP-адрес из указанного пользователем префикса IP-адреса (``--subnet``). Перенаправление и сопоставление портов из узла контейнера в конечные точки контейнера поддерживается.
> Примечание. В обновлении Windows 10 Creators Update теперь реализована поддержка нескольких сетей NAT! 

- **transparent**— контейнеры, подключенные к сети, которая была создана с помощью драйвера "transparent", будут подключены к физической сети напрямую. IP-адреса из физической сети могут назначаться статически (необходим заданный пользователем параметр ``--subnet``) или динамически с помощью внешнего DHCP-сервера. 

- **overlay** - __Новое!__  Когда подсистема Docker запущена в [режиме мелких объектов](./swarm-mode.md), контейнеры, подключенные к сети наложения, могут связываться с другими контейнерами, подключенными к той же сети между несколькими узлами контейнера. Каждая сеть наложения, созданная в кластере мелких объектов, создается с собственной IP-подсетью, заданной частным IP-префиксом. Драйвер сети наложения использует инкапсуляцию VXLAN.
> Требуется Windows Server 2016 с [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) или обновление Windows 10 Creators Update 

- **l2bridge**— контейнеры, подключенные к сети, которая создана с помощью драйвера "l2bridge", будут находиться в той же IP-подсети, что и узел контейнера. IP-адреса должны назначаться статически из того же префикса, что и узел контейнера. У всех конечных точек контейнера на узле будет одинаковый MAC-адрес из-за преобразования адресов второго уровня (перезапись MAC-адресов) во входящем и исходящем трафике.
> Требуется Windows Server 2016 или обновление Windows 10 Creators Update

- **l2tunnel** - _— этот драйвер должен использоваться только в Microsoft Cloud Stack_.

> Сведения о подключении конечных точек контейнера к дополнительной виртуальной сети со стеком Microsoft SDN см. в статье [Подключение контейнеров к виртуальной сети](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

> В обновлении Windows 10 Creators Update была реализована поддержка платформы, чтобы к работающему контейнеру можно было добавлять новую конечную точку контейнера (так называемое "горячее добавление"). Это приводит к сквозному запуску [отложенного в сети запроса на включение Docker](https://github.com/docker/libnetwork/pull/1661)

## <a name="network-topologies-and-ipam"></a>Сетевые топологии и IPAM
В следующей таблице показано, как сетевое подключение предоставляется для внутренних (контейнер-контейнер) и внешних подключений каждого сетевого драйвера.

<figure>
  <img src="media/network-modes-table.png">
</figure>

### <a name="ipam"></a>IPAM 
IP-адреса распределяются и назначаются каждому сетевому драйверу по-разному. Windows использует сетевую службу узлов (HNS) для передачи IPAM драйверу "nat" и работает в режиме мелких объектов Docker (внутренняя служба KVS) для передачи IPAM драйверу "overlay". Все остальные сетевые драйверы используют внешнюю платформу IPAM.

<figure>
  <img src="media/ipam.png">
</figure>

# <a name="details-on-windows-container-networking"></a>Сведения о сетевых подключениях контейнеров Windows

## <a name="isolation-namespace-with-network-compartments"></a>Изоляция (пространство имен) с секциями сети
Каждая конечная точка контейнера размещается в собственной __секции сети__, являющейся аналогом пространства имен сети в Linux. Виртуальная сетевая карта узла управления и сетевой стек узла размещаются в секции сети по умолчанию. Чтобы обеспечить изоляцию сети между контейнерами на одном узле, для каждого контейнера Windows Server и Hyper-V создается секция сети, куда устанавливается сетевой адаптер этого контейнера. Для подключения к виртуальному коммутатору контейнеры Windows Server используют виртуальный сетевой адаптер узла. Для подключения к виртуальному коммутатору контейнеры Hyper-V используют сетевой адаптер синтетической виртуальной машины (не предоставляется служебной виртуальной машине). 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## <a name="windows-firewall-security"></a>Безопасность брандмауэра Windows

Брандмауэр Windows используется для обеспечения безопасности сети через списки управления доступом портов.

> Примечание. По умолчанию для всех конечных точек контейнера, подключенного к сети наложения, создано правило ALLOW ALL (разрешить все)   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## <a name="container-network-management-with-host-network-service"></a>Управление сетью контейнера с помощью сетевой службы узлов

На изображении ниже представлена совместная работа сетевой службы узлов (HNS) и вычислительной службы узлов (HCS) по созданию контейнеров и подключению конечных точек к сети. 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# <a name="advanced-network-options-in-windows"></a>Дополнительные параметры сети в Windows
Ряд параметров сетевых драйверов позволяет воспользоваться преимуществами особых возможностей Windows. 

## <a name="switch-embedded-teaming-with-docker-networks"></a>Объединение внедренных коммутаторов с сетями Docker

> Применимо ко всем сетевым драйверам 

[Объединение внедренных коммутаторов](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set) можно применять при создании сетей узлов контейнеров для подсистемы Docker, указав несколько сетевых адаптеров (разделенных запятыми) с помощью параметра `-o com.docker.network.windowsshim.interface`. 

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>Установка идентификатора виртуальной сети для сети

> Применимо к сетевым драйверам "transparent" и "l2bridge" 

Чтобы задать идентификатор виртуальной локальной сети для сети, используйте параметр `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` для команды `docker network create`. Например, чтобы создать прозрачную сеть с идентификатором виртуальной сети 11, можно использовать следующую команду.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Когда вы настраиваете идентификатор VLAN для сети, вы настраиваете изоляцию VLAN для любых конечных точек контейнера, которые будут присоединены к этой сети.

> Убедитесь, что сетевой адаптер узла (физический) находится в магистральном режиме, чтобы весь помеченный трафик обрабатывался виртуальным коммутатором с портом vNIC (конечной точки контейнера) в режиме доступа в нужной сети VLAN.

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>Указание имени сети для службы HNS

> Применимо ко всем сетевым драйверам 

Обычно при создании сети контейнера с помощью `docker network create` указанное имя сети используется службой Docker, но не службой HNS. Если вы создаете сеть, можно указать имя, которое определяется службой HNS, с помощью параметра `-o com.docker.network.windowsshim.networkname=<network name>` команды `docker network create`. Например, можно использовать следующую команду для создания прозрачной сети с именем, которое указано для службы HNS.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>Привязка сети к определенному сетевому интерфейсу

> Применимо ко всем сетевым драйверам, за исключением "nat"  

Чтобы привязать сеть (подключенную через виртуальный коммутатор Hyper-V) к определенному сетевому интерфейсу, используйте параметр `-o com.docker.network.windowsshim.interface=<Interface>` для команды `docker network create`. Например можно выполнить следующую команду для создания прозрачной сети, подключенной к сетевому интерфейсу «Ethernet 2»:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Примечание. Значение *com.docker.network.windowsshim.interface*— это *имя* сетевого адаптера, которое можно найти с помощью:

>```
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## VFP

The Virtual Filtering Platform (VFP) extension is a Hyper-V virtual switch, forwarding extension used to enforce network policy and manipulate packets. For instance, VFP is used by the 'overlay' network driver to perform VXLAN encapsulation and by the 'l2bridge' driver to perform MAC re-write on ingresss and egress. The VFP extension is only present on Windows Server 2016 and Windows 10 Creators Update. To check and see if this is running correctly a user run two commands:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## <a name="tips--insights"></a>Советы и полезные рекомендации
Ниже приведен список полезных советов и рекомендаций, составленный на основе стандартных вопросов о сетевых подключениях контейнеров Windows, которые задают участники сообщества.

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>Для работы службы HNS требуется включить IPv6 на хост-машинах контейнеров 
В рамках обновления [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) для работы службы HNS требуется включить IPv6 на узлах контейнеров Windows. При возникновении описанной ниже ошибки есть вероятность, что протокол IPv6 отключен на хост-компьютере.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Мы вносим соответствующие изменения в платформу, чтобы эту проблему можно было автоматически обнаружить/предотвратить. В настоящее время, чтобы убедиться, что на хост-компьютере включен протокол IPv6, можно воспользоваться представленным ниже способом.

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition-instead-of-hns-internal-vswitch"></a>Виртуальные машины Moby Linux используют коммутатор DockerNAT с Docker для Windows (продукт [Docker CE](https://www.docker.com/community-edition)) вместо внутреннего виртуального коммутатора службы HNS. 
Docker для Windows (драйвер Windows для подсистемы Docker CE) в Windows 10 использует внутренний виртуальной коммутатор под названием DockerNAT для подключения виртуальных машин Linux Moby к узлу контейнера. Разработчикам, использующим виртуальные машины Linux Moby в Windows, следует иметь в виду, что их узлы будут использовать виртуальный коммутатор DockerNAT вместо виртуального коммутатора, созданного с помощью службы HNS (который является коммутатором по умолчанию, используемым для контейнеров Windows). 

#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>Чтобы для присвоения IP-адресов на виртуальном узле контейнеров можно было использовать протокол DHCP, необходимо активировать MACAddressSpoofing 
Если узел контейнера виртуализирован и вы хотите использовать DHCP для назначения IP-адресов, необходимо включить MACAddressSpoofing на сетевом адаптере виртуальных машин. В противном случае узел Hyper-V будет блокировать сетевой трафик от контейнеров к виртуальной машине с несколькими MAC-адресами. Активировать MACAddressSpoofing в PowerShell можно с помощью следующей команды.
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
Если вы используете VMware в качестве гипервизора, вам необходимо включить неизбирательный режим. Подробнее см. [здесь](https://kb.vmware.com/s/article/1004099).
#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>Создание нескольких прозрачных сетей на одном узле контейнера
Чтобы создать несколько прозрачных сетей, необходимо указать, к какому сетевому адаптеру (виртуальному) должен быть привязан внешний виртуальный коммутатор Hyper-V. Чтобы задать интерфейс для сети, используйте следующий синтаксис.
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>При выполнении статического назначения IP-адресов не забудьте задать параметры *--subnet* и *--gateway*
При статическом назначении IP-адресов необходимо сначала убедиться, что при создании сети указаны параметры *--subnet* и *--gateway*. IP-адрес подсети и шлюза должен совпадать с IP-адресом, указанным в параметрах сети для контейнера узла, т.е. физической сети. Ниже показано, как можно создать прозрачную сеть, а затем запустить в этой сети конечную точку, используя статическое назначение IP-адресов.

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>Назначение IP-адресов по протоколу DHCP не поддерживается для сетей L2Bridge
Для сетей контейнеров, созданных с помощью драйвера "l2bridge", поддерживается только статичное назначение IP-адресов. Как упоминалось выше, не забудьте задать параметры *--subnet* и *--gateway* для создания сети, которая подходит для статического назначения IP-адресов.

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>У сетей, в которых используется внешний виртуальный коммутатор, должен быть собственный сетевой адаптер
Обратите внимание, что если создание нескольких сетей, использующих внешний виртуальный коммутатор для подключения (например сетей Transparent, L2 Bridge, L2 Transparent) выполняется на одном узле контейнера, для каждой из них потребуется собственный сетевой адаптер. 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>Назначение IP-адресов: остановленные и запущенные контейнеры
Назначение статических IP-адресов выполняется непосредственно на сетевом адаптере контейнера и должно осуществляться, только когда контейнер находится в ОСТАНОВЛЕННОМ состоянии. "Горячее добавление" сетевых адаптеров контейнера или внесение изменений в сетевой стек не поддерживаются (в Windows Server 2016) во время выполнения контейнера.
> Примечание. Это поведение меняется в обновлении Windows10 Creators Update, так как теперь платформа поддерживает "горячее добавление". Эта возможность приводит к сквозному запуску после объединения этого [отложенного в сети запроса на включение Docker](https://github.com/docker/libnetwork/pull/1661)

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>Уже имеющийся виртуальный коммутатор (подсистема Docker его не видит) может блокировать создание прозрачный сети
Если при создании прозрачной сети возникнет ошибка, возможно, в системе есть внешний виртуальный коммутатор, который не был автоматически обнаружен Docker и не позволяет привязать прозрачную сеть к внешнему сетевому адаптеру узла контейнера. 

При создании прозрачной сети Docker создает внешний виртуальный коммутатор для сети, а затем пытается привязать коммутатор к (внешнему) сетевому адаптеру. Адаптер может быть сетевым адаптером виртуальной машины или физическим сетевым адаптером. Если виртуальный коммутатор уже создан на узле контейнера *и видим Docker*, подсистема Windows Docker будет использовать этот коммутатор, а не создавать новый. Но если виртуальный коммутатор создан внешними средствами (например, создан на узле контейнера при помощи диспетчера Hyper-V или PowerShell) и еще не виден Docker, подсистема Windows Docker попытается создать виртуальный коммутатор, после чего она не сможет подключить новый коммутатор к внешнему адаптеру сети узла контейнера (так как сетевой адаптер уже будет подключен к коммутатору, созданному внешними средствами).

Например, эта проблема может возникнуть, если вы сначала создали виртуальный коммутатор на узле, когда служба Docker была запущена, после чего попытались создать прозрачную сеть. В этом случае Docker не распознает созданный коммутатор и создаст новый виртуальный коммутатор для прозрачной сети.

Существует три подхода к решению этой проблемы.

* Вы можете удалить виртуальный коммутатор, созданный внешними средствами, что позволит Docker создать новый виртуальный коммутатор и подключить его к сетевому адаптеру узла без проблем. Перед тем как выбрать этот подход убедитесь, что виртуальный коммутатор, созданный внешними средствами, не используется другими службами (например, Hyper-V).
* Кроме того, если вы захотите использовать внешний виртуальный коммутатор, созданный внешними средствами, перезапустите службы Docker и HNS, чтобы *коммутатор стал видим Docker.*
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Другой вариант— использовать "-o com.docker.network.windowsshim.interface1", чтобы привязать внешний виртуальный коммутатор прозрачной сети к определенному сетевому адаптеру, который еще не используется в узле контейнера (например, сетевой адаптер, отличный от используемого виртуальным коммутатором, созданным внешними средствами). Параметр "-o" описан выше, в разделе [Прозрачная сеть](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network) этого документа.


## <a name="unsupported-features-and-network-options"></a>Неподдерживаемые функции и сетевые параметры 

Следующие сетевые параметры не поддерживаются в Windows и их нельзя передать в ``docker run``:
 * Связывание контейнера (например ``--link``)— _альтернативным решением является использование обнаружения служб_
 * IPv6-адреса (например ``--ip6``)
 * Параметры DNS (например ``--dns-option``)
 * Несколько доменов поиска DNS (например ``--dns-search``)
 
Следующие сетевые параметры и функции не поддерживаются в Windows и их нельзя передать в ``docker network create``:
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

Следующие сетевые параметры не поддерживаются в службах Docker
* Шифрование плоскости данных (например ``--opt encrypted``) 


## <a name="windows-server-2016-work-arounds"></a>Обходные пути известных проблем в Windows Server 2016 

Несмотря на то что мы продолжаем разрабатывать и добавлять новые функции, некоторые из них не будут перенесены на платформы предыдущих версий. Поэтому в этом случае пользователям лучше всего получить последние обновления для Windows 10 и Windows Server.  В разделе ниже перечислены некоторые способы обхода известных проблем и оговорки, применимые к Windows Server 2016 и более ранним версиям Windows 10 (т.е. к версиям до обновления Creators Update 1704)

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>Несколько сетей NAT на узле контейнера WS2016

Разделы для новых сетей NAT должны быть созданы в префиксе более крупной внутренней сети NAT. Чтобы найти префикс, запустите следующую команду из PowerShell со ссылкой на поле InternalIPInterfaceAddressPrefix.

```
PS C:\> Get-NetNAT
```

Например, внутренний префикс сети NAT узла может быть 172.16.0.0/16. В этом случае Docker можно использовать для создания дополнительных сетей NAT, *если они являются подмножеством префикса 172.16.0.0/16.* Например, можно создать две сети NAT с IP-префиксами 172.16.1.0/24 (шлюз, 172.16.1.1) и 172.16.2.0/24 (шлюз, 172.16.2.1).

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

Созданные сети можно перечислить с помощью следующих элементов.
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) можно использовать для определения и настройки сетей контейнера вместе с контейнерами и службами, которые будут использовать эти сети. Ключ "networks" Compose используется как ключ верхнего уровня при определении сетей, к которым будут подключены контейнеры. Например, в синтаксисе ниже определена существующая сеть NAT, создаваемая Docker в качестве сети по умолчанию для всех контейнеров и служб, определенных в указанном файле Compose.

```
networks:
 default:
  external:
   name: "nat"
```

Аналогично можно использовать следующий синтаксис для определения пользовательской сети NAT.

> Примечание. Пользовательская сеть NAT, указанная в примере ниже, определена как раздел внутреннего префикса существующей сети NAT узла контейнера. Дополнительные сведения см. в разделе выше, "Несколько сетей NAT".

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Более подробная информация об определении и настройке сетей контейнера при помощи Docker Compose см. в [справочнике по файлам Compose](https://docs.docker.com/compose/compose-file/).

### <a name="service-discovery"></a>Обнаружение служб
Обнаружение служб поддерживают только определенные сетевые драйверы Windows.

|  | Локальное обнаружение служб  | Глобальное обнаружение служб |
| :---: | :---------------     |  :---                |
| nat | ДА | Н/Д |  
| overlay | ДА | ДА |
| transparent | НЕТ | НЕТ |
| l2bridge | НЕТ | НЕТ |


