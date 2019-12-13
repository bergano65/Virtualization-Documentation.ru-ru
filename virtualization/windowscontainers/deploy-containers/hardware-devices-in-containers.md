---
title: Устройства в контейнерах в Windows
description: Какая поддержка устройств существует для контейнеров в Windows
keywords: DOCKER, контейнеры, устройства, оборудование
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910604"
---
# <a name="devices-in-containers-on-windows"></a>Устройства в контейнерах в Windows

По умолчанию контейнеры Windows получают минимальный доступ к устройствам размещения, так же как и к контейнерам Linux. Существуют определенные рабочие нагрузки, которые могут быть полезными, или даже принудительно, для доступа к устройствам узла и взаимодействия с ними. В этом руководстве рассказывается о том, какие устройства поддерживаются в контейнерах и как приступить к работе.

## <a name="requirements"></a>Требования

Чтобы эта функция работала, среда должна соответствовать следующим требованиям.
- Узел контейнера должен работать под управлением Windows Server 2019 или Windows 10 версии 1809 или более поздней.
- Версия базового образа контейнера должна быть 1809 или более поздней.
- Контейнеры должны быть контейнерами Windows, работающими в режиме изоляции процессов.
- На узле контейнера должен быть установлен модуль DOCKER 19,03 или более поздней версии.

## <a name="run-a-container-with-a-device"></a>Запуск контейнера с устройством

Чтобы запустить контейнер с устройством, используйте следующую команду:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Необходимо заменить `{interface class guid}` соответствующим [идентификатором GUID класса интерфейса устройства](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes), который можно найти в разделе ниже.

Чтобы запустить контейнер с несколькими устройствами, используйте следующую команду и строку вместе с несколькими `--device` аргументами:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

В Windows все устройства объявляют список классов интерфейса, которые они реализуют. Передав эту команду в DOCKER, вы убедитесь, что все устройства, которые определяют, как реализует запрошенный класс, будут подключены к контейнеру.

Это означает, что вы **не** назначаете устройство вне узла. Вместо этого узел предоставляет доступ к контейнеру. Аналогично, поскольку вы указываете GUID класса, _все_ устройства, реализующие этот GUID, будут совместно использоваться с контейнером.

## <a name="what-devices-are-supported"></a>Поддерживаемые устройства

Сегодня поддерживаются следующие устройства (и GUID классов интерфейсов устройств):
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Device Type</center></th>
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
<td><center>Ускорение GPU DirectX</center></td>
<td><center>См. документацию по <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">ускорению GPU</a></center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> Поддержка устройств зависит от драйвера. Попытка передать идентификаторы GUID класса, не определенные в приведенной выше таблице, может привести к неопределенному поведению.

## <a name="hyper-v-isolated-windows-container-support"></a>Поддержка контейнеров Windows, изолированных от Hyper-V

Назначение устройств и общий доступ к устройствам для рабочих нагрузок в контейнерах Windows, изолированных с Hyper-V, в настоящее время не поддерживается.

## <a name="hyper-v-isolated-linux-container-support"></a>Поддержка контейнеров Linux, изолированных с Hyper-V

Назначение устройств и общий доступ к устройствам для рабочих нагрузок в контейнерах Linux, изолированных с Hyper-V, в настоящее время не поддерживается.
