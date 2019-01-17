---
title: Устройства в контейнеры в Windows
description: Какая поддержка устройства существует для контейнеров в Windows
keywords: docker, контейнеры, устройства, оборудование
author: cwilhit
ms.openlocfilehash: 6397a5050ee0c7cb4b62dc935af4975d9ab6b3db
ms.sourcegitcommit: 1b6a244c3604e48c42c851e580e3b59e2384c91a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "9014521"
---
# <a name="devices-in-containers-on-windows"></a>Устройства в контейнеры в Windows

По умолчанию контейнеры Windows предоставляются минимальный доступ устройства узла — так же, как контейнеры Linux. Существуют определенные рабочих нагрузок, там, где это полезным--или даже императивного--для доступа к и взаимодействия с устройствами узла. В этом руководстве рассматриваются устройств, которые поддерживаются в контейнерах, а также как для начала работы.

## <a name="requirements"></a>Требования

- Вы должны работать под управлением Windows Server 2019 или более поздней или Windows 10 Pro или Корпоративная с октября 2018 г. обновление
- Вашей версии образа контейнера должна быть 1809 или более поздней версии.
- Контейнеры должны быть контейнеры Windows в режиме изолированного процесса.
- Время существования возможностей устройств с Windows в управляющей программы Docker, оно еще не существует в клиента Docker (см. в разделе этого [запроса на включение](https://github.com/docker/cli/pull/1606) для отслеживания). В interrim необходимо выполнить [сборки docker исполняемые файлы](https://github.com/moby/moby/blob/master/docs/contributing/software-req-win.md) из источника moby качестве обхода. Если вы не неудобно делать это, рекомендуется отложить оценить работу этой функции, пока объединены PR, вышеописанные.

## <a name="run-a-container-with-a-device"></a>Запуск контейнера с устройством

Чтобы запустить контейнер с устройством, используйте следующую команду:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Необходимо заменить `{interface class guid}` с соответствующие [GUID класса интерфейса устройства](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes), который можно найти в разделе ниже.

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
</tbody>
</table>

> [!TIP]
> Устройства, перечисленных выше — _только_ устройства, поддерживаемые в контейнерах Windows уже сегодня. Попытка передать другие идентификаторы GUID класса приведет к ошибкам при запуске контейнера.

## <a name="hyper-v-container-device-support"></a>Поддержка устройств контейнер Hyper-V

Назначение устройств и устройств не поддерживаются в контейнеры Hyper-V изолированными сегодня.

## <a name="linux-containers-on-windows-lcow-device-support"></a>Контейнеры Linux в поддержки устройств Windows (LCOW)

Назначение устройств и устройств не поддерживаются в LCOW сегодня.