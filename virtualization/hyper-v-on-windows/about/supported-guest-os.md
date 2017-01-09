---
title: "Поддерживаемые гостевые ОС Windows"
description: "Поддерживаемые гостевые ОС Windows."
keywords: "windows 10, hyper-v"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: f772a26f2ad52c8ed28d40ca35c91ed6f3eab872

---

# Поддерживаемые гостевые ОС Windows 

В этой статье приводится список сочетаний операционных систем, поддерживаемых в Hyper-V в Windows.  В ней также рассматриваются службы интеграции и другие поддерживаемые факторы.

## Что подразумевается под поддержкой? 
Под поддержкой понимается, что эти сочетания гостевых ОС и узлов протестированы корпорацией Майкрософт.  Проблемы, связанные с взаимодействием этих систем, будут рассмотрены службой технической поддержки Майкрософт.
 
Корпорация Майкрософт обеспечивает поддержку гостевых операционных систем следующим образом:
* Устранение проблем, обнаруженных в операционных системах Майкрософт и в службах интеграции, поддерживается службой технической поддержки Майкрософт.
* Поддержка по проблемам в других операционных системах, сертифицированных поставщиком операционной системы для выполнения в Hyper-V, обеспечивается поставщиком.
* Если обнаруживаются проблемы в других операционных системах, корпорация Майкрософт отправляет описание проблемы в сообщество по оказанию поддержки, состоящее из множества поставщиков, [TSANet](http://www.tsanet.org/).

Поддержка предоставляется, только если на узле Hyper-V и в гостевой операционной системе установлены все критические обновления, доступные через Центр обновления Windows.

## Поддерживаемые операционные системы на виртуальной машине

Поддержка предоставляется, только если в гостевых операционных системах Windows и операционной системе узла установлены все критические обновления, доступные через Центр обновления Windows.

| Гостевая операционная система |  Максимальное число виртуальных процессоров. | Заметки | 
|:-----|:-----|:-----|
| Windows 10 | 32 | |
| Windows 8.1 | 32 | |
| Windows 8 | 32 |  |
| Windows 7 с пакетом обновления 1 (SP1) | 4 | Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные). |
| Windows 7 | 4 | Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные). |
| Windows Vista с пакетом обновления 2 (SP2) | 2 | Business, Enterprise и Ultimate, включая выпуски N и KN | 
| - | | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 с пакетом обновления 1 (SP1) | 64 | Выпуски Datacenter, Enterprise, Standard и Web. |
| Windows Server 2008 с пакетом обновления 2 (SP2) | 4 | Выпуски Datacenter, Enterprise, Standard и Web (32-разрядные и 64-разрядные). |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Выпуск Essentials Edition — 2, Standard Edition — 4 | |
  
 > Windows 10 можно запустить в качестве гостевой операционной системы на узлах Hyper-V с Windows 8.1 и Windows Server 2012 R2.

## Поддерживаемые ОС Linux и FreeBSD

| Гостевая операционная система |  |
|:-----|:------|
| [CentOS и Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Виртуальные машины Debian на базе Hyper-V](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

Дополнительные сведения, в том числе информацию о поддержке предыдущих версий Hyper-V, см. в статье [Linux and FreeBSD Virtual Machines on Hyper-V](https://technet.microsoft.com/library/dn531030.aspx) (Виртуальные машины Linux и FreeBSD на базе Hyper-V).



<!--HONumber=Oct16_HO4-->


