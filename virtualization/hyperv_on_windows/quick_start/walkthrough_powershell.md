---
title: "Работа с Hyper-V и Windows PowerShell"
description: "Работа с Hyper-V и Windows PowerShell"
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: a8e567b6447aa73f14825b7054d977d2b003a726

---

# Работа с Hyper-V и Windows PowerShell

Изучив основы развертывания Hyper-V, создания виртуальных машин и управления ими, давайте теперь узнаем, как можно автоматизировать многие из связанных с этим действий с помощью PowerShell.

### Получение списка команд Hyper-V

1.  Нажмите кнопку "Пуск" в Windows и введите **PowerShell**.
2.  Запустите указанную ниже команду, чтобы отобразить список команд PowerShell, доступных в модуле PowerShell Hyper-V.

 ```powershell
get-command -module hyper-v | out-gridview
```
  Отобразится примерно следующее:

  ![](media\command_grid.png)

3. Чтобы получить дополнительную информацию о конкретной команде PowerShell, введите команду `get-help`. Например, запустив указанную ниже команду, вы получите информацию о команде `get-vm` Hyper-V.

  ```powershell
get-help get-vm
```
 Отобразится информация о синтаксисе команды, обязательных и дополнительных параметрах, а также псевдонимах, которые можно использовать.

 ![](media\get_help.png)


### Получение списка виртуальных машин

Чтобы получить список виртуальных машин, используйте команду `get-vm`.

1. В PowerShell запустите следующую команду:
 
 ```powershell
get-vm
```
 Отобразится примерно следующее:

 ![](media\get_vm.png)

2. Чтобы получить список только тех виртуальных машин, которые включены в данный момент, добавьте к команде `get-vm` фильтр. Фильтр можно добавить с помощью команды where-object. Дополнительные сведения о фильтрации см. в статье [Использование командлета Where-Object](https://technet.microsoft.com/en-us/library/ee177028.aspx).   

 ```powershell
 get-vm | where {$_.State -eq ‘Running’}
 ```
3.  Чтобы получить список всех отключенных виртуальных машин, запустите указанную ниже команду. Эта команда представляет собой копию команды, приведенной ранее (шаг 2), но только значение фильтра изменено с Running на Off.

 ```powershell
 get-vm | where {$_.State -eq ‘Off’}
 ```

### Запуск и завершение работы виртуальных машин

1. Чтобы запустить определенную виртуальную машину, выполните следующую команду с указанием имени виртуальной машины:

 ```powershell
 Start-vm -Name <virtual machine name>
 ```

2. Чтобы запустить все отключенные на данный момент виртуальные машины, получить список этих машин и передать список команде start-vm, используется следующая команда:

  ```powershell
 get-vm | where {$_.State -eq ‘Off’} | start-vm
 ```
3. Чтобы завершить работу всех работающих виртуальных машин, запустите это:
 
  ```powershell
 get-vm | where {$_.State -eq ‘Running’} | stop-vm
 ```

### Создание контрольной точки виртуальной машины

Чтобы создать контрольную точку с помощью PowerShell, выберите нужную виртуальную машину, используя команду `get-vm`, и передайте ее в команду `checkpoint-vm`. В заключение присвойте контрольной точке имя, используя команду `-snapshotname`. Полностью команда выглядит так:

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### Создание новой виртуальной машины

Следующий пример демонстрирует создание виртуальной машины в интегрированной среде сценариев (ISE) PowerShell. Это простой пример. Его можно усложнить, добавив дополнительные функции PowerShell и расширенные сценарии развертывания виртуальной машины.

1. Чтобы открыть среду ISE PowerShell, нажмите кнопку "Пуск" и введите **PowerShell ISE**.
2. Запустите указанный ниже код для создания виртуальной машины. Подробные сведения о New-VM см. в документации по команде [New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx).

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName "
     SwitchName = (get-vmswitch).Name[0]
 }

 New-VM @VM
  ```

## Подведение итогов и справочные материалы

Этот документ позволяет ознакомиться с модулем PowerShell Hyper-V на примере некоторых простых шагов, а также отдельными примерами сценариев. Дополнительные сведения о модуле PowerShell для Hyper-V см. в [справочнике по командлетам Windows PowerShell для Hyper-V](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx).  
 


<!--HONumber=Jun16_HO4-->


