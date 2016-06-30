---
title: HCS PowerShell
description: "Работа с контейнерами Windows и HCS PowerShell."
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 45144ec5-f76a-4460-abd1-9b60e47506d6
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 413b9de08d182635908bc11ce3efc7623bb440e4

---

# Совместимость средств управления

**Это предварительное содержимое. Возможны изменения.** 

## Отображение всех контейнеров

Чтобы отобразить список контейнеров, используйте команду `Get-ComputeProcess`.

```none
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## Остановка контейнера

Чтобы остановить контейнер, независимо от способа его создания (с помощью PowerShell или Docker), используйте команду `Stop-ComputeProcess`.

> На момент написания этой статьи: чтобы контейнеры отображались как остановленные при использовании команды `Get-Container`, требуется перезапустить службу VMMS.

```none
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```



<!--HONumber=Jun16_HO4-->


