---
title: Требования к контейнеру Windows
description: Требования к контейнеру Windows.
keywords: метаданные, контейнеры
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d3df0631a8a61db16ad207f49163a7304c5db717
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681054"
---
# <a name="windows-container-requirements"></a>Требования к контейнеру Windows

В этом руководстве перечислены требования для узла контейнера Windows.

## <a name="os-requirements"></a>Требования к ОС

- Контейнер Windows доступен только в Windows Server 2016 (ядро и с возможностями рабочего стола), Windows 10 профессиональная и Enterprise (годовщина выпуска) и более поздних версий.
<<<<<<< HEAD
- Перед выполнением изоляции Hyper-V необходимо установить роль Hyper-V.
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\. Это ограничение не применяется, если развертываются только изолированные контейнеры Hyper-V.
=======
- Роль Hyper-V должна быть установлена перед выполнением контейнеров с изоляцией Hyper-V.
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\. Если развертываются только контейнеры Hyper-V, это ограничение не применяется.
>>>>>>> Источник и образец

## <a name="virtualized-container-hosts"></a>Виртуализированные узлы контейнера

_Лт__лт__лт__лт__лт__лт__лт_ HEAD если узел контейнера Windows будет запущен с помощью виртуальной машины Hyper-V, а также будет размещена изоляция Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация имеет следующие требования: = = = = = = =, если узел контейнера Windows будет запускаться с помощью виртуальной машины Hyper-V, а также будет размещать контейнеры с изоляцией Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.
>>>>>>> Источник и образец

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- Windows Server 2019, Windows Server версии 1803, Windows Server версии 1709, Windows Server 2016 или Windows 10 в системе размещения и Windows Server (Full, Core) на виртуальной машине.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
<<<<<<< HEAD
- Виртуальная машина контейнера также должна иметь по крайней мере два виртуальных процессора.

## <a name="supported-base-images"></a>Поддерживаемые базовые образы

<a name="windows-containers-are-offered-with-four-container-base-images-windows-server-core-nano-server-windows-and-iot-core-not-all-configurations-support-both-os-images-this-table-details-the-supported-configurations"></a>Контейнеры Windows предлагаются четырьмя базовыми изображениями контейнера: Windows Server Core, Nano Server, Windows и IoT основы. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.
=======
- Для виртуальной машины узла контейнера также понадобится по меньшей мере 2виртуальных процессора.

## <a name="supported-base-images"></a>Поддерживаемые базовые образы

Контейнеры Windows предлагаются четырьмя базовыми изображениями контейнера: Windows Server Core, Nano Server, Windows и IoT основы. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.

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
<td><center>Windows Server 2016/2019 (Standard или Datacenter)</center></td>
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
<td><center>Windows<a href="#warn-2">**</a></center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Базовая</center></td>
<td><center>IoT Базовая</center></td>
<td><center>Недоступно</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">* Начиная с Windows Server, версия 1709 Nano Server больше не доступна в качестве узла контейнера.</span>

> <span id="warn-2">* * Требуется Windows 10 октября Update 2018, и вы напрямую запрашиваете изоляцию процессов с помощью `--isolation=process` флага при запуске контейнеров через `docker run`.</span>
>>>>>>> Источник и образец

|Операционная система узла|Контейнер Windows|Изоляция Hyper-V|
|---------------------|-----------------|-----------------|
|Windows Server 2016 или Windows Server 2019 (Standard или Datacenter)|Server Core, Nano Server, Windows|Server Core, Nano Server, Windows|
|Сервер Nano Server|Сервер Nano Server|Server Core, Nano Server, Windows|
|Windows 10 Pro или Windows 10 корпоративный|Отсутствует|Server Core, Nano Server, Windows|
|IoT Базовая|IoT Базовая|Отсутствует|

> [!WARNING]  
> Начиная с Windows Server версии 1709, Nano Server больше не доступен в качестве узла контейнера.

### <a name="memory-requirements"></a>Требования к памяти

Ограничения доступной контейнерам памяти можно настроить с помощью [элементов управления ресурсами](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) или путем перегрузки узла контейнера.  Минимальный объем памяти, необходимый для запуска контейнера и выполнения основных команд (ipconfig, dir и т. д.), приведен ниже.

>[!NOTE]
>Эти значения не зависят от того, какие данные выделяются между контейнерами или требованиями из приложения, которое выполняется в контейнере.  Например, на узле с 512 МБ свободной памяти можно запустить несколько контейнеров основных серверных компонентов в режиме изоляции Hyper-V, поскольку эти контейнеры совместно используют ресурсы.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Основной образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 40 МБ                     | 130 МБ + 1 ГБ (файл подкачки) |
| Основные серверные компоненты | 50 МБ                     | 325 МБ + 1 ГБ (файл подкачки) |

#### <a name="windows-server-version-1709"></a>Windows Server версии 1709

| Основной образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 30 МБАЙТ                     | 110 Мб + 1 ГБ (файл подкачки) |
| Основные серверные компоненты | 45 МБ                     | 360 МБ + 1 ГБ (файл подкачки) |

### <a name="base-image-differences"></a>Отличия базовых изображений

Как можно выбрать подходящего базового образа для сборки? Несмотря на то, что вы можете создавать любые из них, Вот общие рекомендации по каждому изображению:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Если для вашего приложения требуется полная версия .NET Framework, лучше использовать это изображение.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): для приложений, которым требуется только платформа .NET Core, Nano Server предоставит очень облегченнаяное изображение.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): вы можете обнаружить, что приложение зависит от компонента или библиотеки DLL, отсутствующей в серверных ядрах или в изображениях Nano Server, таких как библиотеки GDI. Это изображение содержит полный набор зависимостей Windows.
- [Ядро IOT](https://hub.docker.com/_/microsoft-windows-iotcore): это изображение предназначено для [приложений IOT](https://developer.microsoft.com/windows/iot). Этот контейнер следует использовать при нацеливании на основной узел IoT.

Для большинства пользователей наиболее подходящим будет использование Windows Server Core или Nano Server. Ниже перечислены некоторые моменты, которые необходимо учитывать при создании сервера Nano Server.

- стек обслуживания был удален;
- компонент .NET Core не включен (хотя вы можете использовать [образ .NET Core Nano Server](https://hub.docker.com/r/microsoft/dotnet/));
- компонент PowerShell был удален;
- компонент WMI был удален.
- Начиная с Windows Server версии 1709, приложения запускаются в контексте пользователя, поэтому команды, требующие привилегий администратора, выполняться не будут. Вы можете указать учетную запись администратора контейнера с помощью флага пользователя (например, Контаинерадминистратор), но в будущем мы планируем полностью удалить учетные записи администратора с сервера.

Это самые существенные различия, но не полный список. Существуют другие компоненты, которые также отсутствуют. Имейте в виду, что вы всегда можете добавить другие компоненты поверх Nano Server по своему усмотрению. Пример см. здесь: [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).