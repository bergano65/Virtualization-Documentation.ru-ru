---
title: Поддерживаемые гостевые ОС Windows
description: Поддерживаемые гостевые ОС Windows.
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &800292699 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
---

# Поддерживаемые гостевые ОС Windows

В этой статье приводится список сочетаний операционных систем, поддерживаемых в Hyper-V в Windows. В ней также рассматриваются службы интеграции и другие поддерживаемые факторы.

## Что подразумевается под поддержкой?

Под поддержкой понимается, что эти сочетания гостевых ОС и узлов протестированы корпорацией Майкрософт. Проблемы, связанные с взаимодействием этих систем, будут рассмотрены службой технической поддержки Майкрософт.

Корпорация Майкрософт обеспечивает поддержку гостевых операционных систем следующим образом:
* Устранение проблем, обнаруженных в операционных системах Майкрософт и в службах интеграции, поддерживается службой технической поддержки Майкрософт.
* Поддержка по проблемам в других операционных системах, сертифицированных поставщиком операционной системы для выполнения в Hyper-V, обеспечивается поставщиком.
* Если обнаруживаются проблемы в других операционных системах, корпорация Майкрософт отправляет описание проблемы в сообщество поддержки разных поставщиков <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">TSANet</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

Поддержка предоставляется, только если на узле Hyper-V и в гостевой операционной системе установлены все критические обновления, доступные через Центр обновления Windows.

## Поддерживаемые операционные системы на виртуальной машине

Поддержка предоставляется, только если в гостевых операционных системах Windows и операционной системе узла установлены все критические обновления, доступные через Центр обновления Windows.

| Гостевая операционная система| Максимальное число виртуальных процессоров.| Заметки|
|:-----|:-----|:-----|
| Windows 10| 32| |
| Windows 8.1| 32| |
| Windows 8| 32| |
| Windows 7 с пакетом обновления 1 (SP1)| 4| Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные).|
| Windows 7| 4| Выпуски Максимальная, Корпоративная и Профессиональная (32-разрядные и 64-разрядные).|
| Windows Vista с пакетом обновления 2 (SP2)| 2| Business, Enterprise и Ultimate, включая выпуски N и KN|
| —| | |
| Windows Server 2012 R2| 64| |
| Windows Server 2012| 64| |
| Windows Server 2008 R2 с пакетом обновления 1 (SP1)| 64| Выпуски Datacenter, Enterprise, Standard и Web.|
| Windows Server 2008 с пакетом обновления 2 (SP2)| 4| Выпуски Datacenter, Enterprise, Standard и Web (32-разрядные и 64-разрядные).|
| Windows Home Server 2011| 4| |
| Windows Small Business Server 2011| Выпуск Essentials Edition — 2, Standard Edition — 4| |

> Windows 10 можно запустить в качестве гостевой операционной системы на узлах Hyper-V с Windows 8.1 и Windows Server 2012 R2.

## Поддерживаемые ОС Linux и FreeBSD

| Гостевая операционная система| |
|:-----|:------|
| <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">CentOS и Red Hat Enterprise Linux </g><g id="1CapsExtId3" ctype="x-title"></g></g>| |
| <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Виртуальные машины Debian на базе Hyper-V</g><g id="1CapsExtId3" ctype="x-title"></g></g>| |
| <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">SUSE</g><g id="1CapsExtId3" ctype="x-title"></g></g>| |
| <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Oracle Linux</g><g id="1CapsExtId3" ctype="x-title"></g></g>| |
| <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Ubuntu</g><g id="1CapsExtId3" ctype="x-title"></g></g>| |
| <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">FreeBSD</g><g id="1CapsExtId3" ctype="x-title"></g></g>| |

Дополнительные сведения, в том числе информацию о поддержке предыдущих версий Hyper-V, см. в статье <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Виртуальные машины Linux и FreeBSD на базе Hyper-V</g><g id="2CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=May16_HO1-->


