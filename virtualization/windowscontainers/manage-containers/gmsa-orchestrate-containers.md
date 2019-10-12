---
title: Согласование контейнеров с Гмса
description: Управление контейнерами Windows с помощью групповой управляемой учетной записи службы (Гмса).
keywords: Dock, Containers, Active Directory, гмса, оркестрации, кубернетес, групповая управляемая учетная запись службы, групповая управляемая учетные записи служб
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209844"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>Согласование контейнеров с Гмса

В рабочих средах часто используется контейнерная Orchestrator для развертывания приложений и служб и управления ими. Каждая из этих Orchestrator имеет собственные парадигмы управления и отвечает за принятие спецификаций учетных данных для предоставления платформе контейнера Windows.

При управлении контейнерами с помощью групповых управляемых учетных записей служб (Гмсас) убедитесь в том, что:

> [!div class="checklist"]
> * Все узлы контейнеров, которые можно запланировать для выполнения контейнеров с Гмсас, являются присоединенными к домену
> * Узлы контейнера имеют доступ, чтобы получить пароли для всех Гмсас, используемых контейнерами.
> * Файлы спецификаций учетных данных создаются и отправляются в Orchestrator или копируются на каждый узел контейнера в зависимости от того, как Orchestrator предпочитает их обрабатывать.
> * Сети контейнеров позволяют контейнерам взаимодействовать с контроллерами домена Active Directory для получения билетов Гмса

## <a name="how-to-use-gmsa-with-service-fabric"></a>Использование Гмса с фабрикой служб

Фабрика служб поддерживает выполнение контейнеров Windows с помощью Гмса, если указать расположение спецификаций учетных данных в манифесте приложения. Вам потребуется создать файл спецификации учетных данных и поместить его в подкаталог **кредентиалспекс** каталога данных Dock на каждом узле, чтобы структура обслуживания могла ее найти. Вы можете выполнить командлет **Get-кредентиалспек** , который входит в состав [модуля кредентиалспек PowerShell](https://aka.ms/credspec), чтобы проверить, правильно ли указано расположение вашей учетной записи.

Дополнительные сведения о том, как настроить приложение, [можно найти в разделе Краткое руководство: развертывание контейнеров Windows в структуре обслуживания](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) и [Настройка Гмса для контейнеров Windows, работающих в структуре служб](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) .

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Использование Гмса с Dock Сварм

Чтобы использовать Гмса с контейнерами, управляемыми стыковочным узлом Сварм, запустите команду [Create Service Dock](https://docs.docker.com/engine/reference/commandline/service_create/) с `--credential-spec` параметром:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Дополнительные сведения об использовании спецификаций учетных данных с помощью служб Dock можно найти в [Сварм примере Dock](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

## <a name="how-to-use-gmsa-with-kubernetes"></a>Использование Гмса с Кубернетес

Поддержка планирования контейнеров Windows с помощью Гмсас в Кубернетес доступна в виде альфа-функции в Кубернетес 1,14. Более подробную информацию об этой функции можно найти в разделе [Настройка гмса для Windows и контейнерах](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) , а также о том, как ее можно протестировать в дистрибутиве кубернетес.

## <a name="next-steps"></a>Дальнейшие действия

Помимо orchestration Containers, вы также можете использовать Гмсас для:

- [Настройка приложений](gmsa-configure-app.md)
- [Работа с контейнерами](gmsa-run-container.md)

Если во время установки возникнут проблемы, ознакомьтесь с нашим [руководством по устранению неполадок для поиска](gmsa-troubleshooting.md) возможных решений.
