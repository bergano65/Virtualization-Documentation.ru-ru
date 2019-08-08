---
title: Журнал базовых образов для контейнеров Windows
description: Список образов для контейнеров Windows, выпущенных с хэшами уровня SHA256
keywords: docker, контейнеры, хэши
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 0ec6eccbcf69532d583c32136f1a0c50c9811a8b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998371"
---
# <a name="windows-container-base-image-history"></a>Журнал базовых образов для контейнеров Windows

Каждый контейнер Windows строится на основе базовой ОС, предоставляемой корпорацией Майкрософт. Если вы не знаете, для какой версии Windows был создан контейнер, можно выполнить `docker inspect <tag>`и сопоставить один или два верхних столбца с таблицей ниже.

Например, при выполнении `docker inspect microsoft/windowsservercore:10.0.14393.447` отобразится следующее:

```
...
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67",
        "sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15"
    ]
}
```

Которые являются двумя уровнями в образе, предоставляемом корпорацией Майкрософт. Верхний уровень не изменяется и представляет исходный выпуск Windows Server, а второй уровень меняется в зависимости от включенных накопительных пакетов обновления.

Если вы хотите узнать, какие изменения были внесены в каждую версию, выполните поиск соответствующей версии по базе знаний в разделе [Журнал обновлений Windows 10 и Windows Server 2016](https://support.microsoft.com/help/12387/windows-10-update-history)


## <a name="tools-to-simplify-this-process"></a>Инструменты для упрощения этого процесса

Стефан Шерер (Stefan Scherer) создал инструмент, который может считывать манифест изображения и определять версию без скачивания полного контейнера. Дополнительные сведения см. в его [блоге](https://stefanscherer.github.io/winspector/) и в репозитории [GitHub](https://github.com/StefanScherer/winspector).


## <a name="image-versions"></a>Версии образов

<table>
    <tr>
        <th>Версия Windows</th>
        <th>microsoft/windowsservercore</th>
        <th>microsoft/nanoserver</th>
    </tr>
    <tr>
        <td>10.0.14393.206</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063</td>
    </tr>
    <tr>
        <td>10.0.14393.321</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:cc6b0a07c696c3679af48ab4968de1b42d35e568f3d1d72df21f0acb52592e0b</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:2c195a33d84d936c7b8542a8d9890a2a550e7558e6ac73131b130e5730b9a3a5</td>
    </tr>
    <tr>
        <td>10.0.14393.447</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:c8606bedb07a714a6724b8f88ce85b71eaf5a1c80b4c226e069aa3ccbbe69154</td>
    </tr>
    <tr>
        <td>10.0.14393.576</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:de57d9086f9a337bb084b78f5f37be4c8f1796f56a1cd3ec8d8d1c9c77eb693c</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:0d812bf7a7032db75770c3d5b92c0ac9390ca4a9efa0d90ba2f55ccb16515381</td>
    </tr>
    <tr>
        <td>10.0.14393.693</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:c28d44287ce521eac86e0296c7677f5d8ca1e86d1e45e7618ec900da08c95df3</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:dd33c5d8d8b3c230886132c328a7801547f13de1dac9a629e2739164a285b3ab</td>
    </tr>
</table>

