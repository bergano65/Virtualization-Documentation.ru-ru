# Развертывание узла контейнера — Windows Server

**Это предварительное содержимое. Возможны изменения.**

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типов операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальной и физической). Действия, рассматриваемые в этом документе, позволяют развернуть узел контейнера Windows на Windows Server 2016 или Windows Server Core 2016 в физической или виртуальной системе. Описание установки узла контейнера Windows на Nano Server см. в разделе [Развертывание узла контейнеров — Nano Server](./deployment_nano.md).

Дополнительные сведения о требованиях к системе см. в разделе [Требования к системе узла контейнера Windows](./system_requirements.md).

Развертывание узла контейнера Windows можно автоматизировать с помощью сценариев PowerShell.
- [Развертывание узла контейнера на новой виртуальной машине Hyper-V](../quick_start/container_setup.md).
- [Развертывание узла контейнера в существующей системе](../quick_start/inplace_setup.md).

# Узел Windows Server

Действия, приведенные в этой таблице, позволят развернуть узел контейнера в Windows Server 2016 и Windows Server 2016 Core. Указаны также настройки, необходимые для контейнеров Windows Server и Hyper-V.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Действие развертывания</strong></td>
<td width="70%"><strong>Подробности</strong></td>
</tr>
<tr>
<td>[Установка компонента контейнеров](#role)</td>
<td>Компонент контейнеров позволяет использовать контейнеры Windows Server и Hyper-V.</td>
</tr>
<tr>
<td>[Создание виртуального коммутатора](#vswitch)</td>
<td>Контейнеры подключаются к виртуальному коммутатору для установления сетевого соединения.</td>
</tr>
<tr>
<td>[Настройка преобразования сетевых адресов (NAT)](#nat)</td>
<td>Если виртуальный коммутатор настроен с преобразованием сетевых адресов (NAT), необходимо настроить и NAT.</td>
</tr>
<tr>
<td>[Установка образов операционной системы контейнера](#img)</td>
<td>Образы ОС представляют собой основу для развертывания контейнера.</td>
</tr>
<tr>
<td>[Установка Docker](#docker)</td>
<td>Этот шаг необязательный, но его необходимо выполнить, чтобы создавать контейнеры Windows и управлять ими с помощью Docker.</td>
</tr>
</table>

Эти шаги необходимо выполнить, если планируется использовать контейнеры Hyper-V. Обратите внимание, что этапы, отмеченные знаком "*", необходимы, только если узел контейнера сам является виртуальной машиной Hyper-V.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Действие развертывания</strong></td>
<td width="70%"><strong>Подробности</strong></td>
</tr>
<tr>
<td>[Включение роли Hyper-V](#hypv) </td>
<td>Hyper-V требуется, только если будут использоваться контейнеры Hyper-V.</td>
</tr>
<tr>
<td>[Включение вложенной виртуализации *](#nest)</td>
<td>Если узел контейнера — виртуальная машина Hyper-V, необходимо включить вложенную виртуализацию.</td>
</tr>
<tr>
<td>[Настройка виртуальных процессоров *](#proc)</td>
<td>Если узел контейнера — виртуальная машина Hyper-V, необходимо настроить по крайней мере два виртуальных процессора.</td>
</tr>
<tr>
<td>[Отключение динамической памяти *](#dyn)</td>
<td>Если узел контейнера — виртуальная машина Hyper-V, необходимо отключить динамическую память.</td>
</tr>
<tr>
<td>[Настройка спуфинга MAC-адресов *](#mac)</td>
<td>Если узел контейнера виртуализирован, необходимо включить спуфинг MAC-адресов.</td>
</tr>
</table>

## Шаги развертывания

### <a name=role></a>Установка компонента контейнеров

Компонент контейнеров можно установить на Windows Server 2016 или Windows Server 2016 Core с помощью Windows Server Manager или PowerShell.

Чтобы установить роль с помощью PowerShell, выполните следующую команду в сеансе PowerShell с повышенными правами.

```powershell
PS C:\> Install-WindowsFeature containers
```
Когда установка роли контейнера завершится, необходимо перезагрузить систему.

```powershell
PS C:\> shutdown /r 
```
После перезагрузки системы используйте команду `Get-ContainerHost`, чтобы убедиться, что роль контейнера успешно установлена:

```powershell
PS C:\> Get-ContainerHost

Name            ContainerImageRepositoryLocation
----            --------------------------------
WIN-LJGU7HD7TEP C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store
```

### <a name=vswitch></a>Создание виртуального коммутатора

Каждый контейнер необходимо подключить к виртуальному коммутатору для связи по сети. Виртуальный коммутатор можно создать с помощью команды `New-VMSwitch`. Контейнеры поддерживают виртуальные коммутаторы типа `внешний` или `NAT`. Подробные сведения о работе контейнеров в сети см. в разделе [Сетевые подключения контейнеров](../management/container_networking.md).

В этом примере создается виртуальный коммутатор с именем "Virtual Switch" типа "NAT" и подсеть NAT 172.16.0.0/12.

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress 172.16.0.0/12
```

### <a name=nat></a>Настройка преобразования сетевых адресов (NAT)

Если тип коммутатора — NAT, в дополнение к виртуальному коммутатору необходимо создать объект NAT. Это можно сделать с помощью команды `New-NetNat`. В этом примере создается объект NAT с именем `ContainerNat`, а также префиксом адреса, соответствующим подсети NAT, назначенной для данного коммутатора контейнера.

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

### <a name=img></a>Установка образов ОС

Образ ОС используется как основа для любого контейнера Windows Server или Hyper-V. Образ используется для развертывания контейнера, который затем можно изменить и поместить в новый образ контейнера. Образы ОС были созданы с использованием Windows Server Core и Nano Server как основной операционной системы.

Образы ОС контейнеров можно найти и установить с помощью модуля ContainerProvider PowerShell. Прежде чем использовать этот модуль, его необходимо установить. Для установки модуля можно использовать приведенные ниже команды.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Чтобы получить список образов из диспетчера пакетов OneGet в PowerShell, воспользуйтесь командой `Find-ContainerImage`:
```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```
Чтобы скачать и установить базовый образ Nano Server, выполните приведенную ниже команду.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Кроме того, эта команда позволит скачать и установить базовый образ Windows Server Core.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

**Проблема.** Командлеты Save-ContainerImage и Install-ContainerImage не работают с образом контейнера WindowsServerCore в сеансе удаленного взаимодействия PowerShell.<br /> **Решение.** Войдите на этот компьютер с помощью удаленного рабочего стола и напрямую используйте командлет Save-ContainerImage.

С помощью команды `Get-ContainerImage` убедитесь, что эти образы были установлены.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
Дополнительные сведения об управлении образами контейнеров см. в статье [Образы контейнеров Windows](../management/manage_images.md).


### <a name=docker></a>Установка Docker

Управляющая программа и интерфейс командной строки для Docker не поставляются вместе с Windows и не устанавливаются с помощью компонента контейнеров Windows. Docker не обязательно использовать для работы с контейнерами Windows. Чтобы установить Docker, следуйте инструкциям из статьи [Docker и Windows](./docker_windows.md).


## Узел контейнера Hyper-V

### <a name=hypv></a>Включение роли Hyper-V

Если будут развертываться контейнеры Hyper-V, роль Hyper-V должна быть включена на узле контейнера. Роль Hyper-V можно установить в Windows Server 2016 или Windows Server 2016 Core с помощью команды `Install-WindowsFeature`. Если узел контейнера — виртуальная машина Hyper-V, необходимо сначала включить вложенную виртуализацию. Для этого см. статью [Настройка вложенной виртуализации](#nest).

```powershell
PS C:\> Install-WindowsFeature hyper-v
```

### <a name=nest></a>Настройка вложенной виртуализации

Если вы хотите разместить контейнеры Hyper-V на узле контейнера, работающем на виртуальной машине Hyper-V, необходимо включить вложенную виртуализацию. Это можно сделать с помощью приведенной ниже команды PowerShell.

**Примечание.** При выполнении этой команды виртуальные машины должны быть отключены.

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>Настройка виртуальных процессоров

Если вы хотите разместить контейнеры Hyper-V на узле контейнера, работающем на виртуальной машине Hyper-V, для виртуальной машины требуется как минимум два процессора. Это можно настроить с помощью параметров виртуальной машины или приведенной ниже команды.

**Примечание.** При выполнении этой команды виртуальные машины должны быть отключены.

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=dyn></a>Отключение динамической памяти

Если узел контейнера сам является виртуальной машиной Hyper-V, необходимо отключить динамическую память на виртуальной машине этого узла контейнера. Это можно настроить с помощью параметров виртуальной машины или приведенной ниже команды.

**Примечание.** При выполнении этой команды виртуальные машины должны быть отключены.

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>Настройка спуфинга MAC-адресов

Наконец, если узел контейнера работает на виртуальной машине Hyper-V, необходимо включить спуфинг MAC-адресов. Благодаря этому каждый контейнер получит IP-адрес. Чтобы включить спуфинг MAC-адресов, выполните приведенную ниже команду на узле Hyper-V. Свойством VMName будет имя узла контейнера.

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```





<!--HONumber=Feb16_HO1-->
