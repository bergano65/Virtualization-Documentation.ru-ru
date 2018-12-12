---
title: Присоединение узла Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Развертывание resoureces Kubernetes в кластере Kubernetes смешанных ОС.
keywords: kubernetes, 1.12, windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 608cda1494d03da59e8a875910c8eedd04ba11dc
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/08/2018
ms.locfileid: "6179033"
---
# <a name="deploying-kubernetes-resources"></a>Развертывание Kubernetes ресурсы #
При условии, что у вас есть кластера Kubernetes, состоящий из главного узла по крайней мере 1 и 1 рабочий, можно приступать к развертыванию Kubernetes ресурсов.
> [!TIP] 
> Значение хотите знать, какие ресурсы Kubernetes сегодня поддерживается в Windows? Дополнительные сведения см. в разделе [официально поддерживается функции](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) и [Kubernetes в схеме Windows](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap) .


## <a name="running-a-sample-service"></a>Запуск образца службы ##
Вы развернете очень простую [веб-службу на основе PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) для проверки присоединения к кластеру и правильности настройки нашей сети.

Прежде чем сделать это, это всегда бывает нужно убедиться, что все узлы работоспособны.
```bash
kubectl get nodes
```

Если все выглядит хорошо, вы можете скачать и запустить следующие службы:
> [!Important] 
> Прежде чем `kubectl apply`, убедитесь, что на двойным-check или изменить `microsoft/windowsservercore` изображения в образце файла образа [контейнера, готов к запуску с узлов](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

В результате создается развертывание и служба. Последняя команда отслеживание запросов модули POD будут неопределенное время отслеживать их состояние; просто нажмите `Ctrl+C` для выхода из `watch` команда когда завершите наблюдение.

Если все прошло нормально:

  - см. в разделе 2 контейнеров каждого модуля в разделе `docker ps` команду на узле Windows
  - вы увидите 2 модуля pod при выполнении команды `kubectl get pods` из мастера Linux
  - `curl` IP-адреса *модуля pod* на порту 80 от главного узла Linux получают ответ от веб-сервера; это подтверждает взаимодействие узлов с модулем pod в сети;
  - проверка связи *между модулями pod* (в том числе между узлами, если у вас несколько узлов Windows) с помощью `docker exec` выполняется успешно; это подтверждает правильную настройку взаимодействия модулей pod;
  - `curl` виртуальный *IP-адрес службы* (показано в разделе `kubectl get services`) из главного узла Linux и отдельных модулей POD; Этот пример демонстрирует необходимые службы для взаимодействия pod.
  - `curl` *имя службы* с Kubernetes [по умолчанию DNS-суффикс](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), демонстрирующий обнаружение правильного служб.
  - `curl` *NodePort* из главного узла Linux или машин за пределами кластера. Этот пример демонстрирует входящие подключения.
  - `curl` внешний IP-адреса из внутри модуля; Этот пример демонстрирует исходящего подключения.

> [!Note]  
> Будет *узлах контейнеров* в Windows **не** сможете получить доступ к IP-адрес службы из служб, запланированные на них. Это [известное ограничение платформы](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) , будет устранено в будущих версий Windows Server. Windows *модули* **,** возможность доступа к IP-адрес службы, однако.

### <a name="port-mapping"></a>Сопоставление портов ### 
Можно также получить доступ к службам, размещенным в модулях pod, через соответствующие им узлы путем сопоставления порта на узле. Для демонстрации этой функции существует [еще один образец YAML, доступный](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) путем сопоставления порта 4444 на узле с портом 80 на модуле pod. Чтобы развернуть его, выполните те же действия.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Теперь можно выполнить команду `curl` на IP-адресе *узла* порта 4444 и получить ответ веб-сервера. Имейте в виду, что это приводит к ограничению масштабирования до одного модуля pod на каждом узле, так как должно применяться сопоставление "один-к-одному".


## <a name="next-steps"></a>Дальнейшие действия ##
В этом разделе мы рассматривается как запланировать Kubernetes ресурсы на узлах Windows. К этому относится руководства. Если возникли проблемы, можно узнать в разделе по устранению неполадок:

> [!div class="nextstepaction"]
> [Поиск и устранение неисправностей](./common-problems.md)