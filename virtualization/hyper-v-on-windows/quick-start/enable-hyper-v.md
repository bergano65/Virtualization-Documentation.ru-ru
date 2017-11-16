---
title: "Включение Hyper-V в Windows10"
description: "Установка Hyper-V в Windows10"
keywords: windows10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: 85f6b30544fc433c9d768c2167024d097a9cef25
ms.sourcegitcommit: a3c9101a05d730d4173720d224bd8df8e9a56739
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/10/2017
---
# <a name="install-hyper-v-on-windows-10"></a>Установка Hyper-V вWindows10

Включение Hyper-V для создания виртуальных машин в Windows 10.  
Hyper-V можно включить разными способами, в том числе используя панель управления Windows10, PowerShell или с помощью средства обслуживания образов развертывания и управления ими (DISM). В этом документе последовательно описан каждый из указанных способов.

> **Примечание.**  Механизм Hyper-V встроен в Windows в качестве дополнительной функции. Скачать или установить Hyper-V нельзя. 

## <a name="check-requirements"></a>Проверьте следующие требования

* Windows 10 Корпоративная, Профессиональная или для образовательных учреждений
* 64-разрядный процессор с поддержкой преобразования адресов второго уровня (SLAT).
* Поддержка расширения режима мониторинга виртуальной машины (технология VT-c на компьютерах с процессорами Intel).
* Не менее 4ГБ оперативной памяти.

Роль Hyper-V **невозможно** установить в Windows 10 Домашняя.  
Выполните обновление с выпуска Windows 10 Домашняя до выпуска Windows 10 Профессиональная, открыв раздел **Параметры** > **Обновление и безопасность** > **Активация**.

Дополнительные сведения и советы по устранению неполадок см. в статье [Требования к системе для Hyper-V в Windows10](../reference/hyper-v-requirements.md).


## <a name="install-hyper-v"></a>Установка Hyper-V 
Механизм Hyper-V встроен в Windows в качестве дополнительной функции. Скачать или установить Hyper-V нельзя.  Существует несколько способов включения встроенной роли Hyper-V.

### <a name="enable-hyper-v-using-powershell"></a>Включение Hyper-V с помощью PowerShell

1. Откройте консоль PowerShell от имени администратора.

2. Выполните следующую команду.
  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName:Microsoft-Hyper-V -All
  ```  

  Если не удается найти команду, убедитесь, что вы используете PowerShell от имени администратора.  

После завершения установки необходимо перезагрузить компьютер.  

### <a name="enable-hyper-v-with-cmd-and-dism"></a>Включение Hyper-V с помощью CMD и DISM

Система обслуживания образов развертывания и управления ими (DISM) позволяет настраивать ОС Windows и образы Windows.  Помимо всего прочего? средство DISM может включать функции Windows во время выполнения операционной системы.  

Чтобы включить роль Hyper-V с помощью DISM, выполните указанные ниже действия.
1. Запустите PowerShell или сеанс CMD от имени администратора.

2. Введите следующую команду:  
  ```powershell
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  ```  
  ![](media/dism_upd.png)

Дополнительные сведения о DISM см. в разделе [Техническое руководство по DISM](https://technet.microsoft.com/en-us/library/hh824821.aspx).

### <a name="manually-enable-the-hyper-v-role"></a>Включение роли Hyper-V вручную

1. Щелкните правой кнопкой мыши кнопку Windows и выберите пункт "Приложения и компоненты".

2. Выберите пункт **Включение или отключение компонентов Windows**.

3. Выберите **Hyper-V** и нажмите кнопку **ОК**.  

![](media/enable_role_upd.png)

После завершения установки вам будет предложено перезапустить компьютер.

![](media/restart_upd.png)


## <a name="make-virtual-machines"></a>Создание виртуальных машин
[Создание первой виртуальной машины](quick-create-virtual-machine.md)
