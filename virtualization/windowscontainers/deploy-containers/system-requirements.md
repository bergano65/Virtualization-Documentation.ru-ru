---
title: Требования к контейнеру Windows
description: Требования к контейнеру Windows.
keywords: метаданные, контейнеры
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254264"
---
# <a name="windows-container-requirements"></a>Требования к контейнеру Windows

В этом руководстве перечислены требования для узла контейнера Windows.

## <a name="operating-system-requirements"></a>Требования к операционной системе

- Контейнер Windows доступен в выпусках Windows Server (полуфабрикаты канала), Windows Server 2019, Windows Server 2016 и Windows 10 профессиональная и Enterprise Edition (версия 1607 и более поздние версии).
- Перед выполнением изоляции Hyper-V необходимо установить роль Hyper-V.
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\. Это ограничение не применяется, если развертываются только изолированные контейнеры Hyper-V.

## <a name="virtualized-container-hosts"></a>Виртуализированные узлы контейнера

Если узел контейнера Windows будет запущен с виртуальной машины Hyper-V, а также будет размещена изоляция Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- Windows Server (одновременный канал), Windows Server 2019, Windows Server 2016 или Windows 10 в основной системе; и Windows Server (Full или Server Core) на виртуальной машине.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
- Виртуальная машина контейнера также должна иметь по крайней мере два виртуальных процессора.

### <a name="memory-requirements"></a>Требования к памяти

Ограничения доступной контейнерам памяти можно настроить с помощью [элементов управления ресурсами](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) или путем перегрузки узла контейнера.  Минимальный объем памяти, необходимый для запуска контейнера и выполнения основных команд (ipconfig, dir и т. д.), приведен ниже.

>[!NOTE]
>Эти значения не зависят от того, какие данные выделяются между контейнерами или требованиями из приложения, которое выполняется в контейнере.  Например, на узле с 512 МБ свободной памяти можно запустить несколько контейнеров основных серверных компонентов в режиме изоляции Hyper-V, поскольку эти контейнеры совместно используют ресурсы.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Основной образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 40 МБ                     | 130 МБ + 1 ГБ (файл подкачки) |
| Основные серверные компоненты | 50 МБ                     | 325 МБ + 1 ГБ (файл подкачки) |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (Semi-Annual Channel)

| Основной образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 30 МБАЙТ                     | 110 Мб + 1 ГБ (файл подкачки) |
| Основные серверные компоненты | 45 МБ                     | 360 МБ + 1 ГБ (файл подкачки) |

## <a name="see-also"></a>См. также

[Политика поддержки для контейнеров и стыковочных окон в локальных сценариях](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)