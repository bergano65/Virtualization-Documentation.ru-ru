---
title: Присоединение узлов Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Развертывание Kubernetes ресаурецес в кластере с смешанными операционными системами Kubernetes.
keywords: kubernetes, 1,14, Windows, начало работы
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909964"
---
# <a name="deploying-kubernetes-resources"></a>Развертывание ресурсов Kubernetes #
Если у вас есть кластер Kubernetes, состоящий как минимум из 1 главного и 1 рабочей роли, можно приступать к развертыванию ресурсов Kubernetes.
> [!TIP] 
> Хотите узнать, какие ресурсы Kubernetes сейчас поддерживаются в Windows? Дополнительные сведения см. в [официально поддерживаемых функциях](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) и [Kubernetes в стратегии Windows](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Запуск примера службы ##
Вы развернете очень простую [веб-службу на основе PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) для проверки присоединения к кластеру и правильности настройки нашей сети.

Перед этим рекомендуется убедиться, что все наши узлы работоспособны.
```bash
kubectl get nodes
```

Если все выглядит хорошо, можно скачать и запустить следующую службу:
> [!Important] 
> Прежде чем `kubectl apply`, убедитесь, что в примере файла установлен флажок, чтобы в [образе контейнера, который будет готов к запуску на ваших узлах, была установлена](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)проверка и изменение образа `microsoft/windowsservercore`.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

При этом создается развертывание и служба. Последняя команда Watch запрашивает у модулей Pod неопределенное состояние для отслеживания их состояния. просто нажмите `Ctrl+C`, чтобы выйти из команды `watch` по завершении наблюдения.

Если все прошло нормально:

  - см. два контейнера на модуль Pod в разделе `docker ps` команда в узле Windows.
  - вы увидите 2 модуля pod при выполнении команды `kubectl get pods` из мастера Linux
  - `curl` на IP-адресах *Pod* на порте 80 с главного компьютера Linux получает ответ веб – сервера; Это демонстрирует правильное взаимодействие между узлами по сети.
  - проверка связи *между модулями pod* (в том числе между узлами, если у вас несколько узлов Windows) с помощью `docker exec` выполняется успешно; это подтверждает правильную настройку взаимодействия модулей pod;
  - `curl` *IP-адрес виртуальной службы* (отображается в разделе `kubectl get services`) из главного и отдельных модулей Linux. Это демонстрирует правильную связь между службами и Pod.
  - `curl` *имя службы* с [суффиксом DNS по умолчанию](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)Kubernetes, демонстрирующим правильное обнаружение службы.
  - `curl` *нодепорт* из главного или виртуальных машин Linux за пределами кластера; Это демонстрирует входящее подключение.
  - `curl` внешние IP-адреса из модуля Pod; Это демонстрирует исходящее подключение.

> [!Note]  
> *Узлы контейнера* Windows **не** смогут получить доступ к IP-адресу службы из служб, запланированных на них. Это [известное ограничение платформы](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) , которое будет улучшено в будущих версиях Windows Server. Однако *Модули Windows Pod могут получить* доступ к IP-адресу службы.

## <a name="next-steps"></a>Дальнейшие действия ##
В этом разделе мы рассмотрели планирование ресурсов Kubernetes на узлах Windows. Это завершается руководством. Если возникли проблемы, ознакомьтесь с разделом устранение неполадок:

> [!div class="nextstepaction"]
> [Устранение неполадок](./common-problems.md)

В противном случае может потребоваться выполнение Kubernetes компонентов в качестве служб Windows.
> [!div class="nextstepaction"]
> [Службы Windows](./kube-windows-services.md)
