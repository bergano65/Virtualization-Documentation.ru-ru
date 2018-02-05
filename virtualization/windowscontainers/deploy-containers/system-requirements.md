---
title: "Требования к контейнеру Windows"
description: "Требования к контейнеру Windows."
keywords: "метаданные, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 88d094202c49cf725e9d608a0810e7d9f8a1e271
ms.sourcegitcommit: 7fc79235cbee052e07366b8a6aa7e035a5e3434f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/13/2018
---
# <a name="windows-container-requirements"></a>Требования к контейнеру Windows

В этих руководствах перечислены требования для узла контейнера Windows.

## <a name="os-requirements"></a>Требования к ОС

- Функция контейнера Windows доступна только в Windows Server сборки 1709, Windows Server 2016 (варианты "Установка основных серверных компонентов" и "С возможностями рабочего стола"), Windows10 Professional и Windows10 Корпоративная (Anniversary Edition).
- Перед запуском контейнеров Hyper-V необходимо установить роль Hyper-V.
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\. Если развертываются только контейнеры Hyper-V, это ограничение не применяется.

## <a name="virtualized-container-hosts"></a>Виртуализированные узлы контейнера

Если вы хотите разместить контейнеры Hyper-V на узле контейнера, работающем на виртуальной машине Hyper-V, и запускать контейнер Windows c виртуальной машины Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- Windows Serverсборки 1709, Windows Server2016 или Windows10 на главном компьютере и Windows Server (полная версия или версия Core) на виртуальной машине.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
- Для виртуальной машины узла контейнера также понадобится по меньшей мере 2виртуальных процессора.

## <a name="supported-base-images"></a>Поддерживаемые базовые образы

Контейнеры Windows предоставляются в двух базовых образах— Windows Server Core и Nano Server. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Гостевая операционная система</center></th>
<th><center>Контейнер Windows Server</center></th>
<th><center>Контейнер Hyper-V</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 (Standard или Datacenter)</center></td>
<td><center>Server Core или Nano Server</center></td>
<td><center>Server Core или Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Сервер Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Сервер Nano Server</center></td>
<td><center>Server Core или Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows10 Pro или Корпоративная</center></td>
<td><center>Недоступно</center></td>
<td><center>Server Core или Nano Server</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">Начиная с Windows Server версии 1709 Nano Server больше не доступен как узел контейнеров.</span>


### <a name="memory-requirments"></a>Требования к памяти
Ограничения доступной контейнерам памяти можно настроить с помощью [элементов управления ресурсами](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) или путем перегрузки узла контейнера.  Минимальный объем памяти, необходимый для запуска контейнера и выполнении основных команд (ipconfig, dir и т. д.), указан ниже.  __Обратите внимание, что эти значения не учитывают совместное использование ресурсов между контейнерами или требования приложения, которое выполняется в контейнере.  Например, на узле с 512 МБ свободной памяти можно запустить несколько контейнеров основных серверных компонентов в режиме изоляции Hyper-V, поскольку эти контейнеры совместно используют ресурсы.__

#### <a name="windows-server-2016"></a>Windows Server 2016
| Базовый образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 40МБ                     | 130МБ + файл подкачки 1ГБ |
| Основные серверные компоненты | 50 МБ                     | 325МБ + файл подкачки 1ГБ |

#### <a name="windows-server-version-1709"></a>Windows Server версии 1709
| Базовый образ  | Контейнер Windows Server | Изоляция Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Сервер Nano Server | 30 МБ                     | 110МБ + файл подкачки 1ГБ |
| Основные серверные компоненты | 45 МБ                     | 360МБ + файл подкачки 1ГБ |


### <a name="nano-server-vs-windows-server-core"></a>Nano Server или Windows Server Core

Как выбрать между Windows Server Core и Nano Server? Вы можете выбрать любой компонент для сборки, но если вашему приложению необходима полная совместимость с .NET Framework, вам следует использовать [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/). С другой стороне, если ваше приложение создано для облака и использует .NET Core, необходимо использовать [Nano Server](https://hub.docker.com/r/microsoft/nanoserver/). Это связано с тем, что компонент Nano Server был создан для обеспечения максимальной компактности, поэтому несколько ненужных библиотек были удалены. Помните о следующем при разработке с использованием Nano Server:

- стек обслуживания был удален;
- компонент .NET Core не включен (хотя вы можете использовать [образ .NET Core Nano Server](https://hub.docker.com/r/microsoft/dotnet/));
- компонент PowerShell был удален;
- компонент WMI был удален.

Это самые существенные различия, но не полный список. Существуют другие компоненты, которые также отсутствуют. Имейте в виду, что вы всегда можете добавить другие компоненты поверх Nano Server по своему усмотрению. Пример см. здесь: [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile).

