---
title: Включение Hyper-V в Windows10
description: Установка Hyper-V в Windows10
keywords: windows10, hyper-v
author: scooley
ms.date: 02/15/2019
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: bad59fcc65bf66ab3c6dc940a17111e46a9bc226
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999031"
---
# <a name="install-hyper-v-on-windows-10"></a>Установка Hyper-V вWindows10

Включение Hyper-V для создания виртуальных машин в Windows 10.  
Технология Hyper-V может быть включена многими способами, в том числе с помощью панели управления Windows 10, PowerShell или с помощью средства обслуживания и управления развертыванием образов системы (DISM). В этом документе последовательно описан каждый из указанных способов.

> **Примечание.** Механизм Hyper-V встроен в Windows в качестве дополнительной функции. Скачать Hyper-V нельзя.

## <a name="check-requirements"></a>Проверьте следующие требования

* Windows 10 корпоративный, профессиональная и Office 365 для образовательных учреждений
* 64-разрядный процессор с поддержкой преобразования адресов второго уровня (SLAT).
* Поддержка ЦП для расширения режима мониторинга виртуальных машин (VT-c на процессорах Intel).
* Не менее 4ГБ оперативной памяти.

Роль Hyper-V **невозможно** установить в Windows 10 Домашняя.

Переход с Windows 10 Home Edition на Windows 10 Pro путем открытия обновления **параметров** > **и активации системы безопасности** > ****.

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

Дополнительные сведения о DISM см. в разделе [Техническое руководство по DISM](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh824821(v=win.10)>).

## <a name="enable-the-hyper-v-role-through-settings"></a>Включение роли Hyper-V через раздел "Параметры"

1. Щелкните правой кнопкой мыши кнопку Windows и выберите пункт "Приложения и компоненты".

2. В разделе связанные параметры выберите пункт **программы и компоненты** справа. 

3. Выберите пункт **Включение или отключение компонентов Windows**.

4. Выберите **Hyper-V** и нажмите кнопку **ОК**.

![Диалоговое окно "Программы и компоненты" Windows](media/enable_role_upd.png)

После завершения установки вам будет предложено перезапустить компьютер.

## <a name="make-virtual-machines"></a>Создание виртуальных машин

[Создание первой виртуальной машины](quick-create-virtual-machine.md)
