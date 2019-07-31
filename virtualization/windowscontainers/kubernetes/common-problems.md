---
title: Устранение неполадок Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Решения распространенных проблем при развертывании Kubernetes и присоединении узлов Windows.
keywords: кубернетес, 1,14, Linux, компиляция
ms.openlocfilehash: bdf1fd78bbbebcad3562872d9e71c961be6c64eb
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883007"
---
# <a name="troubleshooting-kubernetes"></a>Устранение неполадок Kubernetes #
На этой странице описано несколько распространенных проблем, связанных с установкой, сетями и развертываниями Kubernetes.

> [!tip]
> Предложите свой вопрос, создав заявку в [нашем репозитории документации](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

На этой странице выделяются следующие категории:
1. [Общие вопросы](#general-questions)
2. [Общие сетевые ошибки](#common-networking-errors)
3. [Общие ошибки Windows](#common-windows-errors)
4. [Распространенные основные ошибки Кубернетес](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Общие вопросы ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Как узнать, что Start. ps1 в Windows успешно завершено? ###
Вы должны увидеть кубелет, Кубе-proxy и (если вы выбрали Фланнел как сетевое решение), фланнелд на своем узле запущенные процессы агента узла, в котором журналы отображаются в разных окнах ПОШ. Помимо этого, ваш узел Windows должен быть указан в списке "Готово" в кластере Кубернетес.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Можно ли настроить для запуска всего этого элемента в фоновом режиме, а не ПОШ Windows? ###
Начиная с Кубернетес версии 1,11, кубелет & Кубе-proxy можно запускать как собственные [службы Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). Вы также можете использовать альтернативные Диспетчеры служб, такие как [НССМ. exe](https://nssm.cc/) , чтобы всегда запускать эти процессы (фланнелд, кубелет & Кубе-proxy) в фоновом режиме. Ниже приведены примеры действий [на Кубернетес службах Windows](./kube-windows-services.md) .

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>У меня проблемы с Кубернетес процессами в качестве служб Windows ###
Для первоначального устранения неполадок вы можете переадресовать stdout и stderr в выходной файл с помощью следующих флагов в [НССМ. exe](https://nssm.cc/) :
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Дополнительные сведения можно найти в разделе официальные документы [об использовании НССМ](https://nssm.cc/usage) .

## <a name="common-networking-errors"></a>Общие сетевые ошибки ##

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Я вижу ошибки, например "Хнскалл", не удалось найти в Win32 неправильной диск. ###
Эта ошибка может возникать при внесении настраиваемых изменений в объекты HNS или установке нового обновления Windows, которое вносит изменения в службе HNS без разрыва старых объектов HNS. Оно указывает на то, что объект HNS, который был ранее создан перед обновлением, несовместим с текущей установленной версией службе HNS.

В Windows Server 2019 (и более ранних версий) пользователи могут удалять объекты службе HNS, удаляя файл службе HNS. Data. 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Пользователи должны прямо удалять любые несовместимые конечные точки и сети службы HNS.
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Пользователи Windows Server версии 1903 могут перейти к следующему разделу реестра и удалить все сетевые платы, начиная с сетевого имени (например, `vxlan0` или `cbr0`):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Не удается проверить связь с внешними ресурсами в Windows. ###
В Windows не установлены правила исходящих сообщений, запрограммированные для протокола ICMP уже сегодня. Тем не менее, TCP/UDP поддерживается. При попытке продемонстрировать подключение к ресурсам за пределами кластера, подставьте `ping <IP>` соответствующие `curl <IP>` команды.

Если вы по-прежнему столкнулись с проблемами, скорее всего, ваша сетевая конфигурация в [CNI. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) заслуживает некоторых дополнительных внимания. Вы всегда можете изменить этот статический файл, конфигурация будет применена ко всем вновь созданным ресурсам Кубернетес.

Почему?
Один из требований к сети Кубернетес (см. [модель кубернетес](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) предназначен для того, чтобы взаимодействие с кластером происходило без внутренних NAT. Для соблюдения этого требования у нас есть [список исключений](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) для всех сообщений, в которых не требуется исходящий NAT. Однако это также означает, что вы должны исключить внешний IP-адрес, который вы пытаетесь запросить из список исключений. Только после этого трафик, исходящий из ваших модулей Windows, будет Снат'ед правильно, чтобы получить ответ из внешнего мира. В связи с этим список исключений в `cni.conf` этом случае должен выглядеть следующим образом:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Узлу Windows не удается получить доступ к службе Нодепорт ###
Локальный доступ Нодепорт к самому узлу может завершиться сбоем. Корпорации Майкрософт известно об этом ограничении. Нодепорт Access будет работать с другими узлами или внешними клиентами.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>В течение некоторого времени конечные точки Вникс и службе HNS для контейнеров удаляются ###
Эта проблема может возникать, если `hostname-override` параметр не передается в [Кубе-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Чтобы устранить эту проблему, пользователям необходимо передать имя узла в Кубе-proxy, как описано ниже.
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>В режиме Фланнел (вкслан) после повторного присоединения к узлу не удается установить связь между ними. ###
Всякий раз, когда удаленный узел присоединяется к кластеру, Фланнелд попытается назначить узлу новую подсеть Pod. Пользователи должны удалить старые файлы конфигурации подсети Pod по следующим путям:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>После запуска Start. ps1 Фланнелд задерживается в разделе ожидание создания сети. ###
Существует множество отчетов об этой ошибке, которые изучаются. скорее всего, это проблема синхронизации при установке IP-адреса управления фланнел сети. Чтобы устранить проблему, просто запустите файл Start. ps1 или перезапустите его вручную, как описано ниже.
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Кроме того, в настоящее время есть [Цена](https://github.com/coreos/flannel/pull/1042) , которая решает эту ошибку в разделе "Рецензирование".


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>На Фланнел (Host-GW) отсутствует сетевое подключение для среды Windows. ###
Если вы хотите использовать l2bridge для сетей ( [фланнел Host-Gateway](./network-topologies.md#flannel-in-host-gateway-mode)), необходимо убедиться в том, что для виртуальных машин (гостей) контейнера Windows включена ПОДМЕНа MAC-адресов. Для этого необходимо выполнить указанные ниже действия для администратора на компьютере, на котором размещены виртуальные компьютеры (например, для Hyper-V).

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> Если вы используете продукт на базе VMware для удовлетворения потребностей вашей виртуализации, пожалуйста, проследите, чтобы включить [сквозной режим](https://kb.vmware.com/s/article/1004099) для требования подмены Mac.

>[!TIP]
> Если вы разворачиваете Кубернетес на виртуальных машинах Azure или IaaS из других поставщиков облачных служб самостоятельно, вы также можете использовать [наложение сетей](./network-topologies.md#flannel-in-vxlan-mode) .

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Не удается запустить мой набор Windows/рун/фланнел/субнет.ЕНВ из-за отсутствия ###
Это говорит о том, что Фланнел не был запущен должным образом. Вы можете либо попытаться перезапустить фланнелд. exe, либо скопировать файлы вручную с сайта `/run/flannel/subnet.env` кубернетес `C:\run\flannel\subnet.env` на узел рабочего процесса Windows и изменить `FLANNEL_SUBNET` строку на назначенную подсеть. Например, если вы назначили 10.244.4.1/24 подсети узла:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Лучше позволить фланнелд. exe создавать этот файл.

### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>Связь между узлами из модуля "модуль-приложение" в Кубернетес кластере, работающем на всфере, не работает 
Так как оба всфере и Фланнел резервирует порт 4789 (порт по умолчанию для ВКСЛАН) для сетей с наложением, пакеты могут быть перехвачены. Если всфере используется для наложения сети, он должен быть настроен на использование другого порта, чтобы освободить 4789.  


### <a name="my-endpointsips-are-leaking"></a>Утечки конечных точек и IP-адресов ###
Существуют две известные проблемы, которые могут привести к утечке конечных точек. 
1.  Первая [известная проблема](https://github.com/kubernetes/kubernetes/issues/68511) является проблемой в кубернетес версии 1,11. Не используйте Кубернетес версии 1.11.0-1.11.2.
2. Вторая [известная проблема](https://github.com/docker/libnetwork/issues/1950) , из-за которой могут быть обнаружены конечные точки, — это проблема параллелизма в хранилище конечных точек. Для получения исправления необходимо использовать Dock EE 18,09 или более поздней версии.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Не удается запустить мои обыкновенные ресурсы из-за ошибок "Сеть: сбой выделения для диапазона" ###
Это означает, что используется пространство IP-адресов на вашем узле. Чтобы очистить все [потерянные конечные точки](#my-endpointsips-are-leaking), перенесите ресурсы на затронутые узлы & выполните указанные ниже команды.
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Узел Windows не может получить доступ к службам с помощью IP-адреса служб ###
Это известное ограничение текущего сетевого стека для Windows. Тем ** не менее **, Windows, возможно,** сможет получить доступ к IP-адресу службы.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>При запуске Kubelet не удается найти ни один сетевой адаптер ###
Сетевому стеку Windows требуется виртуальный адаптер, чтобы сеть Kubernetes работала. Если следующие команды не дают результатов (в оболочке администратора), создание виртуальной сети&mdash; необходимое условие для работы Kubelet &mdash; завершилось ошибкой:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Часто важно изменить параметр [интерфаценаме](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) для сценария Start. ps1 в тех случаях, когда сетевой адаптер хоста не является "Ethernet". В противном случае ознакомьтесь с результатами `start-kubelet.ps1` сценария, чтобы убедиться, что при создании виртуальной сети возникли ошибки. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Модули pod через некоторое время перестают успешно сопоставлять запросы DNS ###
Существует известная ошибка кэширования DNS в сетевом стеке Windows Server, версия 1803 и более ранние версии, которые иногда могут привести к сбою DNS-запросов. Для решения этой проблемы можно установить для параметра "максимальное количество значений кэша TTL" значение ноль с помощью следующих разделов реестра:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Я по-прежнему вижу проблемы. Что мне делать? ### 
Могут существовать дополнительные ограничения в сети или на узлах, предотвращающие определенные виды взаимодействия между узлами. Убедитесь, что:
  - Вы правильно настроили выбранную [топологию сети](./network-topologies.md)
  - трафик, поступающий от модулей pod, разрешен;
  - HTTP-трафик разрешен, если вы развертываете веб-службы;
  - Пакеты с разными протоколами (IE ICMP и TCP/UDP) не отбрасываются


## <a name="common-windows-errors"></a>Общие ошибки Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Мои модули Kubernetes остаются в состоянии "ContainerCreating" ###
Эта проблема может быть вызвана множеством причин, однако самая распространенная из них— неправильная настройка образа "Пауза". Это высокоуровневый признак следующей проблемы.


### <a name="when-deploying-docker-containers-keep-restarting"></a>При развертывании контейнеры Docker продолжают перезапускаться ###
Убедитесь, что образ "Пауза" совместим с вашей версией ОС. В [инструкциях](./deploying-resources.md) предполагается, что ОС и контейнеры имеют версию 1803. Если у вас установлена более поздняя версия Windows, например сборка для участников программы предварительной оценки, необходимо настроить образы. Сведения об образах см. в [репозитории Docker](https://hub.docker.com/u/microsoft/) корпорации Майкрософт. Независимо от этого, файл Dockerfile образа "Пауза" и образец службы будут ожидать, что образ помечен как `:latest`.


## <a name="common-kubernetes-master-errors"></a>Распространенные основные ошибки Кубернетес ##
Отладка главного узла Kubernetes делятся на три основных категории (в порядке вероятности):

  - что-то не так с системой контейнеров Kubernetes;
  - что-то не так с выполнением `kubelet`;
  - что-то не так с системой.

Выполните `kubectl get pods -n kube-system` для просмотра модулей pod, созданных Kubernetes. Это может дать определенные сведения о том, в каких из них возникли сбои или какие модули не запускаются. Затем выполните `docker ps -a`, чтобы просмотреть все необработанные контейнеры, поддерживающие эти модули. Наконец, выполните `docker logs [ID]` для контейнеров, которые могут быть причиной проблемы, чтобы просмотреть необработанные выходные данные процессов.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Не удается подключиться к API-серверу по адресу `https://[address]:[port]` ###
Чаще эта ошибка указывает на проблемы с сертификатом. Убедитесь, что файл конфигурации создан корректно, что IP-адреса в нем соответствуют адресу узла и что вы скопировали его в каталог, подключенный API-сервером.

Если вы выйдете [по нашим инструкциям](./creating-a-linux-master.md), лучше всего найти это в следующих случаях:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 в противном случае ознакомьтесь с файлом манифеста сервера API, чтобы проверить точки монтирования.