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
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910504"
---
# <a name="windows-container-requirements"></a>Требования к контейнеру Windows

В этом руководством перечислены требования для узла контейнера Windows.

## <a name="operating-system-requirements"></a>Требования к операционной системе

- Функция контейнера Windows доступна в выпусках Windows Server (полугодовой канал), Windows Server 2019, Windows Server 2016 и Windows 10 профессиональная и Enterprise (версия 1607 и более поздние версии).
- Перед выполнением изоляции Hyper-V необходимо установить роль Hyper-V
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\. Это ограничение не применяется, если будут развернуты только изолированные контейнеры Hyper-V.

## <a name="virtualized-container-hosts"></a>Узлы виртуализированного контейнера

Если узел контейнера Windows будет запускаться с виртуальной машины Hyper-V, а также будет размещена изоляция Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- Windows Server (полугодовой канал), Windows Server 2019, Windows Server 2016 или Windows 10 в главной системе; и Windows Server (Full или Server Core) на виртуальной машине.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
- Для виртуальной машины узла контейнера также потребуется по крайней мере два виртуальных процессора.

### <a name="memory-requirements"></a>Требования к памяти

Ограничения доступной контейнерам памяти можно настроить с помощью [элементов управления ресурсами](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) или путем перегрузки узла контейнера.  Минимальный объем памяти, необходимый для запуска контейнера и выполнения основных команд (ipconfig, dir и т. д.), приведен ниже.

>[!NOTE]
>Эти значения не принимают в качестве учетных данных общий доступ к ресурсам между контейнерами или требованиями из приложения, выполняемого в контейнере.  Например, на узле с 512 МБ свободной памяти можно запустить несколько контейнеров основных серверных компонентов в режиме изоляции Hyper-V, поскольку эти контейнеры совместно используют ресурсы.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 40 МБ                     | 130 Мб + файл подкачки размером 1 ГБ |
| Основные серверные компоненты | 50 МБ                     | 325 МБ + файл подкачки размером 1 ГБ |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (Semi-Annual Channel)

| Base image  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 30 МБ                     | 110 Мб + файл подкачки размером 1 ГБ |
| Основные серверные компоненты | 45 МБ                     | 360 Мб + файл подкачки размером 1 ГБ |

## <a name="see-also"></a>См. также

[Политика поддержки для контейнеров Windows и DOCKER в локальных сценариях](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)