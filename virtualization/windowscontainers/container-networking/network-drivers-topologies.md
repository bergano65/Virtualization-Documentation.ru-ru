---
title: Сетевые подключения контейнеров Windows
description: Сетевые драйверы и топологии для контейнеров Windows.
keywords: docker, контейнеры
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: cd16f496b85c0977af0d40142768833acadea0f4
ms.sourcegitcommit: 6f505becbafb1e9785c67d6b0715c4c3af074116
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2020
ms.locfileid: "78338048"
---
# <a name="windows-container-network-drivers"></a>Драйверы сети контейнеров Windows  

Помимо использования сети "nat" по умолчанию, созданной подсистемой Docker в Windows, пользователи могут определять собственные сети контейнеров. Определяемые пользователем сети можно создавать с помощью команды DOCKER CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) . В Windows доступны следующие типы сетевых драйверов.

- **nat** — контейнеры, подключенные к сети, которая была создана с помощью драйвера "nat", будут подключены к *внутреннему* коммутатору Hyper-V и получать IP-адрес из указанного пользователем префикса IP-адреса (``--subnet``). Перенаправление и сопоставление портов из узла контейнера в конечные точки контейнера поддерживается.
  
  >[!NOTE]
  > Сети NAT, созданные в Windows Server 2019 (или более поздней версии), больше не сохраняются после перезагрузки.

  > Если на компьютере установлено обновление Windows 10 Creators (или более поздней версии), поддерживаются несколько сетей NAT.
  
- **transparent** — контейнеры, подключенные к сети, которая была создана с помощью драйвера "transparent", будут подключены к физической сети напрямую через *внешний* коммутатор Hyper-V. IP-адреса из физической сети могут назначаться статически (необходим заданный пользователем параметр ``--subnet``) или динамически с помощью внешнего DHCP-сервера.
  
  >[!NOTE]
  >Из-за следующих требований подключение узлов контейнера к прозрачной сети не поддерживается на виртуальных машинах Azure.
  
  > Требуется: Если этот режим используется в сценарии виртуализации (узел контейнера является виртуальной машиной) _, требуется подмена Mac-адреса_.

- **overlay** — когда подсистема Docker запущена в [режиме мелких объектов](../manage-containers/swarm-mode.md), контейнеры, подключенные к сети наложения, могут связываться с другими контейнерами, подключенными к той же сети между несколькими узлами контейнера. Каждая сеть наложения, созданная в кластере мелких объектов, создается с собственной IP-подсетью, заданной частным IP-префиксом. Драйвер сети наложения использует инкапсуляцию VXLAN. **Можно использовать с Kubernetes при использовании подходящих плоскостей управления сетью (например, Фланнел).**
  > Требуется: Убедитесь, что среда [удовлетворяет необходимым требованиям для создания](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) наложения сетей.

  > Требуется: в Windows Server 2019 для этого требуется [KB4489899](https://support.microsoft.com/help/4489899).

  > Требуется: в Windows Server 2016 для этого требуется [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217).

  >[!NOTE]
  >В Windows Server 2019 наложение сетей, созданных DOCKER Swarm, используют правила NAT VFP для исходящего подключения. Это означает, что заданный контейнер получает 1 IP-адрес. Это также означает, что средства на основе протокола ICMP, такие как `ping` или `Test-NetConnection`, должны быть настроены с использованием параметров TCP/UDP в ситуациях отладки.

- **l2bridge** — подобно `transparent` сетевому режиму, контейнеры, подключенные к сети, созданной с помощью драйвера "l2bridge", будут подключены к физической сети с помощью *внешнего* коммутатора Hyper-V. Разница в l2bridge заключается в том, что конечные точки контейнера будут иметь тот же MAC-адрес, что и узел, из-за операции преобразования адресов уровня 2 (MAC-адреса) для входящего и исходящего трафика. В сценариях кластеризации это помогает уменьшить нагрузку на коммутаторы, требующие изучения MAC-адресов иногда кратковременных контейнеров. L2bridge сети можно настроить двумя разными способами:
  1. L2bridge сеть настроена с той же IP-подсетью, что и узел контейнера
  2. L2bridge сеть настроена с новой настраиваемой IP-подсетью
  
  В конфигурации 2 пользователям потребуется добавить конечную точку в сегмент сети узла, который выступает в качестве шлюза, и настроить возможности маршрутизации для назначенного префикса. 
  >[!TIP]
  >Дополнительные сведения о настройке и установке l2bridge можно найти [здесь](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923).

- **l2tunnel** — похоже на l2bridge, однако _этот драйвер следует использовать только в стеке Microsoft Cloud (Azure)_ . Пакеты, поступающие из контейнера, отправляются на узел виртуализации, на котором применяется политика SDN.


## <a name="network-topologies-and-ipam"></a>Сетевые топологии и IPAM

В следующей таблице показано, как сетевое подключение предоставляется для внутренних (контейнер-контейнер) и внешних подключений каждого сетевого драйвера.

### <a name="networking-modesdocker-drivers"></a>Сетевые режимы и драйверы DOCKER

  | Сетевой драйвер Windows для Docker | Типичные варианты использования | Контейнер в контейнер (один узел) | Между контейнерами и внешними узлами (один узел и несколько узлов) | Контейнер в контейнер (с несколькими узлами) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (по умолчанию)** | Удобно для разработчиков | <ul><li>Одна подсеть: подключение с мостом через виртуальный коммутатор Hyper-V</li><li> Перекрестная подсеть: не поддерживается (только один внутренний префикс NAT)</li></ul> | Маршрутизация через виртуальный сетевой адаптер управления (привязка к WinNAT) | Не поддерживается напрямую: требуется предоставление доступа к портам через узел |
  | **Прозрачное** | Удобно для разработчиков и небольших развертываний | <ul><li>Одна подсеть: подключение с мостом через виртуальный коммутатор Hyper-V</li><li>Подключение между подсетями: маршрутизация через узел контейнеров</li></ul> | Маршрутизация через узел контейнеров с прямым доступом к (физическому) сетевому адаптеру | Маршрутизация через узел контейнеров с прямым доступом к (физическому) сетевому адаптеру |
  | **Overlay** | Подходит для нескольких узлов; требуется для DOCKER Swarm, доступно в Kubernetes | <ul><li>Одна подсеть: подключение с мостом через виртуальный коммутатор Hyper-V</li><li>Подключение между подсетями: сетевой трафик инкапсулируется и маршрутизируется через виртуальный сетевой адаптер управления</li></ul> | Не поддерживается напрямую. требуется Вторая конечная точка контейнера, подключенная к сети NAT в правиле NAT Windows Server 2016 или VFP в Windows Server 2019.  | Одна подсеть/подключение между подсетями: сетевой трафик инкапсулируется с помощью VXLAN и маршрутизируется через виртуальный сетевой адаптер управления |
  | **L2Bridge** | Используется для Kubernetes и Microsoft SDN | <ul><li>Одна подсеть: подключение с мостом через виртуальный коммутатор Hyper-V</li><li> Подключение между подсетями: MAC-адрес контейнера повторно записывается при входе и выходе и маршрутизируется</li></ul> | MAC-адрес контейнера повторно записывается при входе и выходе | <ul><li>Одна подсеть: подключение с мостом</li><li>Перекрестная подсеть: направляется через vNIC для WSv1809 и более поздних версий</li></ul> |
  | **L2Tunnel**| Только в Azure | Одна подсеть/подключение между подсетями: разворот пакетов на виртуальный коммутатор Hyper-V физического узла, к которому применяется политика | Трафик должен проходить через виртуальный сетевой шлюз Azure | Одна подсеть/подключение между подсетями: разворот пакетов на виртуальный коммутатор Hyper-V физического узла, к которому применяется политика |

### <a name="ipam"></a>IPAM

IP-адреса распределяются и назначаются каждому сетевому драйверу по-разному. Windows использует сетевую службу узлов (HNS) для передачи IPAM драйверу "nat" и работает в режиме мелких объектов Docker (внутренняя служба KVS) для передачи IPAM драйверу "overlay". Все остальные сетевые драйверы используют внешнюю платформу IPAM.

| Сетевой режим/драйвер | IPAM |
| -------------------------|:----:|
| NAT | Динамическое выделение и назначение IP-адресов службой Networking Service (HNS) от префикса внутренней подсети NAT |
| Прозрачный | Статическое или динамическое (с помощью внешнего DHCP-сервера) выделение и назначение IP-адресов из числа адресов в префиксе сети узла контейнеров |
| Наложение | Динамическое выделение IP-адресов из префиксов под управлением подсистемы Docker в режиме мелких объектов и их назначение через службу HNS |
| L2Bridge | Выделение статического IP-адреса и назначение IP-адресов в пределах префикса сети узла контейнера (также может быть назначено через HNS) |
| L2Tunnel | Только в Azure: динамическое выделение IP-адресов и их назначение через подключаемый модуль |

### <a name="service-discovery"></a>Обнаружение служб

Обнаружение служб поддерживают только определенные сетевые драйверы Windows.

|  | Локальное обнаружение служб  | Глобальное обнаружение служб |
| :---: | :---------------     |  :---                |
| nat | ДА | ДА, с помощью Docker EE |  
| overlay | ДА | Да, с DOCKER EE или KUBE-DNS |
| transparent | НЕТ | НЕТ |
| l2bridge | НЕТ | Да, с KUBE-DNS |
