---
title: Сетевые подключения контейнеров Windows
description: Расширенные сетевые возможности для контейнеров Windows.
keywords: docker, контейнеры
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: deea1bfbcd3032f52a6912eb0c36ba467d8b9a9c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910714"
---
# <a name="advanced-network-options-in-windows"></a>Дополнительные параметры сети в Windows

Ряд параметров сетевых драйверов позволяет воспользоваться преимуществами особых возможностей Windows. 

## <a name="switch-embedded-teaming-with-docker-networks"></a>Объединение внедренных коммутаторов с сетями Docker

> Применимо ко всем сетевым драйверам

[Объединение внедренных коммутаторов](https://docs.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/RDMA-and-Switch-Embedded-Teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set) можно применять при создании сетей узлов контейнеров для подсистемы Docker, указав несколько сетевых адаптеров (разделенных запятыми) с помощью параметра `-o com.docker.network.windowsshim.interface`.

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

## <a name="specify-outboundnat-policy-for-a-network"></a>Указание политики Аутбаунднат для сети

> Применяется к l2bridge сетям

Обычно при создании `l2bridge` сети контейнеров с помощью `docker network create`к конечным точкам контейнера не применяется политика Аутбаунднат HNS, что приводит к невозможности доступа контейнеров к внешнему миру. При создании сети можно использовать параметр `-o com.docker.network.windowsshim.enable_outboundnat=<true|false>`, чтобы применить политику Аутбаунднат HNS для предоставления контейнерам доступа к внешнему миру:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true MyL2BridgeNetwork
```

Если существует набор назначений (например, требуется подключение контейнера к контейнеру), в котором мы не хотим Нат'инг, необходимо также указать список исключений:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true -o com.docker.network.windowsshim.outboundnat_exceptions=10.244.10.0/24
```

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

> Примечание. Значение *com.docker.network.windowsshim.interface* — это *имя* сетевого адаптера, которое можно найти с помощью:

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>Указание суффикса DNS и DNS-серверов сети

> Применимо ко всем сетевым драйверам 

Используйте параметр `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>`, чтобы указать DNS-суффикс сети, и параметр `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>`, чтобы указать DNS-серверы сети. Например, чтобы задать DNS-суффикс «example.com» и DNS-серверы сети 4.4.4.4 и 8.8.8.8, можно использовать следующую команду:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

Дополнительные сведения см. в [этой статье](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/).

## <a name="tips--insights"></a>Советы и полезные рекомендации
Ниже приведен список полезных советов и рекомендаций, составленный на основе стандартных вопросов о сетевых подключениях контейнеров Windows, которые задают участники сообщества.

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>Для работы службы HNS требуется включить IPv6 на хост-машинах контейнеров 
В рамках обновления [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217) для работы службы HNS требуется включить IPv6 на узлах контейнеров Windows. При возникновении описанной ниже ошибки есть вероятность, что протокол IPv6 отключен на хост-компьютере.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Мы вносим соответствующие изменения в платформу, чтобы эту проблему можно было автоматически обнаружить/предотвратить. В настоящее время, чтобы убедиться, что на хост-компьютере включен протокол IPv6, можно воспользоваться представленным ниже способом.

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Контейнеры Linux в Windows

**НОВОЕ.** Мы работаем над возможностью параллельного запуска контейнеров Linux и Windows _без помощи виртуальной машины Linux Moby_. Подробные сведения см. в этой [записи блога о контейнерах Linux в Windows (LCOW)](https://blog.docker.com/2017/11/docker-for-windows-17-11/). Вот как приступить к [работе](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux).
> ПРИМЕЧАНИЕ. Решение LCOW приходит на замену виртуальной машине Linux Moby и будет использовать внутренний виртуальный коммутатор "nat" службы HNS по умолчанию.

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition"></a>Виртуальные машины Moby Linux используют коммутатор DockerNAT с Docker для Windows (продукт [Docker CE](https://www.docker.com/community-edition))

Docker для Windows (драйвер Windows для подсистемы Docker CE) в Windows 10 использует внутренний виртуальный коммутатор под названием DockerNAT для подключения виртуальных машин Linux Moby к узлу контейнера. Разработчикам, использующим виртуальные машины Linux Moby в Windows, следует иметь в виду, что их узлы будут использовать виртуальный коммутатор DockerNAT вместо виртуального коммутатора "nat", созданного с помощью службы HNS (который является коммутатором по умолчанию, используемым для контейнеров Windows).



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
При статическом назначении IP-адресов необходимо сначала убедиться, что при создании сети указаны параметры *--subnet* и *--gateway*. IP-адрес подсети и шлюза должен совпадать с IP-адресом, указанным в параметрах сети для контейнера узла, т. е. физической сети. Ниже показано, как можно создать прозрачную сеть, а затем запустить в этой сети конечную точку, используя статическое назначение IP-адресов.

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
* Другой вариант — использовать "-o com.docker.network.windowsshim.interface1", чтобы привязать внешний виртуальный коммутатор прозрачной сети к определенному сетевому адаптеру, который еще не используется в узле контейнера (например, сетевой адаптер, отличный от используемого виртуальным коммутатором, созданным внешними средствами). Параметр "-o" описан далее в разделе [Создание нескольких прозрачных сетей в одном узле контейнера](advanced.md#creating-multiple-transparent-networks-on-a-single-container-host) этого документа.


## <a name="windows-server-2016-work-arounds"></a>Обходные пути известных проблем в Windows Server 2016 

Несмотря на то что мы продолжаем разрабатывать и добавлять новые функции, некоторые из них не будут перенесены на платформы предыдущих версий. Поэтому в этом случае пользователям лучше всего получить последние обновления для Windows 10 и Windows Server.  В разделе ниже перечислены некоторые способы обхода известных проблем и оговорки, применимые к Windows Server 2016 и более ранним версиям Windows 10 (т. е. к версиям до обновления Creators Update 1704)

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
