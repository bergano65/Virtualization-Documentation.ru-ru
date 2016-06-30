---
title: "Требования к контейнеру Windows"
description: "Требования к контейнеру Windows."
keywords: metadata, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: cc216f56acd5e547d05a48beea57450ba5fcb28b
ms.openlocfilehash: 12ae565f012dc87a2cab883c0486322db42b1dcc

---

# Требования к контейнеру Windows

**Это предварительное содержимое. Возможны изменения.** 

В этих руководствах перечислены требования для узла контейнера Windows.

## Требования к ОС

- Роль контейнера Windows доступна только в Windows Server 2016 TP5 (полной версии и версии Core), Nano Server и Windows 10 (начиная со сборки 14352 для участников программы предварительной оценки).
- При использовании контейнеров Hyper-V должна быть установлена роль Hyper-V.
- На узлах контейнеров Windows Server операционная система Windows должна устанавливаться в каталог C:\\. Если развертываются только контейнеры Hyper-V, это ограничение не применяется.

## Виртуализированные узлы контейнера

Если вы хотите разместить контейнеры Hyper-V на узле контейнера, работающем на виртуальной машине Hyper-V, и запускать контейнер Windows c виртуальной машины Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- ОС Windows Server 2016 Technical Preview 5 или Windows 10 сборки 10565 на главном компьютере и Windows Server Technical Preview 5 (полная версия или версия Core) или Nano Server на виртуальной машине.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
- Для виртуальной машины узла контейнера также понадобится по меньшей мере 2 виртуальных процессора.

## Поддерживаемые образы ОС

Windows Server Technical Preview 5 предлагается с двумя образами ОС контейнера: Windows Server Core и Nano Server. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.

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
<td><center>Полный пользовательский интерфейс Windows Server 2016</center></td>
<td><center>Образ Server Core</center></td>
<td><center>Образ Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Образ Server Core</center></td>
<td><center> Образ Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Образ Nano Server</center></td>
<td><center>Образ Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Выпуски Windows 10 для предварительной оценки</center></td>
<td><center>Недоступно</center></td>
<td><center>Образ Nano Server</center></td>
</tr>
</tbody>
</table>



<!--HONumber=Jun16_HO4-->


