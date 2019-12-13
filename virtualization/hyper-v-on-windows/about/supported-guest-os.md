---
title: Поддерживаемые гостевые ОС Windows
description: Поддерживаемые гостевые ОС Windows.
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: e3255d236a3fbb5ac4d908143750b84e3db82ceb
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911684"
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
| Windows 10 | 32 |Режим расширенного сеанса не работает в выпуске Windows 10 Домашняя |
| Windows 8.1 | 32 | |
| Windows 8 | 32 ||
| Windows 7 с пакетом обновления 1 (SP1) | 4 | Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные). |
| Windows 7 | 4 | Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные). |
| Windows Vista с пакетом обновления 2 (SP2) | 2 | Business, Enterprise и Ultimate, включая выпуски N и KN |
| - | | |
| [Windows Server Semi-Annual Channel](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2019 | 64 | |
| Windows Server 2016 | 64 | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 с пакетом обновления 1 (SP1) | 64 | Выпуски Datacenter, Enterprise, Standard и Web. |
| Windows Server 2008 с пакетом обновления 2 (SP2) | 4 | Выпуски Datacenter, Enterprise, Standard и Web (32-разрядные и 64-разрядные). |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Выпуск Essentials Edition — 2, Standard Edition — 4 | |

> Windows 10 можно запустить в качестве гостевой операционной системы на узлах Hyper-V с Windows 8.1 и Windows Server 2012 R2.

## <a name="supported-linux-and-free-bsd"></a>Поддерживаемые ОС Linux и FreeBSD

| Гостевая операционная система |  |
|:-----|:------|
| [CentOS и Red Hat Enterprise Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-CentOS-and-Red-Hat-Enterprise-Linux-virtual-machines-on-Hyper-V) | |
| [Debian виртуальные машины в Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Debian-virtual-machines-on-Hyper-V) | |
| [SUSE](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-SUSE-virtual-machines-on-Hyper-V) | |
| [Oracle Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Oracle-Linux-virtual-machines-on-Hyper-V)  | |
| [Ubuntu](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Ubuntu-virtual-machines-on-Hyper-V) | |
| [FreeBSD](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-FreeBSD-virtual-machines-on-Hyper-V) | |

Дополнительные сведения, в том числе информацию о поддержке предыдущих версий Hyper-V, см. в статье [Linux and FreeBSD Virtual Machines on Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows) (Виртуальные машины Linux и FreeBSD на базе Hyper-V).
