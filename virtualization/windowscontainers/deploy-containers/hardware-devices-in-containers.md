---
title: Устройства в контейнеры в Windows
description: Какая поддержка устройства существует для контейнеров в Windows
keywords: docker, контейнеры, устройства, оборудование
author: cwilhit
ms.openlocfilehash: f32ba3de347bcf968088d2f3f20f22f82166d652
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621562"
---
# <a name="devices-in-containers-on-windows"></a>Устройства в контейнеры в Windows

По умолчанию контейнеры Windows предоставляются минимальный доступ устройства узла — так же, как контейнеры Linux. Существуют определенные рабочих нагрузок, там, где это полезным--или даже императивного--для доступа к и взаимодействия с устройствами узла. В этом руководстве рассматриваются устройств, которые поддерживаются в контейнерах, а также как для начала работы.

> [!IMPORTANT]
> Для этой функции требуется версию Docker, который поддерживает `--device` параметр командной строки для контейнеров Windows. Формальный поддержка Docker запланирована ближайших выпусках 19.03 модуль Docker EE. До тех пор, [несоответствий источника](https://master.dockerproject.org/) Docker содержит необходимые биты.

## <a name="requirements"></a>Требования

Эта функция работать среду должны соответствовать следующим требованиям:
- Узла контейнера должны работать под управлением Windows Server 2019 или Windows 10, версия 1809 или более поздней версии.
- Ваш версия базового образа контейнера должна быть 1809 или более поздней версии.
- Контейнеры должны быть контейнеры Windows в режиме изолированного процесса.
- Узла контейнера должны работать под управлением подсистемы Docker 19.03 или более поздней версии.

## <a name="run-a-container-with-a-device"></a>Запуск контейнера с устройством

Чтобы запустить контейнер с устройством, используйте следующую команду:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Необходимо заменить `{interface class guid}` с соответствующие [GUID класса интерфейса устройства](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes), который можно найти в разделе ниже.

Для запуска контейнера с несколькими устройствами, используйте следующую команду и строка друг с другом несколько `--device` аргументы:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

В Windows все устройства объявите список классов интерфейса, которые они реализуют. Путем передачи этой команды Docker, он гарантирует, что все устройства, которые определяют, как реализация запрошенного класса будет шинам в контейнер.

Это означает, что вы **не** назначаются устройства от узла. Вместо этого узла предоставляет доступ к его контейнера. Аналогичным образом так как вы указываете GUID класса, _все_ устройства, которые реализуют этот идентификатор GUID будет использоваться совместно с контейнера.

## <a name="what-devices-are-supported"></a>Новые устройства поддерживают

Следующие устройства (и интерфейс GUID класса устройства) поддерживаются сегодня:
  
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
<td><center>Шины SPI</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>Ускорение DirectX GPU</center></td>
<td><center>См. в разделе выделенные документы</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Устройства, перечисленных выше — _только_ устройства, поддерживаемые в контейнерах Windows уже сегодня. Попытка передать другие идентификаторы GUID класса приведет к ошибкам при запуске контейнера.

## <a name="hyper-v-isolated-windows-container-support"></a>Поддержка контейнеров Hyper-V-изолированными Windows

Назначение устройств и устройства общего доступа для рабочих нагрузок в контейнерах Windows с изоляцией Hyper-V сейчас не поддерживается.

## <a name="hyper-v-isolated-linux-container-support"></a>Поддержка контейнеров Hyper-V-изолированными Linux

Назначение устройств и устройства общего доступа для рабочих нагрузок в контейнеры Linux с изоляцией Hyper-V сейчас не поддерживается.
