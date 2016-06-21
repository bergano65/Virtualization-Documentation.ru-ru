---
title: &675317387 Работа с Hyper-V и Windows PowerShell
description: Работа с Hyper-V и Windows PowerShell
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1488941761 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
---

# Работа с Hyper-V и Windows PowerShell

Изучив основы развертывания Hyper-V, создания виртуальных машин и управления ими, давайте теперь узнаем, как можно автоматизировать многие из связанных с этим действий с помощью PowerShell.

### Получение списка команд Hyper-V

1.  Нажмите кнопку "Пуск" в Windows и введите <g id="2" ctype="x-strong">PowerShell</g>.
2.  Запустите указанную ниже команду, чтобы отобразить список команд PowerShell, доступных в модуле PowerShell Hyper-V.

 ```powershell
get-command -module hyper-v | out-gridview
 ```
  Отобразится примерно следующее:

  <g id="1" ctype="x-linkText"></g>

3. Чтобы получить дополнительные сведения о конкретной команде PowerShell, введите команду <g id="2" ctype="x-code">get-help</g>. Например, запустив указанную ниже команду, вы получите сведения о команде <g id="2" ctype="x-code">get-vm</g> для Hyper-V.

  ```powershell
get-help get-vm
  ```
 Отобразится информация о синтаксисе команды, обязательных и дополнительных параметрах, а также псевдонимах, которые можно использовать.

 <g id="1" ctype="x-linkText"></g>


### Получение списка виртуальных машин

Чтобы получить список виртуальных машин, используйте команду <g id="2" ctype="x-code">get-vm</g>.

1. В PowerShell запустите следующую команду:

 ```powershell
get-vm
 ```
 Отобразится примерно следующее:

 <g id="1" ctype="x-linkText"></g>

2. Чтобы получить список только тех виртуальных машин, которые включены в данный момент, добавьте к команде <g id="2" ctype="x-code">get-vm</g> фильтр. Фильтр можно добавить с помощью команды where-object. Дополнительные сведения о фильтрации см. в статье <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Использование командлета Where-Object</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

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

Чтобы создать контрольную точку с помощью PowerShell, выберите нужную виртуальную машину, используя команду <g id="2" ctype="x-code">get-vm</g>, и передайте ее команде <g id="4" ctype="x-code">checkpoint-vm</g>. В заключение присвойте контрольной точке имя, используя команду <g id="2" ctype="x-code">-snapshotname</g>. Полностью команда выглядит так:

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### Создание новой виртуальной машины

Следующий пример демонстрирует создание виртуальной машины в интегрированной среде сценариев (ISE) PowerShell. Это простой пример. Его можно усложнить, добавив дополнительные функции PowerShell и расширенные сценарии развертывания виртуальной машины.

1. Чтобы открыть среду ISE PowerShell, нажмите кнопку "Пуск" и введите <g id="2" ctype="x-strong">PowerShell ISE</g>.
2. Запустите указанный ниже код для создания виртуальной машины. Подробные сведения о New-VM см. в документации по команде <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">New-VM</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

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

Этот документ позволяет ознакомиться с модулем PowerShell Hyper-V на примере некоторых простых шагов, а также отдельными примерами сценариев. Дополнительные сведения о модуле PowerShell Hyper-V см. в <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">справочнике по командлетам Windows PowerShell для Hyper-V</g><g id="2CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=May16_HO1-->


