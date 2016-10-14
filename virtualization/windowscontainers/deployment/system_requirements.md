---
title: "Требования к контейнеру Windows"
description: "Требования к контейнеру Windows."
keywords: "метаданные, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: 08355d7a7da50d0f244bd750508fd42084818d7f
ms.openlocfilehash: ac48bc7ee7b70483d8a368749aea0862c52f049c

---

# Требования к контейнеру Windows

В этих руководствах перечислены требования для узла контейнера Windows.

## Требования к ОС

- Функция контейнера Windows доступна только в Windows Server 2016 (варианты "Установка основных серверных компонентов" и "С возможностями рабочего стола"), Nano Server, Windows 10 Professional и Windows 10 Корпоративная (Anniversary Edition).
- При использовании контейнеров Hyper-V должна быть установлена роль Hyper-V.
- На узлах контейнера Windows Server ОС Windows должна быть установлена на диске c:\. Если развертываются только контейнеры Hyper-V, это ограничение не применяется.

## Виртуализированные узлы контейнера

Если вы хотите разместить контейнеры Hyper-V на узле контейнера, работающем на виртуальной машине Hyper-V, и запускать контейнер Windows c виртуальной машины Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- ОС Windows Server 2016 или Windows 10 на главном компьютере и Windows Server (полная версия или версия Core) или Nano Server на виртуальной машине.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
- Для виртуальной машины узла контейнера также понадобится по меньшей мере 2 виртуальных процессора.

## Поддерживаемые базовые образы

Контейнеры Windows предоставляются в двух базовых образах — Windows Server Core и Nano Server. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.

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
<td><center>Windows Server 2016 в настольной версии</center></td>
<td><center>Server Core или Nano Server</center></td>
<td><center>Server Core или Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core или Nano Server</center></td>
<td><center>Server Core или Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Сервер Nano Server</center></td>
<td><center> Сервер Nano Server</center></td>
<td><center>Server Core или Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro или Корпоративная</center></td>
<td><center>Недоступно</center></td>
<td><center>Server Core или Nano Server</center></td>
</tr>
</tbody>
</table>


<!--HONumber=Oct16_HO2-->


