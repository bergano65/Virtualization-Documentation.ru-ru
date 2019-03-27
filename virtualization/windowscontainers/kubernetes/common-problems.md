---
title: Устранение неполадок Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Решения распространенных проблем при развертывании Kubernetes и присоединении узлов Windows.
keywords: kubernetes, 1.12, linux, компиляция
ms.openlocfilehash: 1c5a5ec90b828a4f2430508f02cb9b9afb1c4d53
ms.sourcegitcommit: 1715411ac2768159cd9c9f14484a1cad5e7f2a5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2019
ms.locfileid: "9263511"
---
# <a name="troubleshooting-kubernetes"></a>Устранение неполадок Kubernetes #
На этой странице описано несколько распространенных проблем, связанных с установкой, сетями и развертываниями Kubernetes.

> [!tip]
> Предложите свой вопрос, создав заявку в [нашем репозитории документации](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

На этой странице подразделяется на следующие категории:
1. [Общие вопросы](#general-questions)
2. [Распространенные сетевые ошибки](#common-networking-errors)
3. [Распространенные ошибки Windows](#common-windows-errors)
4. [Распространенные ошибки главной Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Общие вопросы ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Как узнать, start.ps1 в Windows завершено успешно; ###
Вы должны увидеть kubelet, помощью kube прокси, и (Если вы выбрали Flannel как сетевые решения) flanneld узла агента процессы, выполняемые на вашем узле с управлением журналы, отображаемых в отдельные PoSh windows. В дополнение к этому узлу Windows должны быть указаны «Готово» кластера Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Можно ли настроить для выполнения всех этих в фоновом режиме вместо PoSh windows? ###
Начиная с версии 1.11 Kubernetes, kubelet & помощью kube-прокси-сервера может выполняться как собственной [Службы Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). Можно также всегда используют диспетчеры альтернативная служба как [nssm.exe](https://nssm.cc/) всегда выполнять эти процессы (flanneld, kubelet & помощью kube-прокси-сервера) в фоновом режиме для вас. См. в разделе, например шаги [Службы Windows от Kubernetes](./kube-windows-services.md) .

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>У меня есть проблемы запущенных процессов Kubernetes как службы Windows ###
Для исходного по устранению неполадок, можно использовать следующие флаги в [nssm.exe](https://nssm.cc/) для перенаправления стандартных выходных данных и стандартной ошибки в файл:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Дополнительные сведения см. в разделе официальной [nssm использования](https://nssm.cc/usage) документации.

## <a name="common-networking-errors"></a>Распространенные сетевые ошибки ##

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Приходят ошибок, таких как «не удалось hnsCall в Win32: неверный находится на диске.» ###
Эта ошибка может возникнуть при создании пользовательских изменения HNS объектов или установкой нового центра обновления Windows, которые вносите изменения для службы HNS без разрывов вниз старые объекты HNS. Он означает, что объект HNS, который ранее был создан перед обновлением несовместима с установленная версия HNS.

В Windows Server 2019 (и ниже) пользователи могут удалять объекты HNS, удалив файл HNS.data 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Пользователи должны иметь возможность непосредственно удалять несовместимым HNS конечных точек и сети:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Пользователи в Windows Server версии 1903 года могут перейти на следующий раздел реестра и удалить все сетевые адаптеры, начиная с сетевого имени (например `vxlan0` или `cbr0`):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Модули POD в Windows не может выполнять проверку связи внешние ресурсы ###
POD в Windows нет правила для исходящего подключения запрограммирован протокола ICMP, уже сегодня. Тем не менее поддерживается TCP/UDP. При попытке отправить продемонстрировать подключения к ресурсам за пределами кластера, необходимо заменить на `ping <IP>` с соответствующим `curl <IP>` команды.

Если вы по-прежнему сталкивается проблемы, скорее всего настройки сети в [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) заслуживает некоторые особое внимание. Вы всегда можете отредактировать это статический файл, конфигурация будет применяться к только что созданный ресурсы Kubernetes.

Почему?
Требования к сети Kubernetes (см. в разделе [Kubernetes модели](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) является кластерное взаимодействие происходит без NAT внутри организации. Принять это требование, у нас есть [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) для всего обмена которых не нужно исходящего трафика NAT выполнялся. Тем не менее это также означает, что вам нужно исключить внешний IP-адрес, вы пытаетесь запрос из ExceptionList. Только после этого трафик от вашего POD в Windows будет SNAT'ed правильно получать ответ с Интернетом. В этом формате вашего ExceptionList в `cni.conf` должен выглядеть следующим образом:
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Узел Windows не может получить доступ к NodePort службы ###
Локальный доступ NodePort из самого узла завершится ошибкой. Корпорации Майкрософт известно об этом ограничении. Доступ к NodePort будет работать из других узлов или внешних клиентов.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Через некоторое время удаляются виртуальные сетевые карты и HNS конечных точек контейнеров ###
Эта проблема может возникать при `hostname-override` параметр с [помощью kube прокси](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)не передается. Чтобы устранить ее, пользователям необходимо передать имя узла помощью kube прокси следующим образом:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Режим Flannel (инкапсуляция) модули POD возникают проблемы с подключением к после присоединиться узла ###
Всякий раз, когда ранее удаленный узел является повторно подключиться к кластеру, flannelD попытается назначить новый подсети pod на узел. Пользователи должны удалить старые файлы конфигурации модуля подсети в следующих путях:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>После запуска start.ps1, Flanneld стоит на «Ожидает сети to be created» ###
Существует множество отчеты об этой проблеме, которые являются описанных; скорее всего это ошибки синхронизации для момента установки управления IP-адреса flannel сети. Решение — просто повторно запустить start.ps1 или перезапустите его вручную, как показано ниже:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Существует также [PR](https://github.com/coreos/flannel/pull/1042) , которое в настоящее время эта проблема на рассмотрении.


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>На Flannel (хост gw) модули POD в Windows не имеют сетевого подключения ###
Следует требуется использовать l2bridge для сети (то есть [flannel узел шлюз](./network-topologies.md#flannel-in-host-gateway-mode)), необходимо убедиться, спуфинг MAC-адресов включена для узла контейнера Windows виртуальных машин (гостевые ОС). Чтобы добиться этого, необходимо выполните следующую команду от имени администратора на компьютере, размещающие виртуальные машины (пример для Hyper-V):

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> Если вы используете продуктов на основе VMware в соответствии с потребностями виртуализации, можно найти в включить [неизбирательный режим](https://kb.vmware.com/s/article/1004099) для требования подмена MAC.

>[!TIP]
> При развертывании Kubernetes в Azure или IaaS виртуальные машины на другие поставщики облачных самостоятельно, можно также использовать [наложение сеть](./network-topologies.md#flannel-in-vxlan-mode) вместо.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Модули POD в Windows не удается запустить из-за отсутствующих /run/flannel/subnet.env ###
Это означает, что Flannel невозможность запуска правильно. Можно либо попробовать перезапустить flanneld.exe или скопировать файлы по вручную из `/run/flannel/subnet.env` от главного узла Kubernetes для `C:\run\flannel\subnet.env` на рабочий узел Windows и изменять `FLANNEL_SUBNET` строки, чтобы другое значение. Например, при необходимости 10.244.4.1/24 подсети узел:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>Утечки Мои конечных точек или IP-адреса ###
Существует 2 в настоящее время известных проблем, которые могут привести к конечные точки, чтобы стать причиной утечки данных. 
1.  Первый [Известная проблема](https://github.com/kubernetes/kubernetes/issues/68511) вызвана в Kubernetes версия 1.11. Следует Избегайте использования Kubernetes версии 1.11.0 - 1.11.2.
2. Второй [Известная проблема](https://github.com/docker/libnetwork/issues/1950) , может привести к конечных точек утечку проблема параллелизма в хранилище конечных точек. Чтобы получить исправление, необходимо использовать Docker EE 18.09 или выше.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Модули POD не удается запустить из-за «сети: не удалось выделить для диапазона» ошибки ###
Это означает, что адресное пространство IP-адрес на вашем узле используется. Чтобы очистить все [утечки конечных точек](#my-endpointsips-are-leaking), выполните перенос все ресурсы на & затронутые узлов, выполните следующие команды:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Узел Windows не может получить доступ к службам с помощью IP-адреса служб ###
Это известное ограничение текущего сетевого стека для Windows. Windows *модули* **— это** возможность доступа к IP-адрес службы, однако.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>При запуске Kubelet не удается найти ни один сетевой адаптер ###
Сетевому стеку Windows требуется виртуальный адаптер, чтобы сеть Kubernetes работала. Если следующие команды не дают результатов (в оболочке администратора), создание виртуальной сети&mdash; необходимое условие для работы Kubelet &mdash; завершилось ошибкой:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Часто имеет смысл изменить параметр [имя интерфейса](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) start.ps1 сценария, в случаях, где сетевой адаптер узла не «Ethernet». В противном случае Изучите выходные данные из `start-kubelet.ps1` сценарий, чтобы узнать, возникли ли ошибки во время создания виртуальной сети. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Модули pod через некоторое время перестают успешно сопоставлять запросы DNS ###
Существует известных DNS, кэширование проблема сетевого стека для Windows Server, версия 1803, а под ним может образовать DNS-запросы к сбою. Чтобы обойти эту проблему, можно задать значения по максимальный срок ЖИЗНИ кэша нулю, используя следующие разделы реестра:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Все равно приходят проблемы. Что мне делать? ### 
Могут существовать дополнительные ограничения в сети или на узлах, предотвращающие определенные виды взаимодействия между узлами. Убедитесь, что:
  - Вы правильно настроили выбранную [топологию сети](./network-topologies.md)
  - трафик, поступающий от модулей pod, разрешен;
  - HTTP-трафик разрешен, если вы развертываете веб-службы;
  - Пакеты через различные протоколы ie ICMP и TCP/UDP () не удаляются.


## <a name="common-windows-errors"></a>Распространенные ошибки Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Мои модули Kubernetes остаются в состоянии "ContainerCreating" ###
Эта проблема может быть вызвана множеством причин, однако самая распространенная из них— неправильная настройка образа "Пауза". Это высокоуровневый признак следующей проблемы.


### <a name="when-deploying-docker-containers-keep-restarting"></a>При развертывании контейнеры Docker продолжают перезапускаться ###
Убедитесь, что образ "Пауза" совместим с вашей версией ОС. [Инструкции](./deploying-resources.md) подразумевается, что операционная система и контейнеров, которые версии 1803. Если у вас установлена более поздняя версия Windows, например сборка для участников программы предварительной оценки, необходимо настроить образы. Сведения об образах см. в [репозитории Docker](https://hub.docker.com/u/microsoft/) корпорации Майкрософт. Независимо от этого, файл Dockerfile образа "Пауза" и образец службы будут ожидать, что образ помечен как `:latest`.


## <a name="common-kubernetes-master-errors"></a>Распространенные ошибки главной Kubernetes ##
Отладка главного узла Kubernetes делятся на три основных категории (в порядке вероятности):

  - что-то не так с системой контейнеров Kubernetes;
  - что-то не так с выполнением `kubelet`;
  - что-то не так с системой.

Выполните `kubectl get pods -n kube-system` для просмотра модулей pod, созданных Kubernetes. Это может дать определенные сведения о том, в каких из них возникли сбои или какие модули не запускаются. Затем выполните `docker ps -a`, чтобы просмотреть все необработанные контейнеры, поддерживающие эти модули. Наконец, выполните `docker logs [ID]` для контейнеров, которые могут быть причиной проблемы, чтобы просмотреть необработанные выходные данные процессов.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Не удается подключиться к API-серверу по адресу `https://[address]:[port]` ###
Чаще эта ошибка указывает на проблемы с сертификатом. Убедитесь, что файл конфигурации создан корректно, что IP-адреса в нем соответствуют адресу узла и что вы скопировали его в каталог, подключенный API-сервером.

Если [наши инструкции](./creating-a-linux-master.md), как надо помещает найти это:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 в противном случае см. файл манифеста API-сервера для проверки точек подключения.