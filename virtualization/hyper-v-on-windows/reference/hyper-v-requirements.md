---
title: Требования к системе для Hyper-V в Windows 10
description: Требования к системе для Hyper-V в Windows 10
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: d3375cd912097f85f0a350b8f329c008323cab37
ms.sourcegitcommit: cea415924b7b6a690d0ba9ff31beed30e9c187d2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/25/2020
ms.locfileid: "76750195"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Требования к системе для Hyper-V в Windows 10

Hyper-V доступен в 64-разрядной версии Windows 10 профессиональная, Корпоративная и для образовательных учреждений. Для Hyper-V требуется функция преобразования адресов второго уровня (SLAT). Она есть в текущем поколении 64-разрядных процессоров Intel и AMD.

На узле, имеющем 4 ГБ оперативной памяти, можно запустить три-четыре базовые виртуальные машины, однако для большего числа виртуальных машин потребуется больше ресурсов. Кроме того, можно создать мощные виртуальные машины с 32 процессорами и 512 ГБ ОЗУ в зависимости от оборудования.

## <a name="operating-system-requirements"></a>Требования к операционной системе

Роль Hyper-V можно включить в таких версиях Windows 10:

- Windows 10 Корпоративная
- Windows 10 Pro
- Windows 10 для образовательных учреждений

Роль Hyper-V **невозможно** установить в следующих версиях:

- Windows 10 Домашняя
- Windows 10 Mobile
- Windows 10 Mobile Корпоративная

>Windows 10 Домашняя версия может быть обновлена до Windows 10 Pro. Для этого перейдите в раздел **Параметры** > **Обновление и безопасность** > **Активация**. Здесь вы можете посетить Магазин Windows и приобрести обновление.

## <a name="hardware-requirements"></a>Требования к оборудованию

Хотя в этом документе не приводится полный список оборудования, совместимого с Hyper-V, укажем следующие обязательные требования:

- 64-разрядный процессор с поддержкой преобразования адресов второго уровня (SLAT).
- Поддержка ЦП для расширения режима монитора ВМ (VT-x на ЦП Intel).
- Не менее 4 ГБ оперативной памяти. Так как виртуальные машины и узел Hyper-V используют память совместно, необходимо обеспечить достаточный объем памяти для обработки предполагаемой рабочей нагрузки на виртуальной машине.

В BIOS системы необходимо включить следующие компоненты.
- Virtualization Technology (Технология виртуализации) — может иметь другое название в зависимости от производителя системной платы.
- Hardware Enforced Data Execution Prevention (Принудительное аппаратное предотвращение выполнения данных).

## <a name="verify-hardware-compatibility"></a>Проверка совместимости оборудования

Чтобы проверить совместимость, откройте PowerShell или командную строку (cmd.exe) и введите **systeminfo**. Если для всех перечисленных требований для Hyper-V указан ответ **Yes**, то ваша система поддерживает роль Hyper-V. Если хотя бы для одного пункта указан ответ **No**, проверьте указанные выше требования и внесите необходимые изменения.

![](media/SystemInfo-upd.png)

Если команда **systeminfo** запускается на существующем узле Hyper-V, в разделе Hyper-V Requirements отображается следующее сообщение:

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```
