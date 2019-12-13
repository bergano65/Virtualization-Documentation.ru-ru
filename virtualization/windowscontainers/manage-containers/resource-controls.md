---
title: Реализация элементов управления ресурсами
description: Сведения об элементах управления ресурсами для контейнеров Windows
keywords: docker, контейнеры, цп, память, диск, ресурсы
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: 3e9f7e3208222cd6c0f512c5f892453ac6e6980c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910174"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>Реализация элементов управления ресурсами для контейнеров Windows
Существует несколько элементов управления ресурсами, которые можно реализовать на уровне контейнера и ресурса.  По умолчанию управление контейнерами осуществляется как типичными ресурсами Windows, которое обычно основано на справедливом распределении, однако с помощью конфигурации этих элементов управления разработчик или администратор может ограничить использование ресурсов.  Можно управлять следующими ресурсами: ЦП или процессор, память или ОЗУ, диск или хранилище, сеть или пропускная способность.

Контейнеры Windows используют [объекты заданий](https://docs.microsoft.com/windows/desktop/ProcThread/job-objects) для группировки и отслеживания процессов, связанных с каждым контейнером.  Элементы управления ресурсами реализуются в родительском объекте задания, связанном с контейнером. 

При использовании [изоляции Hyper-V](./hyperv-container.md) элементы управления ресурсами автоматически применяются как к виртуальной машине, так и к объекту задания контейнера, который выполняется на виртуальной машине. Даже если процесс, выполняемый в контейнере, обойдет элементы управления объектами заданий, виртуальная машина все равно обеспечит применение заданных элементов управления ресурсами.

## <a name="resources"></a>Ресурсы
Для каждого ресурса в этом разделе представлено сопоставление между интерфейсом командной строки Docker в качестве примера использования элемента управления ресурсами (может быть настроено в оркестраторе или другом инструменте) и соответствующим API-интерфейсом службы контейнера узлов конкретной версии (HCS) Windows вычислительного узла Windows, а также описание реализации элемента управления ресурсами в Windows (обратите внимание, что это высокоуровневое описание и что базовая реализация может измениться).

|  | |
| ----- | ------|
| *Память* ||
| Интерфейс Docker | [--память](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| Интерфейс HCS | [меморимаксимуминмб](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_basic_limit_information) |
| Изоляция Hyper-V | Память виртуальной машины |
| _Примечание о изоляции Hyper-V в Windows Server 2016. при использовании ограничения памяти вы увидите, что контейнер сначала выделит объем памяти, а затем начнет возвратить его обратно в узел контейнера.  В более поздних версиях (1709 или более) это было оптимизировано._ |
| ||
| *ЦП (количество)* ||
| Интерфейс Docker | [--ЦП](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Интерфейс HCS | [ProcessorCount](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | Эмулируется с помощью [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)* |
| Изоляция Hyper-V | Доступное число виртуальных процессоров |
| ||
| *ЦП (в процентах)* ||
| Интерфейс Docker | [--ЦП-проценты](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Интерфейс HCS | [процессормаксимум](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Изоляция Hyper-V | Ограничение виртуальных процессоров, применяемое низкоуровневой оболочкой |
| ||
| *ЦП (общие ресурсы)* ||
| Интерфейс Docker | [--cpu-shares](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Интерфейс HCS | [процессорвеигхт](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Изоляция Hyper-V | Веса виртуальных процессоров низкоуровневой оболочки |
| ||
| *Хранилище (изображение)* ||
| Интерфейс Docker | [--IO-MaxBandwidth/--IO-максиопс](https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| Интерфейс HCS | [Сторажеиопсмаксимум и Сторажебандвидсмаксимум](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Изоляция Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| ||
| *Хранилище (тома)* ||
| Интерфейс Docker | [--Storage-неявное изменение размера =](https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| Интерфейс HCS | [сторажесандбокссизе](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Общее ядро | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Изоляция Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

## <a name="additional-notes-or-details"></a>Дополнительные примечания и сведения

### <a name="memory"></a>Память

В контейнерах Windows выполняются некоторые системные процессоры. Обычно они предоставляют определенные возможности, такие как управление пользователями, сетями и т. д. При этом память, необходимая этим процессам, совместно используется контейнерами, поэтому ограничение памяти должно быть достаточно большим для их выполнения.  В документе с [требованиями к системе](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments) представлена таблица для каждого типа базового образа с изоляцией Hyper-V и без нее.

### <a name="cpu-shares-without-hyper-v-isolation"></a>Общие ресурсы ЦП (без изоляции Hyper-V)

При использовании общих ресурсов ЦП базовая реализация (без изоляции Hyper-V) настраивает [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information), в частности устанавливает для флага управления значение JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED и указывает соответствующий вес.  Допустимые значения весов объекта задания: 1–9. Значение по умолчанию — 5, при этом точность меньше, чем у значений служб контейнера узлов (1–10 000).  Например, вес общего ресурса, равный 7500, соответствует весу 7, а вес общего ресурса 2500 соответствует весу 2.
