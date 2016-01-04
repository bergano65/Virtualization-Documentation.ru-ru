# Требования к контейнеру Windows

**Это предварительное содержимое. Возможны изменения.**

В этом руководстве содержится список требований для узла контейнера Windows.

## Поддерживаемые образы ОС

Windows Server Technical Preview 4 предлагается с двумя образами ОС контейнера: Windows Server Core и Nano Server. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td><center>**Гостевая операционная система**</center></td>
<td><center>**Контейнер Windows Server**</center></td>
<td><center>**Контейнер Hyper-V**</center></td>
</tr>
<tr valign="top">
<td><center>Полный пользовательский интерфейс Windows Server 2016</center></td>
<td><center>Образ операционной системы Core</center></td>
<td><center>Образ операционной системы Nano</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Образ операционной системы Core</center></td>
<td><center> Образ операционной системы Nano</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Образ операционной системы Nano</center></td>
<td><center>Образ операционной системы Nano</center></td>
</tr>
</table>

## Требования к контейнеру Hyper-V

Если вы хотите разместить контейнеры Hyper-V на узле контейнера, работающем на виртуальной машине Hyper-V, и запускать контейнер Windows c виртуальной машины Hyper-V, необходимо включить вложенную виртуализацию. Вложенная виртуализация должна соответствовать следующим требованиям.

- Не менее 4 ГБ ОЗУ для виртуализированного узла Hyper-V.
- Windows Server 2016 Technical Preview 4 или Windows 10 build 10565 и на физическом и на виртуализированном узле.
- Процессор с Intel VT-x (в настоящий момент эта функция доступна только для процессоров Intel).
- Для виртуальной машины узла контейнера также понадобится по крайней мере 2 виртуальных процессора.





