---
title: "Требования к контейнеру Windows"
description: "Требования к контейнеру Windows."
keywords: "метаданные, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: ecc11468bbd5aad2638da3c4f733e4d5068f0056
ms.sourcegitcommit: 77a6195318732fa16e7d5be727bdb88f52f6db46
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/20/2017
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
<td><center>Сервер Nano Server*</center></td>
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
* Начиная с Windows Server версии 1709 Nano Server больше не доступен как узел контейнеров.

### <a name="memory-requirments"></a>Требования к памяти
Ограничения доступной контейнерам памяти можно настроить с помощью [элементов управления ресурсами](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) или путем перегрузки узла контейнера.  Минимальный объем памяти, необходимый для запуска контейнера и выполнении основных команд (ipconfig, dir и т. д.), указан ниже.  Обратите внимание, что эти значения не учитывают совместное использование ресурсов между контейнерами или требования приложения, которое выполняется в контейнере.

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

## <a name="matching-container-host-version-with-container-image-versions"></a>Сопоставление версии узла контейнера с версиями образа контейнера
### <a name="windows-server-containers"></a>Контейнеры Windows Server
Так как контейнеры Windows Server и базовые узлы имеют одно ядро, базовые образы контейнера и узла должны совпадать.  Если версии не совпадают, контейнер может запуститься, но работа всех возможностей не гарантируется. Операционная система Windows имеет четыре уровня указания версии: основной номер, дополнительный номер, номер сборки и номер редакции. Например: 10.0.14393.103. Номер сборки (т. е. 14393) меняется только при публикации новых версий операционной системы, например версии 1709, 1803, обновления Fall Creators Update и т. д. Номер редакции (т. е. 103) обновляется по мере установки обновлений Windows.
#### <a name="build-number-new-release-of-windows"></a>Номер сборки (новая версия Windows)
Контейнеры Windows Server не будут запускаться, если номера сборки узла контейнера и образа контейнера отличаются. Например: 10.0.14393.* (Windows Server 2016) и 10.0.16299.* (Windows Server версии 1709).  
#### <a name="revision-number-patching"></a>Номер редакции (исправления)
Контейнеры Windows Server _не_ блокируются, если номера редакции узла контейнера и образа контейнера отличаются. Например: 10.0.14393.1914.* (Windows Server 2016 с исправлением KB4051033) и 10.0.14393.1944.* (Windows Server 2016 с исправлением KB4053579).  
Для узлов и образов на основе Windows Server 2016: редакция образа контейнера должна соответствовать узлу с поддерживаемой конфигурацией.  Начиная с Windows Server версии 1709 это требование больше не применяется: редакции образов узла контейнера не обязаны совпадать.  Как всегда, рекомендуется устанавливать последние исправления и обновления системы.
#### <a name="practical-application"></a>Практическое применение
Пример 1. Узел контейнера работает под управлением Windows Server 2016 с исправлением KB4041691.  Все контейнеры Windows Server, развернутые на этом узле, должны быть основаны на образах контейнера 10.0.14393.1770.  Если обновление KB4053579 применено к узлу, образы контейнеры необходимо обновить одновременно для сохранения поддержки.
Пример 2. Узел контейнера работает под управлением Windows Server версии 1709 с исправлением KB4043961.  Все контейнеры Windows Server, развернутые на этом узле, должны быть основано на базовом образе контейнера Windows Server версии 1709 (10.0.16299) , но не обязаны соответствовать KB узла.  Если к узлу применено обновление KB4054517, образы контейнеров необязательно обновлять, хотя это рекомендуется для обеспечения безопасности.
#### <a name="querying-version"></a>Запрос версии
Метод 1. В версии 1709 появились командная строка и команда ver для получения сведений о версиях.
```
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125] 
```
Метод 2. Запросите следующий раздел реестра: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion. Например:
```
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```
или
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Чтобы узнать версию используемого базового образа, просмотрите теги в Центре Docker или хэш-таблицу образа в описании образа.  Информацию о датах выпусках всех сборок и редакций можно найти на странице [журнала обновлений Windows10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history).

### <a name="hyper-v-isolation-for-containers"></a>Изоляция Hyper-V для контейнеров
Контейнеры Windows можно запускать изоляцией Hyper-V и без нее.  Изоляция Hyper-V создает границу безопасности вокруг контейнере с оптимизированной ВМ.  В отличие от стандартных контейнеров Windows, где для контейнеров и узла используется одно ядро, каждый изолированный контейнер Hyper-V использует собственный экземпляр ядра Windows.  Из-за этого в узле контейнера и образе могут быть разные версии ОС (см. матрицу совместимости ниже).  

Для запуска контейнера с изоляцией Hyper-V просто добавьте тег «--isolation=hyper-v» в вашу команду docker run.

### <a name="compatibility-matrix"></a>Матрица совместимости
Сборки с номером больше общедоступной версии Windows Server2016 (10.0.14393.206) могут запускать образы общедоступной версии Windows Server2016 в Windows Server Core или Nano Server в поддерживаемой конфигурации, независимо от номера редакции.
На узле Windows Server версии 1709 также можно запускать контейнеры на основе Windows Server 2016, но не наоборот.

Важно понимать, что для обеспечения полной функциональности, надежности и гарантий безопасности, предоставляемых обновлениями Windows, следует использовать последние версии на всех системах.  
