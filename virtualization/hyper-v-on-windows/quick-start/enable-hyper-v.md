---
title: Включение Hyper-V в Windows10
description: Установка Hyper-V в Windows10
keywords: windows10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: dd4021c653d98a3bd7a338d9f502a572830f344d
ms.sourcegitcommit: 5e5644bff6dba70e384db6c80787b3bbe7adb93c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/03/2018
ms.locfileid: "4303930"
---
# <a name="install-hyper-v-on-windows-10"></a>Установка Hyper-V вWindows10

Включение Hyper-V для создания виртуальных машин в Windows 10.  
Hyper-V можно включить разными способами, в том числе используя панель управления Windows10, PowerShell или с помощью средства обслуживания образов развертывания и управления ими (DISM). В этом документе последовательно описан каждый из указанных способов.

> **Примечание.** Механизм Hyper-V встроен в Windows в качестве дополнительной функции. Скачать Hyper-V нельзя.

## <a name="check-requirements"></a>Проверьте следующие требования

* Windows 10 Корпоративная, Профессиональная или для образовательных учреждений
* 64-разрядный процессор с поддержкой преобразования адресов второго уровня (SLAT).
* Поддержка расширения режима мониторинга виртуальной машины (технология VT-c на компьютерах с процессорами Intel).
* Не менее 4ГБ оперативной памяти.

Роль Hyper-V **невозможно** установить в Windows 10 Домашняя.

Выполните обновление с ОС Windows 10 Домашняя до Windows 10 Профессиональная, открыв **Параметры** > **обновление и безопасность** > **активации**.

Дополнительные сведения и советы по устранению неполадок см. в статье [Требования к системе для Hyper-V в Windows10](../reference/hyper-v-requirements.md).

## <a name="enable-hyper-v-using-powershell"></a>Включение Hyper-V с помощью PowerShell

1. Откройте консоль PowerShell от имени администратора.

2. Выполните следующую команду.

  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  ```

  Если не удается найти команду, убедитесь, что вы используете PowerShell от имени администратора.

После завершения установки выполните перезагрузку.

## <a name="enable-hyper-v-with-cmd-and-dism"></a>Включение Hyper-V с помощью CMD и DISM

Система обслуживания образов развертывания и управления ими (DISM) позволяет настраивать ОС Windows и образы Windows.  Помимо всего прочего? средство DISM может включать функции Windows во время выполнения операционной системы.

Чтобы включить роль Hyper-V с помощью DISM, выполните указанные ниже действия.

1. Запустите PowerShell или сеанс CMD от имени администратора.

1. Введите следующую команду:

  ```powershell
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  ```

  ![Окно консоли с процессом включения Hyper-V.](media/dism_upd.png)

Дополнительные сведения о DISM см. в разделе [Техническое руководство по DISM](https://technet.microsoft.com/en-us/library/hh824821.aspx).

## <a name="enable-the-hyper-v-role-through-settings"></a>Включение роли Hyper-V через раздел "Параметры"

1. Щелкните правой кнопкой мыши кнопку Windows и выберите пункт "Приложения и компоненты".

2. Выберите пункт **Включение или отключение компонентов Windows**.

3. Выберите **Hyper-V** и нажмите кнопку **ОК**.

![Диалоговое окно "Программы и компоненты" Windows](media/enable_role_upd.png)

После завершения установки вам будет предложено перезапустить компьютер.

## <a name="make-virtual-machines"></a>Создание виртуальных машин

[Создание первой виртуальной машины](quick-create-virtual-machine.md)
