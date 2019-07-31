---
title: Соединение с узлами Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Развертывание Кубернетес ресаурецес на кластере с смешанными ОС Кубернетес
keywords: кубернетес, 1,14, Windows, Приступая к работе
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 8c21581433f672a22a247db6643a19168eedea6c
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883197"
---
# <a name="deploying-kubernetes-resources"></a>Развертывание ресурсов Кубернетес #
Если у вас есть кластер Кубернетес, состоящий минимум из 1 основного и 1 рабочего, вы можете развернуть Кубернетес ресурсов.
> [!TIP] 
> Хотите узнать, какие ресурсы Кубернетес поддерживаются сегодня в Windows? Более подробную информацию можно найти в разделе [официально поддерживаемые функции](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) и [кубернетес на планах Windows](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Запуск примера службы ##
Вы развернете очень простую [веб-службу на основе PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) для проверки присоединения к кластеру и правильности настройки нашей сети.

Перед тем как это сделать, рекомендуется всегда быть уверенным, что все наши узлы являются работоспособными.
```bash
kubectl get nodes
```

Если все выглядит хорошо, вы можете скачать и запустить следующую службу:
> [!Important] 
> Убедитесь `kubectl apply`в том, что вы хотите дважды проверить или изменить `microsoft/windowsservercore` изображение в файле примера на [изображение контейнера, которое будет доступно вашим узлам](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions).

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

При этом создается развертывание и служба. Команда последняя контрольные значения запрашивает неопределенное состояние для некоторых дел. просто нажмите `Ctrl+C` , чтобы выйти `watch` из команды по завершении наблюдения.

Если все прошло нормально:

  - на вкладке "раздел" в `docker ps` разделе "два контейнера на модуль"
  - вы увидите 2 модуля pod при выполнении команды `kubectl get pods` из мастера Linux
  - `curl` IP-адреса *модуля pod* на порту 80 от главного узла Linux получают ответ от веб-сервера; это подтверждает взаимодействие узлов с модулем pod в сети;
  - проверка связи *между модулями pod* (в том числе между узлами, если у вас несколько узлов Windows) с помощью `docker exec` выполняется успешно; это подтверждает правильную настройку взаимодействия модулей pod;
  - `curl` *IP-адрес виртуальной службы* (рассматривается `kubectl get services`в разделе) из справочника Linux и из отдельных модулей. Это показывает, что служба взаимосвязи с Pod является нужной.
  - `curl` *имя службы* с [DNS-суффиксом по умолчанию](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)кубернетес, которое демонстрирует правильность обнаружения служб.
  - `curl` *нодепорт* из справочника Linux или из компьютеров, находящихся за пределами кластера; Это показывает входящее подключение.
  - `curl` внешние IP – адреса из модуля Pod; в этом примере показана Исходящая связь.

> [!Note]  
> *Узлы контейнера* Windows **не** смогут получать доступ к IP-адресу службы из служб, запланированных на них. Это известное [ограничение на платформы](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) , которое будет улучшено в будущих версиях Windows Server. Тем ** не менее **, Windows, возможно,** сможет получить доступ к IP-адресу службы.

### <a name="port-mapping"></a>Сопоставление порта ### 
Можно также получить доступ к службам, размещенным в модулях pod, через соответствующие им узлы путем сопоставления порта на узле. Для демонстрации этой функции существует [еще один образец YAML, доступный](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) путем сопоставления порта 4444 на узле с портом 80 на модуле pod. Чтобы развернуть его, выполните те же действия.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Теперь можно выполнить команду `curl` на IP-адресе *узла* порта 4444 и получить ответ веб-сервера. Имейте в виду, что это приводит к ограничению масштабирования до одного модуля pod на каждом узле, так как должно применяться сопоставление "один-к-одному".


## <a name="next-steps"></a>Дальнейшие действия ##
В этом разделе мы рассмотрели планирование ресурсов Кубернетес на узлах Windows. На этом руководстве завершается. Если возникли проблемы, ознакомьтесь с разделом устранение неполадок:

> [!div class="nextstepaction"]
> [Поиск и устранение неисправностей](./common-problems.md)

В противном случае вам также может потребоваться запустить компоненты Кубернетес в качестве служб Windows.
> [!div class="nextstepaction"]
> [Службы Windows](./kube-windows-services.md)
