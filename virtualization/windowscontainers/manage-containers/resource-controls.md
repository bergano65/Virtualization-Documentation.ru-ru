---
title: "Реализация элементов управления ресурсами"
description: "Сведения об элементах управления ресурсами для контейнеров Windows"
keywords: "docker, контейнеры, цп, память, диск, ресурсы"
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: d3eb7e2b751468953a152e8c723551fb3e1d12dd
ms.sourcegitcommit: a072513214b0dabb9dba20ce43ea52aaf7806c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/01/2018
---
# <a name="implementing-resource-controls-for-windows-containers"></a>Реализация элементов управления ресурсами для контейнеров Windows
Существует несколько элементов управления ресурсами, которые можно реализовать на уровне контейнера и ресурса.  По умолчанию управление контейнерами осуществляется как типичными ресурсами Windows, которое обычно основано на справедливом распределении, однако с помощью конфигурации этих элементов управления разработчик или администратор может ограничить использование ресурсов.  Можно управлять следующими ресурсами: ЦП или процессор, память или ОЗУ, диск или хранилище, сеть или пропускная способность.
Контейнеры Windows используют [объекты заданий]( https://msdn.microsoft.com/en-us/library/windows/desktop/ms684161(v=vs.85).aspx) для группировки и отслеживания процессов, связанных с каждым контейнером.  Элементы управления ресурсами реализуются в родительском объекте задания, связанном с контейнером.  При использовании [изоляции Hyper-V](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/index#windows-container-types) элементы управления ресурсами автоматически применяются как к виртуальной машине, так и к объекту задания контейнера, который выполняется на виртуальной машине. Даже если процесс, выполняемый в контейнере, обойдет элементы управления объектами заданий, виртуальная машина все равно обеспечит применение заданных элементов управления ресурсами.

## <a name="resources"></a>Ресурсы
Для каждого ресурса в этом разделе представлено сопоставление между интерфейсом командной строки Docker в качестве примера использования элемента управления ресурсами (может быть настроено в оркестраторе или другом инструменте) и соответствующим API-интерфейсом службы контейнера узлов конкретной версии (HCS) Windows вычислительного узла Windows, а также описание реализации элемента управления ресурсами в Windows (обратите внимание, что это высокоуровневое описание и что базовая реализация может измениться).

|  | |
| ----- | ------|
| *Память* ||
| Интерфейс Docker | [--memory](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| Интерфейс HCS | [MemoryMaximumInMB]( https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684147(v=vs.85).aspx) |
| Изоляция Hyper-V | Память виртуальной машины |
| ||
| *ЦП (число)* ||
| Интерфейс Docker | [--cpus](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Интерфейс HCS | [ProcessorCount]( https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | Эмулируется с помощью [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx)* |
| Изоляция Hyper-V | Доступное число виртуальных процессоров |
| ||
| *ЦП (процент)* ||
| Интерфейс Docker | [--cpu-percent](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Интерфейс HCS | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx) |
| Изоляция Hyper-V | Ограничение виртуальных процессоров, применяемое низкоуровневой оболочкой |
| ||
| *ЦП (общие ресурсы)* ||
| Интерфейс Docker | [--cpu-shares](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Интерфейс HCS | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx) |
| Изоляция Hyper-V | Веса виртуальных процессоров низкоуровневой оболочки |
| ||
| *Хранилище (образ)* ||
| Интерфейс Docker | [--io-maxbandwidth/--io-maxiops]( https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| Интерфейс HCS | [StorageIOPSMaximum и StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |
| Изоляция Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |
| ||
| *Хранилище (тома)* ||
| Интерфейс Docker | [--storage-opt size=]( https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| Интерфейс HCS | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |
| Изоляция Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |

## <a name="additional-notes-or-details"></a>Дополнительные примечания и сведения
### <a name="memory"></a>Память
В контейнерах Windows выполняются некоторые системные процессоры. Обычно они предоставляют определенные возможности, такие как управление пользователями, сетями и т. д. При этом память, необходимая этим процессам, совместно используется контейнерами, поэтому ограничение памяти должно быть достаточно большим для их выполнения.  В документе с [требованиями к системе](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments) представлена таблица для каждого типа базового образа с изоляцией Hyper-V и без нее.

### <a name="cpu-shares-without-hyper-v-isolation"></a>Общие ресурсы ЦП (без изоляции Hyper-V)
При использовании общих ресурсов ЦП базовая реализация (без изоляции Hyper-V) настраивает [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx), в частности устанавливает для флага управления значение JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED и указывает соответствующий вес.  Допустимые значения весов объекта задания: 1–9. Значение по умолчанию— 5, при этом точность меньше, чем у значений служб контейнера узлов (1–10 000).  Например, вес общего ресурса, равный 7500, соответствует весу 7, а вес общего ресурса 2500 соответствует весу 2.
