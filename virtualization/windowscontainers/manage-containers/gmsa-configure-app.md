---
title: Настройка приложения для использования групповой управляемой учетной записи службы
description: Настройка приложений для использования групповых управляемых учетных записей служб (Gmsa) для контейнеров Windows.
keywords: DOCKER, контейнеры, Active Directory, gmsa, приложения, приложения, групповая управляемая учетная запись службы, групповые управляемые учетные записи служб, конфигурация
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909794"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Настройка приложения для использования gMSA

В типичной конфигурации контейнер получает только одну групповую управляемую учетную запись службы (gMSA), которая используется всякий раз, когда учетная запись компьютера контейнера пытается пройти проверку подлинности в сетевых ресурсах. Это означает, что приложение должно быть запущено как **Локальная система** или **Сетевая служба** , если необходимо использовать удостоверение gMSA.

## <a name="run-an-iis-app-pool-as-network-service"></a>Запуск пула приложений IIS в качестве сетевой службы

Если в контейнере размещен веб-сайт IIS, то все, что нужно сделать для использования gMSA, — это задать удостоверение пула приложений для **Network Service**. Это можно сделать в Dockerfile, добавив следующую команду:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Если ранее вы использовали статические учетные данные пользователя для пула приложений IIS, рассмотрите возможность использования gMSA в качестве замены этих учетных данных. Вы можете изменить gMSA между средами разработки, тестирования и рабочей средой, и IIS автоматически выберет текущее удостоверение без необходимости изменения образа контейнера.

## <a name="run-a-windows-service-as-network-service"></a>Запуск службы Windows в качестве сетевой службы

Если контейнерное приложение выполняется как служба Windows, можно настроить службу для работы в качестве **сетевой службы** в Dockerfile:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Запуск произвольных консольных приложений в качестве сетевой службы

Для универсальных консольных приложений, которые не размещаются в службах IIS или Service Manager, проще всего запустить контейнер как **сетевую службу** , чтобы приложение автоматически наследовало контекст gMSA. Эта функция доступна в Windows Server версии 1709.

Добавьте следующую строку в Dockerfile, чтобы она выполнялась как сетевая служба по умолчанию:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Вы также можете подключаться к контейнеру как к сетевой службе на основе одной `docker exec`. Это особенно полезно при устранении проблем с подключением в работающем контейнере, когда контейнер обычно не запускается как сетевая служба.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Дальнейшие действия

В дополнение к настройке приложений можно также использовать Gmsa для:

- [Выполнение контейнеров](gmsa-run-container.md)
- [Управление контейнерами](gmsa-orchestrate-containers.md)

Если во время установки возникнут проблемы, ознакомьтесь с нашим [руководством по устранению неполадок](gmsa-troubleshooting.md) , чтобы получить возможные решения.
