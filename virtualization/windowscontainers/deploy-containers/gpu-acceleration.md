---
title: Ускорение GPU в контейнерах Windows
description: Какой уровень ускорения GPU существует в контейнерах Windows
keywords: DOCKER, контейнеры, устройства, оборудование
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909914"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Ускорение GPU в контейнерах Windows

Для многих контейнерных рабочих нагрузок вычислительные ресурсы ЦП обеспечивают достаточную производительность. Тем не менее, для определенного класса рабочей нагрузки вычислительная мощность, обеспечиваемая графическими процессорами (графические процессоры), может ускорить операции в порядке, увеличивая затраты и повышая пропускную способность.

Графические процессоры уже являются распространенным средством для многих популярных рабочих нагрузок, от традиционных средств визуализации и моделирования до обучения и вывода машинного обучения. Контейнеры Windows поддерживают ускорение GPU для DirectX и всех платформ, построенных на его основе.

> [!NOTE]
> Эта функция доступна в DOCKER Desktop, версии 2,1 и DOCKER Engine — Enterprise, версии 19,03 или более поздней.

## <a name="requirements"></a>Требования

Чтобы эта функция работала, среда должна соответствовать следующим требованиям.

- Узел контейнера должен работать под управлением Windows Server 2019 или Windows 10 версии 1809 или более поздней.
- Базовый образ контейнера должен быть [MCR.Microsoft.com/Windows:1809](https://hub.docker.com/_/microsoft-windows) или более поздней версии. Образы контейнеров Windows Server Core и Nano Server в настоящее время не поддерживаются.
- На узле контейнера должен быть установлен модуль DOCKER 19,03 или более поздней версии.
- На узле контейнера должны быть установлены драйверы дисплея версии WDDM 2,5 или более поздней.

Чтобы проверить версию WDDM драйверов, запустите средство диагностики DirectX (dxdiag. exe) на узле контейнера. На вкладке "дисплей" средства найдите раздел "драйверы", как показано ниже.

![Работа](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Запуск контейнера с ускорением GPU

Чтобы запустить контейнер с ускорением GPU, выполните следующую команду:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (и все основанные на нем платформы) — это единственные API, которые можно ускорить с помощью GPU. платформы сторонних производителей не поддерживаются.

## <a name="hyper-v-isolated-windows-container-support"></a>Поддержка контейнеров Windows, изолированных от Hyper-V

Ускорение GPU для рабочих нагрузок в контейнерах Windows, изолированных с Hyper-V, в настоящее время не поддерживается.

## <a name="hyper-v-isolated-linux-container-support"></a>Поддержка контейнеров Linux, изолированных с Hyper-V

Ускорение GPU для рабочих нагрузок в контейнерах Linux, изолированных с Hyper-V, в настоящее время не поддерживается.

## <a name="more-information"></a>Дополнительные сведения

Полный пример контейнерного приложения DirectX, использующего ускорение GPU, см. в разделе [Пример контейнера DirectX](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx).
