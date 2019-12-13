---
title: Устранение неполадок Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Решения распространенных проблем при развертывании Kubernetes и присоединении узлов Windows.
keywords: kubernetes, 1,14, Linux, компиляция
ms.openlocfilehash: 471731ec50c7c03816a956bd7aae859ad218be6d
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910454"
---
# <a name="troubleshooting-kubernetes"></a>Устранение неполадок Kubernetes #
На этой странице описано несколько распространенных проблем, связанных с установкой, сетями и развертываниями Kubernetes.

> [!tip]
> Предложите свой вопрос, создав заявку в [нашем репозитории документации](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Эта страница делится на следующие категории:
1. [Общие вопросы](#general-questions)
2. [Распространенные сетевые ошибки](#common-networking-errors)
3. [Общие ошибки Windows](#common-windows-errors)
4. [Распространенные основные ошибки Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Общие вопросы ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Разделы справки знание запуска. ps1 в Windows успешно завершено? ###
Вы должны увидеть kubelet, KUBE-proxy и (если вы выбрали Фланнел в качестве сетевого решения) фланнелд процессы агента узла, запущенные на узле, в котором запущенные журналы отображаются в отдельных окнах PoSh. В дополнение к этому в кластере Kubernetes должен быть указан узел Windows "Готово".

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Можно ли настроить для запуска все это в фоновом режиме вместо PoSh Windows? ###
Начиная с Kubernetes версии 1,11, kubelet & KUBE-proxy можно запускать как собственные [службы Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). Можно также использовать альтернативные Диспетчеры служб, например [НССМ. exe](https://nssm.cc/) , чтобы всегда запускать эти процессы (фланнелд, kubelet & KUBE-proxy) в фоновом режиме. Примеры действий см. [в разделе службы Windows на Kubernetes](./kube-windows-services.md) .

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>У меня возникли проблемы при выполнении процессов Kubernetes в качестве служб Windows ###
Для первоначального устранения неполадок можно использовать следующие флаги в [НССМ. exe](https://nssm.cc/) для перенаправления stdout и stderr в выходной файл:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Дополнительные сведения см. в разделе официальные документы [об использовании НССМ](https://nssm.cc/usage) .

## <a name="common-networking-errors"></a>Распространенные сетевые ошибки ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>Подсистемы балансировки нагрузки несогласованны между узлами кластера ###
В Windows KUBE-proxy создает балансировщик нагрузки HNS для каждой службы Kubernetes в кластере. В конфигурации (по умолчанию) KUBE-proxy узлы в кластерах, содержащих множество (обычно 100) подсистем балансировки нагрузки, могут работать с недоступными временными TCP-портами (то есть динамический диапазон портов, который по умолчанию охватывает порты с 49152 по 65535. Это происходит из-за большого количества портов, зарезервированных на каждом узле для каждой подсистемы балансировки нагрузки (не DSR). Эта проблема может проявляться через ошибки в Kube прокси, например:
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

Пользователи могут опознать эту ошибку, запустив сценарий [коллектлогс. ps1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) и выполняя `*portrange.txt` файлы.

`CollectLogs.ps1` также будет имитировать логику распределения HNS для тестирования доступности выделения ресурсов пула портов в диапазоне временных TCP-портов и отчета об успешном или неуспешном выполнении в `reservedports.txt`. Скрипт резервирует 10 диапазонов TCP-портов 64 (для эмуляции поведения в службе HNS), зарезервированных счетчиков завершается & сбоев, а затем освобождает выделенные диапазоны портов. Число успешных попыток, меньшее 10, означает, что в эфемерном пуле заканчивается свободное место. В `reservedports.txt`также создается эвристическая сводка по количеству резервируемых резервирований портов 64.

Чтобы устранить эту проблему, можно выполнить несколько действий.
1.  Для постоянного решения балансировка нагрузки KUBE-proxy должна быть установлена в [режим DSR](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710). Режим DSR полностью реализован и доступен в новой версии [Windows Server Insider build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (или более поздней версии).
2. В качестве обходного решения пользователи также могут увеличить конфигурацию Windows для временных портов, доступную по умолчанию, с помощью команды, например `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>`. *Предупреждение:* Переопределение диапазона динамических портов по умолчанию может повлиять на другие процессы и службы на узле, которые используют доступные TCP-порты из диапазона, отличного от временного, поэтому этот диапазон следует выбирать осторожно.
3. Существует возможность улучшения масштабируемости для подсистемы балансировки нагрузки, не являющейся режимом DSR, с помощью интеллектуального общего доступа к пулу портов, который планируется выпустить с помощью накопительного обновления в Q1 2020.

### <a name="hostport-publishing-is-not-working"></a>Публикация Хостпорт не работает ###
Сейчас невозможно опубликовать порты с помощью поля Kubernetes `containers.ports.hostPort`, так как это поле не учитывается подключаемыми модулями Windows CNI. Используйте публикацию Нодепорт для времени публикации портов на узле.

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Я вижу ошибки, такие как "сбой Хнскалл в Win32: неправильная дискета находится в накопителе". ###
Эта ошибка может возникать при внесении пользовательских изменений в объекты HNS или при установке новых Центр обновления Windows, которые представляют изменения в службе HNS, без уничтожения старых объектов HNS. Это означает, что объект HNS, который был создан ранее до обновления, несовместим с текущей установленной версией системы HNS.

В Windows Server 2019 (и ниже) пользователи могут удалять объекты HNS, удаляя файл HNS. Data. 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Пользователи должны иметь возможность непосредственно удалить все несовместимые конечные точки или сети в службе HNS:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Пользователи Windows Server версии 1903 могут войти в следующее расположение реестра и удалить все сетевые карты, начинающиеся с сетевого имени (например, `vxlan0` или `cbr0`):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>Контейнеры в моем Фланнел Host-GW в Azure не могут подключиться к Интернету ###
При развертывании Фланнел в режиме host-GW в Azure пакеты должны проходить через физический узел vSwitch Azure. Пользователи должны программировать [определяемые пользователем маршруты](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) типа "виртуальный модуль" для каждой подсети, назначенной узлу. Это можно сделать с помощью портал Azure (см. [пример) или](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)с помощью `az` Azure CLI. Ниже приведен пример UDR с именем "Мирауте" с помощью команды AZ для узла с IP-10.0.0.4 и соответствующей подсетью Pod 10.244.0.0/24:
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> При развертывании Kubernetes на виртуальных машинах Azure или IaaS из других поставщиков облачных служб можно также использовать [наложение сети](./network-topologies.md#flannel-in-vxlan-mode) .

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Мои модули Windows не могут проверить связь с внешними ресурсами ###
В настоящее время для модулей Windows Pod не предусмотрены правила исходящего трафика, запрограммированные для протокола ICMP. Однако поддерживается TCP/UDP. При попытке продемонстрировать подключение к ресурсам за пределами кластера замените `ping <IP>` соответствующими командами `curl <IP>`.

Если у вас по-прежнему возникают проблемы, скорее всего, конфигурация сети в [CNI. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) заслуживает некоторого дополнительного внимания. Вы всегда можете изменить этот статический файл, и конфигурация будет применена ко всем вновь созданным ресурсам Kubernetes.

Почему?
Одно из требований к сети Kubernetes (см. [модель Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) заключается в том, что взаимодействие с кластером происходит без внутренних подключений NAT. Чтобы удовлетворить это требование, у нас есть [список исключений](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) для связи, при котором не требуется исходящий трафик NAT. Однако это также означает, что необходимо исключить внешний IP-адрес, к которому вы пытаетесь выполнить запрос из этого список исключений. Только после этого трафик, исходящий из модулей Windows, будет правильно Снат'ед для получения ответа от внешнего мира. В этом отношении список исключений в `cni.conf` должен выглядеть следующим образом:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Мой узел Windows не может получить доступ к службе Нодепорт ###
Локальный доступ Нодепорт из самого узла завершится ошибкой. Это известное ограничение. Нодепорт доступ будет работать с других узлов или внешних клиентов.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Через некоторое время удаляются конечные точки vNIC и HNS контейнеров. ###
Эта проблема может быть вызвана тем, что параметр `hostname-override` не передается в [KUBE-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Чтобы устранить эту проблему, пользователям необходимо передать имя узла в Kube-proxy следующим образом:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>В режиме Фланнел (вкслан) в моих модулях Pod возникают проблемы с подключением после повторного присоединения к узлу ###
При каждом повторном присоединении ранее удаленного узла к кластеру Фланнелд попытается присвоить узлу новую подсеть Pod. Пользователи должны удалить старые файлы конфигурации подсети Pod по следующим путям:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>После запуска Start. ps1 Фланнелд зависает в ожидании создания сети. ###
Существует множество отчетов об этой проблемы, которые изучаются. скорее всего, это будет вызвано временной проблемой при установке IP-адреса управления для сети фланнел. Чтобы решить проблему, просто перезапустите Start. ps1 или перезапустите его вручную следующим образом:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Кроме [того, существует вопрос, который решает](https://github.com/coreos/flannel/pull/1042) эту ошибку в настоящее время.


### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Не удается запустить мой набор модулей Windows из-за отсутствия/рун/фланнел/субнет.ЕНВ ###
Это означает, что Фланнел не был запущен правильно. Можно либо попытаться перезапустить фланнелд. exe, либо скопировать файлы вручную с `/run/flannel/subnet.env` на главный узел Kubernetes, чтобы `C:\run\flannel\subnet.env` на рабочем узле Windows и изменить строку `FLANNEL_SUBNET` на назначенную подсеть. Например, если была назначена подсеть node 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Безопаснее позволить фланнелд. exe создать этот файл.


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>Подключение Pod к Pod между узлами в кластере Kubernetes, работающем на vSphere, разорвано 
Так как как vSphere, так и Фланнел резервирует порт 4789 (порт ВКСЛАН по умолчанию) для наложения сетей, пакеты могут перехватываться. Если vSphere используется для наложения сети, необходимо настроить использование другого порта, чтобы освободить 4789.  


### <a name="my-endpointsips-are-leaking"></a>Утечка конечных точек и IP-адресов ###
В настоящее время существует две известные проблемы, которые могут привести к утечке конечных точек. 
1.  Первая [известная проблема](https://github.com/kubernetes/kubernetes/issues/68511) — проблема в Kubernetes версии 1,11. Не используйте Kubernetes версии 1.11.0-1.11.2.
2. Вторая [известная проблема](https://github.com/docker/libnetwork/issues/1950) , которая может привести к утечке конечных точек, — это проблема параллелизма в хранилище конечных точек. Чтобы получить исправление, необходимо использовать DOCKER EE 18,09 или более поздней версии.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Не удается запустить мои модули Pod из-за ошибок "Сеть: не удалось выделить диапазон" ###
Это означает, что пространство IP-адресов на узле используется. Чтобы очистить все [утечки конечных точек](#my-endpointsips-are-leaking), необходимо перенести все ресурсы на затронутые узлы & выполнить следующие команды:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Узел Windows не может получить доступ к службам с помощью IP-адреса служб ###
Это известное ограничение текущего сетевого стека для Windows. Однако *Модули Windows Pod могут получить* доступ к IP-адресу службы.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>При запуске Kubelet не удается найти ни один сетевой адаптер ###
Сетевому стеку Windows требуется виртуальный адаптер, чтобы сеть Kubernetes работала. Если следующие команды не дают результатов (в оболочке администратора), создание виртуальной сети &mdash; необходимое условие для работы Kubelet &mdash; завершилось ошибкой:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Часто целесообразно изменить параметр [interfacename](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) сценария Start. ps1 в случаях, когда сетевой адаптер узла не имеет значение "Ethernet". В противном случае просмотрите выходные данные скрипта `start-kubelet.ps1`, чтобы проверить наличие ошибок во время создания виртуальной сети. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Модули pod через некоторое время перестают успешно сопоставлять запросы DNS ###
Существует известная проблема кэширования DNS в сетевом стеке Windows Server версии 1803 и ниже, которая иногда может привести к сбою DNS-запросов. Чтобы обойти эту ошибку, можно установить значение кэша Max TTL равным нулю, используя следующие разделы реестра:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Я по-прежнему вижу проблемы. Что нужно сделать? ### 
Могут существовать дополнительные ограничения в сети или на узлах, предотвращающие определенные виды взаимодействия между узлами. Убедитесь, что:
  - Выбранная [топология сети](./network-topologies.md) правильно настроена
  - трафик, поступающий от модулей pod, разрешен;
  - HTTP-трафик разрешен, если вы развертываете веб-службы;
  - Пакеты из разных протоколов (IE ICMP и TCP/UDP) не отбрасываются

>[!TIP]
> Для получения дополнительных ресурсов самостоятельного разрешения проблемы также можно воспользоваться руководством по устранению неполадок Kubernetes [для Windows.](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)

## <a name="common-windows-errors"></a>Общие ошибки Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Мои модули Kubernetes остаются в состоянии "ContainerCreating" ###
Эта проблема может быть вызвана множеством причин, однако самая распространенная из них — неправильная настройка образа "Пауза". Это высокоуровневый признак следующей проблемы.


### <a name="when-deploying-docker-containers-keep-restarting"></a>При развертывании контейнеры Docker продолжают перезапускаться ###
Убедитесь, что образ "Пауза" совместим с вашей версией ОС. В [инструкциях](./deploying-resources.md) предполагается, что ОС и контейнеры имеют версию 1803. Если у вас установлена более поздняя версия Windows, например сборка для участников программы предварительной оценки, необходимо настроить образы. Сведения об образах см. в [репозитории Docker](https://hub.docker.com/u/microsoft/) корпорации Майкрософт. Независимо от этого, файл Dockerfile образа "Пауза" и образец службы будут ожидать, что образ помечен как `:latest`.


## <a name="common-kubernetes-master-errors"></a>Распространенные основные ошибки Kubernetes ##
Отладка главного узла Kubernetes делятся на три основных категории (в порядке вероятности):

  - что-то не так с системой контейнеров Kubernetes;
  - что-то не так с выполнением `kubelet`;
  - что-то не так с системой.

Выполните `kubectl get pods -n kube-system` для просмотра модулей pod, созданных Kubernetes. Это может дать определенные сведения о том, в каких из них возникли сбои или какие модули не запускаются. Затем выполните `docker ps -a`, чтобы просмотреть все необработанные контейнеры, поддерживающие эти модули. Наконец, выполните `docker logs [ID]` для контейнеров, которые могут быть причиной проблемы, чтобы просмотреть необработанные выходные данные процессов.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Не удается подключиться к API-серверу по адресу `https://[address]:[port]` ###
Чаще эта ошибка указывает на проблемы с сертификатом. Убедитесь, что файл конфигурации создан корректно, что IP-адреса в нем соответствуют адресу узла и что вы скопировали его в каталог, подключенный API-сервером.

Если [следуйте нашим инструкциям](./creating-a-linux-master.md), лучше всего найти следующее:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 в противном случае обратитесь к файлу манифеста сервера API для проверки точек подключения.
