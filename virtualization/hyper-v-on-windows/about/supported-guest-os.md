---
title: Поддерживаемые гостевые ОС Windows
description: Поддерживаемые гостевые ОС Windows.
keywords: windows10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 9b19a82c94fbe9af9f141d4845a8ed1045a10302
ms.sourcegitcommit: 94e8ae4be1b0d3d13fca06e0775dd2aab895a12c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/07/2018
ms.locfileid: "1840996"
---
# <a name="supported-windows-guests"></a>Поддерживаемые гостевые ОС Windows

В этой статье приводится список сочетаний операционных систем, поддерживаемых в Hyper-V в Windows.  В ней также рассматриваются службы интеграции и другие поддерживаемые факторы.

Эти сочетания гостевых ОС и узлов протестированы корпорацией Майкрософт.  Проблемы, связанные с взаимодействием этих систем, будут рассмотрены службой технической поддержки Майкрософт.

Майкрософт предоставляет поддержку следующим образом:

* Устранение проблем, обнаруженных в операционных системах Майкрософт и в службах интеграции, поддерживается службой технической поддержки Майкрософт.

* Поддержка по проблемам в других операционных системах, сертифицированных поставщиком операционной системы для выполнения в Hyper-V, обеспечивается поставщиком.

* Если обнаруживаются проблемы в других операционных системах, корпорация Майкрософт отправляет описание проблемы в сообщество по оказанию поддержки, состоящее из множества поставщиков, [TSANet](http://www.tsanet.org/).

Для получения поддержки все операционные системы (гостевые и узловые) должны быть обновлены до последней версии.  Откройте Центр обновления Windows и проверьте наличие критических обновлений.

## <a name="supported-guest-operating-systems"></a>Поддерживаемые операционные системы на виртуальной машине

| Гостевая операционная система |  Максимальное число виртуальных процессоров. | Заметки |
|:-----|:-----|:-----|
| Windows 10 | 32 |Режим расширенного сеанса не работает в выпуске Windows 10 Домашняя |
| Windows8.1 | 32 | |
| Windows 8 | 32 ||
| Windows7 с пакетом обновления1 (SP1) | 4 | Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные). |
| Windows7 | 4 | Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные). |
| Windows Vista с пакетом обновления 2 (SP2) | 2 | Business, Enterprise и Ultimate, включая выпуски N и KN |
| - | | |
| [Windows Server Semi-Annual Channel](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2016 | 64 | |
| WindowsServer2012R2 | 64 | |
| Windows Server2012 | 64 | |
| Windows Server2008R2 с пакетом обновления1 (SP1) | 64 | Выпуски Datacenter, Enterprise, Standard и Web. |
| Windows Server2008 с пакетом обновления2 (SP2) | 4 | Выпуски Datacenter, Enterprise, Standard и Web (32-разрядные и 64-разрядные). |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Выпуск Essentials Edition — 2, Standard Edition — 4 | |

> Windows 10 можно запустить в качестве гостевой операционной системы на узлах Hyper-V с Windows 8.1 и Windows Server 2012 R2.

## <a name="supported-linux-and-free-bsd"></a>Поддерживаемые ОС Linux и FreeBSD

| Гостевая операционная система |  |
|:-----|:------|
| [CentOS и Red Hat Enterprise Linux](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Виртуальные машины Debian на базе Hyper-V](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

Дополнительные сведения, в том числе информацию о поддержке предыдущих версий Hyper-V, см. в статье [Linux and FreeBSD Virtual Machines on Hyper-V](https://technet.microsoft.com/library/dn531030.aspx) (Виртуальные машины Linux и FreeBSD на базе Hyper-V).
