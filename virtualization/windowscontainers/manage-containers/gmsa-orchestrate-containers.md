---
title: Управление контейнерами с помощью gMSA
description: Как управлять контейнерами Windows с помощью групповой управляемой учетной записи службы (gMSA).
keywords: DOCKER, контейнеры, Active Directory, gmsa, оркестрации, kubernetes, групповая управляемая учетная запись службы, групповые управляемые учетные записи служб
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910264"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>Управление контейнерами с помощью gMSA

В рабочих средах вы часто используете контейнер Orchestrator для развертывания приложений и служб и управления ими. Каждая Orchestrator имеет собственные парадигмы управления и отвечает за принятие спецификаций учетных данных для предоставления платформе контейнера Windows.

При управлении контейнерами с помощью групповых управляемых учетных записей служб (Gmsa) убедитесь в том, что:

> [!div class="checklist"]
> * Все узлы контейнеров, для которых можно запланировать выполнение контейнеров с Gmsa, присоединены к домену
> * Узлы контейнера имеют доступ для получения паролей всех Gmsa, используемых контейнерами.
> * Файлы спецификации учетных данных создаются и передаются в Orchestrator или копируются на каждый узел контейнера в зависимости от того, как Orchestrator предпочитает их обработку.
> * Сети контейнеров позволяют контейнерам взаимодействовать с контроллерами домен Active Directory для получения билетов gMSA.

## <a name="how-to-use-gmsa-with-service-fabric"></a>Использование gMSA с Service Fabric

Service Fabric поддерживает выполнение контейнеров Windows с помощью gMSA при указании расположения спецификации учетных данных в манифесте приложения. Необходимо создать файл спецификации учетных данных и поместить его в подкаталог **кредентиалспекс** каталога данных DOCKER на каждом узле, чтобы Service Fabric мог его разместить. Вы можете выполнить командлет **Get-кредентиалспек** , который входит в [модуль PowerShell кредентиалспек](https://aka.ms/credspec), чтобы проверить, находится ли ваша спецификация учетных данных в правильном расположении.

Дополнительные сведения о настройке приложения см. [в разделе Краткое руководство. Развертывание контейнеров Windows для Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) и [Настройка контейнеров gMSA для Windows, выполняемых на Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) .

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Как использовать gMSA с DOCKER Swarm

Чтобы использовать gMSA с контейнерами, управляемыми DOCKER Swarm, выполните команду [DOCKER Service Create](https://docs.docker.com/engine/reference/commandline/service_create/) с параметром `--credential-spec`:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Дополнительные сведения об использовании спецификаций учетных данных с помощью служб DOCKER см. в [примере DOCKER Swarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

## <a name="how-to-use-gmsa-with-kubernetes"></a>Как использовать gMSA с Kubernetes

Поддержка планирования контейнеров Windows с помощью Gmsa в Kubernetes доступна в виде альфа-функции в Kubernetes 1,14. Последние сведения об этой функции и их тестировании в дистрибутиве Kubernetes см. в разделе [Настройка gMSA для модулей Windows и контейнеров](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) .

## <a name="next-steps"></a>Дальнейшие действия

Помимо управляемых контейнеров, можно также использовать Gmsa для:

- [Настройка приложений](gmsa-configure-app.md)
- [Выполнение контейнеров](gmsa-run-container.md)

Если во время установки возникнут проблемы, ознакомьтесь с нашим [руководством по устранению неполадок](gmsa-troubleshooting.md) , чтобы получить возможные решения.
