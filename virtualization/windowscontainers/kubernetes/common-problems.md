---
title: Устранение неполадок Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Решения распространенных проблем при развертывании Kubernetes и присоединении узлов Windows.
keywords: kubernetes, 1.12, linux, компиляция
ms.openlocfilehash: a5e9369b000aa83aa7ec6ec9bb147f0fd844c820
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178917"
---
# <a name="troubleshooting-kubernetes"></a>Устранение неполадок Kubernetes #
На этой странице описано несколько распространенных проблем, связанных с установкой, сетями и развертываниями Kubernetes.

> [!tip]
> Предложите свой вопрос, создав заявку в [нашем репозитории документации](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Эта страница разделена на следующие категории:
1. [Общие вопросы](#general-questions)
2. [Распространенные сетевые ошибки](#common-networking-errors)
3. [Распространенные ошибки Windows](#common-windows-errors)
4. [Распространенные ошибки главной Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Общие вопросы ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Как узнать, start.ps1 в Windows завершено успешно? ###
Вы должны увидеть kubelet, помощью kube прокси, и (Если вы выбрали Flannel как сетевые решения) процессы узла агента flanneld, работающие на вашем узле с управлением журналы, отображается в отдельные PoSh windows. В дополнение к этому вашего узла Windows должны быть указаны как «Готово» в кластере Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Можно ли настроить для выполнения всех этих в фоновом режиме вместо PoSh windows? ###
Начиная с версии 1.11 Kubernetes, kubelet & помощью kube прокси могут выполняться как собственные [Службы Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). Можно также всегда используют диспетчеры альтернативная служба как [nssm.exe](https://nssm.cc/) всегда выполнять эти процессы, (flanneld, kubelet и помощью kube прокси) в фоновом режиме для вас.


## <a name="common-networking-errors"></a>Распространенные сетевые ошибки ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>Модули POD в Windows не имеют сетевого подключения ###
При использовании любых виртуальных машин, убедитесь, что спуфинг Mac-включена на всех сетевых адаптерах виртуальной Машины. См. в разделе [защиты от подделывания](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection) Дополнительные сведения.


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Модули POD в Windows не может выполнять проверку связи внешние ресурсы ###
POD в Windows нет правила для исходящего подключения запрограммирован протокола ICMP сегодня. Тем не менее поддерживается TCP/UDP. При попытке подключения к ресурсам за пределами кластера продемонстрировать, пожалуйста заменить `ping <IP>` с помощью соответствующего `curl <IP>` команды.

Если вы по-прежнему сталкиваются проблемы, скорее всего настройки сети в [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) заслуживает некоторые особое внимание. Вы всегда можете отредактировать этот статический файл, конфигурация будет применяться к только что созданного ресурсы Kubernetes.

Почему?
Один из требования к сети Kubernetes (см. в разделе [Kubernetes модели](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) — для обмена данными кластера происходит без NAT внутри организации. Чтобы учитывают это требование, у нас есть [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) для всего обмена где мы не хотите исходящего трафика NAT выполнялся. Тем не менее это также означает, что вам нужно исключить внешний IP-адрес, вы пытаетесь запрос из ExceptionList. Только затем трафик, исходящий из POD в Windows будет SNAT'ed правильно для получения ответа извне. В этом учета вашего ExceptionList в `cni.conf` должен выглядеть следующим образом:
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>После запуска start.ps1, Flanneld стоит на «Ожидает сети to be created» ###
Существует множество отчеты об этой проблеме, которые являются высокоприоритетным; скорее всего его проблема синхронизации для при управления IP-адреса flannel сети. Решение — просто повторно запустить start.ps1 или повторно запустить вручную, как показано ниже:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Существует также [PR](https://github.com/coreos/flannel/pull/1042) , которое в настоящее время эта проблема на рассмотрении.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Модули POD в Windows не удается запустить из-за отсутствующих /run/flannel/subnet.env ###
Это означает, что Flannel невозможность запуска правильно. Либо можно попытаться перезапустить flanneld.exe или вы можете скопировать файлы вручную из `/run/flannel/subnet.env` от главного узла Kubernetes для `C:\run\flannel\subnet.env` на рабочий узел Windows и изменять `FLANNEL_SUBNET` строки, чтобы другое число. Например, при необходимости 10.244.4.1/24 подсети узел:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>Мои конечных точек или IP-адреса утечки ###
Существует 2 в настоящее время известных проблем, которые могут привести к конечные точки, чтобы стать причиной утечки данных. 
1.  [Известная проблема](https://github.com/kubernetes/kubernetes/issues/68511) первый — проблема на Kubernetes версия 1.11. Следует Избегайте использования Kubernetes версии 1.11.0 - 1.11.2.
2. Второй [Известная проблема](https://github.com/docker/libnetwork/issues/1950) , может привести к конечные точки, чтобы стать причиной утечки данных является проблемой параллелизма в хранилище конечных точек. Чтобы получить исправление, необходимо использовать Docker EE 18.09 или выше.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Модули POD не удается запустить из-за» сети: не удалось выделить для диапазона «ошибки ###
Это означает, что адресное пространство IP-адрес на вашем узле используется. Чтобы очистить все [утечки конечных точек](#my-endpointsips-are-leaking), перенос все ресурсы на затронутых узлах & выполните следующие команды:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Узел Windows не может получить доступ к службам с помощью IP-адреса служб ###
Это известное ограничение текущего сетевого стека для Windows. Windows *модули* **,** возможность доступа к IP-адрес службы, однако.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>При запуске Kubelet не удается найти ни один сетевой адаптер ###
Сетевому стеку Windows требуется виртуальный адаптер, чтобы сеть Kubernetes работала. Если следующие команды не дают результатов (в оболочке администратора), создание виртуальной сети&mdash; необходимое условие для работы Kubelet &mdash; завершилось ошибкой:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Изучите выходные данные сценария `start-kubelet.ps1`, чтобы узнать, возникли ли ошибки во время создания виртуальной сети.

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Модули pod через некоторое время перестают успешно сопоставлять запросы DNS ###
Существует известных DNS, кэширование проблема сетевого стека Windows Server, версия 1803 и ниже, иногда может привести к DNS-запросы к сбою. Чтобы обойти эту проблему, можно задать значения по максимальный срок ЖИЗНИ кэша равными 0, используя следующие разделы реестра:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Я по-прежнему присутствует проблемы. Что мне делать? ### 
Могут существовать дополнительные ограничения в сети или на узлах, предотвращающие определенные виды взаимодействия между узлами. Убедитесь, что:
  - Вы правильно настроили выбранную [топологию сети](./network-topologies.md)
  - трафик, поступающий от модулей pod, разрешен;
  - HTTP-трафик разрешен, если вы развертываете веб-службы;
  - Пакеты через различные протоколы (ie ICMP и TCP/UDP) не удаляются.


## <a name="common-windows-errors"></a>Распространенные ошибки Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Мои модули Kubernetes остаются в состоянии "ContainerCreating" ###
Эта проблема может быть вызвана множеством причин, однако самая распространенная из них— неправильная настройка образа "Пауза". Это высокоуровневый признак следующей проблемы.


### <a name="when-deploying-docker-containers-keep-restarting"></a>При развертывании контейнеры Docker продолжают перезапускаться ###
Убедитесь, что образ "Пауза" совместим с вашей версией ОС. [Инструкции](./deploying-resources.md) подразумевается, что ОС и контейнеров, которые версии 1803. Если у вас установлена более поздняя версия Windows, например сборка для участников программы предварительной оценки, необходимо настроить образы. Сведения об образах см. в [репозитории Docker](https://hub.docker.com/u/microsoft/) корпорации Майкрософт. Независимо от этого, файл Dockerfile образа "Пауза" и образец службы будут ожидать, что образ помечен как `:latest`.


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