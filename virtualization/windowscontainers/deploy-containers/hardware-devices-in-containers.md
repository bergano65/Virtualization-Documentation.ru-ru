---
title: Устройства в контейнерах в Windows
description: Какая поддержка устройств существует для контейнеров в Windows
keywords: Dock, контейнеры, устройства, оборудование
author: cwilhit
ms.openlocfilehash: ee9c5da5ef87dceb3374977670da2ea50ea87382
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883167"
---
# <a name="devices-in-containers-on-windows"></a>Устройства в контейнерах в Windows

По умолчанию контейнеры Windows получают минимальный доступ к ведущим устройствам, так же, как и к контейнерам Linux. Существуют определенные рабочие нагрузки, в которых она может быть полезна (или даже принудительно) для доступа к устройствам с ведущим устройством и для связи с ними. В этом руководстве рассказывается о том, какие устройства поддерживаются в контейнерах, и как приступить к работе.

## <a name="requirements"></a>Требования

Для работы этой функции необходимо, чтобы ваша среда соответствовала следующим требованиям:
- Узел контейнера должен работать под управлением Windows Server 2019 или Windows 10 версии 1809 или более поздней.
- Версия базового образа контейнера должна быть 1809 или более поздней.
- Контейнеры должны быть контейнерами Windows, работающими в режиме изолированной обработки.
- Ведущему узлу должно быть присвоено средство Dock 19,03 или более поздней версии.

## <a name="run-a-container-with-a-device"></a>Запуск контейнера с устройством

Чтобы запустить контейнер с устройством, введите следующую команду:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Необходимо заменить на `{interface class guid}` соответствующий [класс GUID класса интерфейса устройства](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes), который можно найти в разделе ниже.

Чтобы запустить контейнер с несколькими устройствами, используйте следующую команду и строку вместе с несколькими `--device` аргументами:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

В Windows все устройства объявляют список классов интерфейсов, которые они реализуют. Передав эту команду в стыковочный элемент, вы убедитесь, что все устройства, которые определяют требуемый класс, будут подключены к контейнеру.

Это означает, что устройство **не** будет назначаться с узла. Вместо этого узел предоставляет доступ к контейнеру. Точно так же, так как вы указываете GUID класса, _все_ устройства, реализующие этот GUID, будут совместно использоваться с контейнером.

## <a name="what-devices-are-supported"></a>Какие устройства поддерживаются?

В настоящее время поддерживаются следующие устройства (и GUID класса интерфейса устройства):
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Тип устройства</center></th>
<th><center>GUID класса интерфейса</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>Шина I2C</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM-порт</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>Шина SPI</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>Ускорение GPU для DirectX</center></td>
<td><center>Просмотр документов <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">ускорения GPU</a></center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Перечисленные выше устройства являются единственными __ устройствами, поддерживаемыми в контейнерах Windows уже сегодня. При попытке передать любые другие GUID класса произойдет сбой при запуске контейнера.

## <a name="hyper-v-isolated-windows-container-support"></a>Поддержка контейнера Windows с изолированной изоляцией Hyper-V

Назначение устройств и общий доступ к устройствам для рабочих нагрузок в контейнерах Windows с изоляцией Hyper-V не поддерживается в настоящее время.

## <a name="hyper-v-isolated-linux-container-support"></a>Поддержка контейнеров в среде Linux с изоляцией Hyper-V

Назначение устройств и общий доступ к устройствам для рабочих нагрузок в контейнерах Linux с изоляцией Hyper-V в настоящее время не поддерживаются.