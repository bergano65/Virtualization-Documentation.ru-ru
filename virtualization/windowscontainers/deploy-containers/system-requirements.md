---
title: Требования к контейнеру Windows
description: Требования к контейнеру Windows.
keywords: метаданные, контейнеры
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 942676be30760cbe1701d75f7d2fbca9539ce03b
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380228"
---
# <a name="windows-container-requirements"></a>Требования к контейнеру Windows

В этом руководстве приведены требования для узла контейнера Windows.

## <a name="os-requirements"></a>Требования к ОС

- Функция контейнера Windows доступна только в Windows Server 2016 (основной и с возможностями рабочего стола), Windows 10 Профессиональная и Корпоративная (Anniversary Edition) и более поздних версиях.
- Перед запуском изоляции Hyper-V необходимо установить роль Hyper-V
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\. Это ограничение применяется, если только контейнеры Hyper-V изолированными будут развернуты.

## <a name="virtualized-container-hosts"></a>Виртуализированные узлы контейнера

Если узел контейнера Windows будут выполняться на виртуальной машине Hyper-V, а также будет размещен изоляции Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- Windows Server 2019 Windows Server версии 1803, Windows Server версии 1709, Windows Server 2016 или Windows 10 на главном компьютере и Windows Server (полная Core) на виртуальной машине.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
- Для виртуальной Машины узла контейнера также понадобится по меньшей мере два виртуальных процессоров.

## <a name="supported-base-images"></a>Поддерживаемые базовые образы

Контейнеры Windows предоставляются в четыре базовых образов контейнеров: Windows Server Core, Nano Server, Windows и IoT базовая. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Гостевая операционная система</center></th>
<th><center>Контейнер Windows Server</center></th>
<th><center>Изоляция Hyper-V</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 / 2019 г. (Standard или Datacenter)</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Сервер Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Сервер Nano Server</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows10 Pro или Корпоративная</center></td>
<td><center>Недоступно</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Базовая</center></td>
<td><center>IoT Базовая</center></td>
<td><center>Недоступно</center></td>
</tr>
</tbody>
</table>

> [!WARNING]  
> Начиная с Windows Server версии 1709 Nano Server больше не доступен как узел контейнера.

### <a name="memory-requirements"></a>Требования к памяти

Ограничения доступной контейнерам памяти можно настроить с помощью [элементов управления ресурсами](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) или путем перегрузки узла контейнера.  Минимальный объем памяти, необходимый для запуска контейнера и выполнении основных команд (ipconfig, dir и т. д.), приведены ниже.

>[!NOTE]
>Эти значения не учитывают совместное использование ресурсов между контейнерами или требования приложения, которое выполняется в контейнере.  Например, на узле с 512 МБ свободной памяти можно запустить несколько контейнеров основных серверных компонентов в режиме изоляции Hyper-V, поскольку эти контейнеры совместно используют ресурсы.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Базовый образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 40 МБ                     | Файл подкачки 130 МБ + 1 ГБ |
| Основные серверные компоненты | 50 МБ                     | Файл подкачки 325 МБ + 1 ГБ |

#### <a name="windows-server-version-1709"></a>Windows Server версии 1709

| Базовый образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 30 МБ                     | Файл подкачки 110 МБ + 1 ГБ |
| Основные серверные компоненты | 45 МБ                     | Файл подкачки 360 МБ + 1 ГБ |

### <a name="base-image-differences"></a>Отличия базового образа

Как выбрать вправо базового образа на основе? В то время как вы можете для сборки все, что требуется, ниже перечислены общие рекомендации для каждого изображения:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Если приложению полной платформе .NET framework, это изображение лучше использовать.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): для приложений, требующих только .NET Core Nano Server обеспечит гораздо тонкими образа.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): вы можете обнаружить ваше приложение зависит от компонент или DLL, который отсутствует в Server Core или образов Nano Server, таких как библиотеки GDI. Этот образ выполняет зависимостей полный набор Windows.
- [IoT базовая](https://hub.docker.com/_/microsoft-windows-iotcore): это изображение основная файловая система для [приложения для Интернета вещей](https://developer.microsoft.com/en-us/windows/iot). Следует использовать этот образ контейнера при нацеливании на узле IoT базовая.

Для большинства пользователей Windows Server Core или Nano Server будет наиболее подходящего изображения для использования. Ниже перечислены моменты, помните о разработке поверх Nano Server.

- стек обслуживания был удален;
- компонент .NET Core не включен (хотя вы можете использовать [образ .NET Core Nano Server](https://hub.docker.com/r/microsoft/dotnet/));
- компонент PowerShell был удален;
- компонент WMI был удален.
- Начиная с Windows Server версии 1709, приложения запускаются в контексте пользователя, поэтому команды, требующие привилегий администратора, выполняться не будут. Вы можете указать учетную запись администратора контейнера через флаг пользователя (например, "run" docker — "ContainerAdministrator пользователя") тем не менее в будущем планируется полное удаление учетных записей администратора из NanoServer.

Это самые существенные различия, но не полный список. Существуют другие компоненты, которые также отсутствуют. Имейте в виду, что вы всегда можете добавить другие компоненты поверх Nano Server по своему усмотрению. Пример см. здесь: [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).