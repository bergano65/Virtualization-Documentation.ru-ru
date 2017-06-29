---
title: "Требования к контейнеру Windows"
description: "Требования к контейнеру Windows."
keywords: "метаданные, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: c6f8dd68c6c9f346e26d29b0072a0e3d8c18759f
ms.sourcegitcommit: 8f096e9d557c60985c8512b43e4e4058cb307ed2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/09/2017
---
# <a name="windows-container-requirements"></a>Требования к контейнеру Windows

В этих руководствах перечислены требования для узла контейнера Windows.

## <a name="os-requirements"></a>Требования к ОС

- Функция контейнера Windows доступна только в Windows Server2016 (варианты "Установка основных серверных компонентов" и "С возможностями рабочего стола"), Nano Server, Windows10 Professional и Windows10 Корпоративная (Anniversary Edition).
- Перед запуском контейнеров Hyper-V необходимо установить роль Hyper-V.
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\. Если развертываются только контейнеры Hyper-V, это ограничение не применяется.

## <a name="virtualized-container-hosts"></a>Виртуализированные узлы контейнера

Если вы хотите разместить контейнеры Hyper-V на узле контейнера, работающем на виртуальной машине Hyper-V, и запускать контейнер Windows c виртуальной машины Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- ОС Windows Server2016 или Windows10 на главном компьютере и Windows Server (полная версия или версия Core) или Nano Server на виртуальной машине.
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
<td><center>Сервер Nano Server</center></td>
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

## <a name="matching-container-host-version-with-container-image-versions"></a>Сопоставление версии узла контейнера с версиями образа контейнера
### <a name="windows-server-containers"></a>Контейнеры Windows Server
Так как контейнеры Windows Server и базовые узлы имеют одно ядро, базовые образы контейнера и узла должны совпадать.  Если версии не совпадают, контейнер может запуститься, но работа всех возможностей не гарантируется. Поэтому несовпадающие версии не поддерживаются.  Операционная система Windows имеет четыре уровня указания версии: основной номер, дополнительный номер, номер сборки и номер редакции. Например, 10.0.14393.0. Номер сборки меняется только при публикации новой версии операционной системы. Номер редакции меняется при установке обновлений Windows. Запуск контейнеров Windows Server блокируется, если номера сборки отличаются. Например, 10.0.14300.1030 (Technical Preview5) и 10.0.14393 (Windows Server2016RTM). Если номера сборки совпадают, а номера редакции отличаются, запуск не блокируется. Например, 10.0.14393 (Windows Server2016RTM) и 10.0.14393.206 (общедоступная версия Windows Server2016). Несмотря на то что с технической точки зрения запуск контейнеров не блокируется, такая конфигурация не сможет правильно работать во всех возможных условиях, и поэтому она не поддерживается для рабочих сред. 

Чтобы проверить, какая версия узла Windows установлена, выполните запрос к HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion.  Чтобы узнать версию используемого базового образа, просмотрите теги в Центре Docker или хэш-таблицу образа в описании образа.  Информацию о датах выпусках всех сборок и редакций можно найти на странице [журнала обновлений Windows10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history).

В этом примере 14393 является основным номером сборки, и 321— номером редакции.
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### <a name="hyper-v-containers"></a>Контейнеры Hyper-V
В отличие от контейнеров Windows Server, где для контейнеров и узла используется одно ядро, каждый контейнер Hyper-V использует собственный экземпляр ядра Windows.  Из-за этого пользователи могут неправильно сопоставить версии узла контейнера и образа контейнера.  Сейчас сборки с номером, равным или больше общедоступной версии Windows Server2016 (10.0.14393.206), могут запускать образы общедоступной версии Windows Server2016 в Windows Server Core или Nano Server в поддерживаемой конфигурации, независимо от номера редакции.  В будущем на основе отзывов клиентов мы предоставим конкретные рекомендации относительно того, настолько номера сборок могут отличаться друг от друга.  Важно понимать, что для обеспечения полной функциональности, надежности и гарантий безопасности, предоставляемых обновлениями Windows, следует использовать последние версии на всех системах.  
