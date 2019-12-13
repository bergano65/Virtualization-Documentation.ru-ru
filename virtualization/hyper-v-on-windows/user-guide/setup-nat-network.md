---
title: Настройка сети NAT
description: Настройка сети NAT
keywords: windows 10, hyper-v
author: jmesser81
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
ms.openlocfilehash: e69775c15359645f3659c9bee3562733415228d5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909434"
---
# <a name="set-up-a-nat-network"></a>Настройка сети NAT

Windows 10 Hyper-V разрешает использовать для виртуальной сети собственное преобразование сетевых адресов (NAT).

В этом руководство рассматриваются следующие темы:
* Создание сети NAT
* Подключение существующей виртуальной машины к новой сети
* Проверка правильности подключения виртуальной машины

Требования:
* Юбилейное обновление Windows 10 или более поздняя версия
* Hyper-V включен (инструкции см. [здесь](../quick-start/enable-hyper-v.md))

> **Примечание.** Сейчас можно создать только одну сеть NAT для узла. Дополнительные сведения о реализации, возможностях и ограничениях NAT для Windows (WinNAT) см. в [блоге, посвященном возможностям и ограничениям WinNAT](https://techcommunity.microsoft.com/t5/Virtualization/Windows-NAT-WinNAT-Capabilities-and-limitations/ba-p/382303).

## <a name="nat-overview"></a>Обзор NAT
NAT предоставляет виртуальной машине доступ к сетевым ресурсам с помощью IP-адреса и порта главного компьютера через внутренний виртуальный коммутатор Hyper-V.

Преобразования сетевых адресов (NAT) — это сетевой режим, предназначенный для экономии IP-адресов за счет сопоставления внешнего IP-адреса и порта с гораздо большим набором внутренних IP-адресов.  По сути, NAT использует таблицу потоков для маршрутизации трафика с внешнего IP-адреса (адреса узла) и номера порта на правильный внутренний IP-адрес, связанный с конечной точкой в сети (виртуальной машиной, компьютером, контейнером и т. д.).

Кроме того, NAT позволяет нескольким виртуальным машинам размещать приложения, которым требуются одинаковые (внутренние) порты связи, сопоставляя их с уникальными внешними портами.

По всем этим причинам NAT часто применяется в технологии контейнеров (см. статью [Сетевые подключения контейнеров](https://docs.microsoft.com/virtualization/windowscontainers/container-networking/architecture)).


## <a name="create-a-nat-virtual-network"></a>Создание виртуальной сети NAT
Давайте рассмотрим настройку новой сети NAT.

1.  Откройте консоль PowerShell от имени администратора.  

2. Создайте внутренний коммутатор.

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. Найдите индекс интерфейса созданного виртуального коммутатора.

    Индекс интерфейса можно найти, выполнив `Get-NetAdapter`

    Выходные данные должны иметь следующий вид:

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    Внутренний коммутатор будет иметь такое имя, как `vEthernet (SwitchName)`, и описание интерфейса `Hyper-V Virtual Ethernet Adapter`. Запишите его `ifIndex` для использования на следующем шаге.

4. Настройте шлюз NAT с помощью [New-NetIPAddress](https://docs.microsoft.com/powershell/module/nettcpip/New-NetIPAddress).  

  Ниже приведена общая команда:
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  Чтобы настроить шлюз, вам потребуется некоторая информация о сети:  
  * **IPAddress** — "NAT Gateway IP" задает IP-адрес шлюза NAT в формате IPv4 или IPv6.  
    Общая форма имеет вид a.b.c.1 (например, 172.16.0.1).  Хотя последняя позиция необязательно должна быть равна 1, обычно используется именно это значение (в зависимости от длины префикса).

    Общий IP-адрес шлюза имеет значение 192.168.0.1.  

  * **PrefixLength** — "NAT Subnet Prefix Length" определяет размер локальной подсети NAT (маску подсети).
    Длина префикса подсети является целым числом от 0 до 32.

    Значение 0 соответствует всему Интернету, а значение 32 — всего одному IP-адресу.  Обычно используются значения в диапазоне от 24 до 12 в зависимости от того, сколько IP-адресов необходимо подключить к NAT.

    Общее значение PrefixLength равно 24. Это маска подсети 255.255.255.0.

  * **InterfaceIndex**: ifIndex — это индекс интерфейса виртуального коммутатора, который вы определили на предыдущем шаге.

  Выполните следующую команду, чтобы создать шлюз NAT:

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

5. Настройте сеть NAT с помощью [New-NetNat](https://docs.microsoft.com/powershell/module/netnat/New-NetNat).  

  Ниже приведена общая команда:

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  Чтобы настроить шлюз, потребуется указать информацию о сети и шлюзе NAT:  
  * **Name** — NATOutsideName описывает имя сети NAT.  Оно используется для удаления сети NAT.

  * **InternalIPInterfaceAddressPrefix** — "NAT subnet prefix" задает описанные ранее префикс IP-адреса шлюза NAT и длину префикса подсети NAT.

    Общая форма имеет вид a.b.c.0/NAT Subnet Prefix Length.

    Учитывая приведенные выше данные, для этого примера мы используем 192.168.0.0/24.

  В рамках данного примера выполните следующую команду для настройки сети NAT:

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

Поздравляем!  Теперь у вас есть виртуальная сеть NAT.  Чтобы добавить виртуальную машину в сеть NAT, выполните [эти инструкции](#connect-a-virtual-machine).

## <a name="connect-a-virtual-machine"></a>Соединение с виртуальной машиной

Чтобы подключить виртуальную машину к новой сети NAT, подключите внутренний коммутатор, созданный на первом шаге в разделе [Настройка сети NAT](#create-a-nat-virtual-network), к виртуальной машине с помощью меню параметров виртуальной машины.

Так как служба WinNAT сама по себе не выделяет и не назначает IP-адреса конечным точкам (например, виртуальной машины), вам потребуется сделать это вручную в виртуальной машине, т. е. задать IP-адреса в диапазоне внутреннего префикса NAT, задать IP-адрес шлюза по умолчанию, указать данные DNS-сервера. Единственной оговоркой является наличие подключения конечной точки к контейнеру. В этом случае служба HNS выделяет и использует службу HCS для назначения IP-адреса, IP-адреса шлюза и сведений о DNS непосредственно контейнеру.


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>Пример конфигурации: подключение виртуальных машин и контейнеров к сети NAT

_Если необходимо подключить несколько виртуальных машин и контейнеров к одному NAT, необходимо убедиться, что префикс внутренней подсети NAT достаточно велик, чтобы охватывать диапазоны IP-адресов, назначаемые различными приложениями или службами (например, Docker для Windows и контейнер Windows). СЛУЖБЕ HNS). Для этого потребуется назначение уровня приложения IP-адресов и конфигурации сети или ручная настройка, которая должна выполняться администратором и гарантировать, что не будет повторно использовать существующие назначения IP-адресов на том же узле._

### <a name="docker-for-windows-linux-vm-and-windows-containers"></a>Docker для Windows (для виртуальных машин Linux) и компонент контейнеров Windows
Приведенное ниже решение позволит Docker для Windows (виртуальным машинам Linux с контейнерами Linux) и компоненту контейнеров Windows совместно использовать общий экземпляр WinNAT с помощью отдельных внутренних коммутаторов vSwitch. Будет работать подключение между контейнерами Linux и Windows.

Пользователь подключил виртуальные машины к сети NAT через внутренний коммутатор vSwitch с именем VMNAT и теперь хочет установить компонент "Контейнеры Windows" с подсистемой Dосker.
```
PS C:\> Get-NetNat “VMNAT”| Remove-NetNat (this will remove the NAT but keep the internal vSwitch).
Install Windows Container Feature
DO NOT START Docker Service (daemon)
Edit the arguments passed to the docker daemon (dockerd) by adding –fixed-cidr=<container prefix> parameter. This tells docker to create a default nat network with the IP subnet <container prefix> (e.g. 192.168.1.0/24) so that HNS can allocate IPs from this prefix.
PS C:\> Start-Service Docker; Stop-Service Docker
PS C:\> Get-NetNat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> Start-Service docker
```
DOCKER/HNS назначает IP-адреса контейнерам Windows, а администратор назначает им IP-адреса для виртуальных машин из набора разностей этих двух.

Пользователь установил компонент "Контейнеры Windows" с работающей подсистемой Docker и хочет подключить виртуальные машины к сети NAT.
```
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
PS C:\> Get-NetNat | Remove-NetNat (this will remove the NAT but keep the internal vSwitch)
Edit the arguments passed to the docker daemon (dockerd) by adding -b “none” option to the end of docker daemon (dockerd) command to tell docker not to create a default NAT network.
PS C:\> New-ContainerNetwork –name nat –Mode NAT –subnetprefix <container prefix> (create a new NAT and internal vSwitch – HNS will allocate IPs to container endpoints attached to this network from the <container prefix>)
PS C:\> Get-Netnat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> New-VirtualSwitch -Type internal (attach VMs to this new vSwitch)
PS C:\> Start-Service docker
```
DOCKER/HNS назначает IP-адреса контейнерам Windows, а администратор назначает им IP-адреса для виртуальных машин из набора разностей этих двух.

В итоге вы должны получить два внутренних коммутатора виртуальных машин и один общий для них коммутатор NetNat.

## <a name="multiple-applications-using-the-same-nat"></a>Несколько приложений, использующих одну систему NAT

В некоторых сценариях требуется, чтобы несколько приложений или служб использовали одну систему NAT. В этом случае необходимо придерживаться описанной ниже процедуры, чтобы несколько приложений или служб могли использовать больший внутренний префикс подсети NAT.

**_Мы подробно рассмотрим виртуальную машину DOCKER 4 Windows-DOCKER Beta-Linux с функцией контейнера Windows на том же узле, что и пример. Этот рабочий процесс может быть изменен_**

1. C:\> net stop docker
2. Stop Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *Удаляет все ранее существующие сети контейнеров (т. е. удаляет vSwitch, удаляет NetNat, очищает).*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (эта подсеть используется для компонента контейнеров Windows) *Создает внутренний vSwitch с именем nat.*  
   *Создает сеть NAT с именем "NAT" с префиксом IP-адресов 10.0.76.0/24.*  

6. Remove-NetNAT  
   *Удаляет сети NAT DockerNAT и NAT (поддерживает внутренний коммутаторы vSwitch)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (создает более крупную сеть NAT для совместного использования D4W и контейнерами)  
   *Создает сеть NAT с именем DockerNAT с более крупным префиксом 10.0.0.0/17*  

8. Run Docker4Windows (MobyLinux.ps1)  
   *Создает внутренний DockerNAT vSwitch*  
   *Создает сеть NAT с именем "DockerNAT" с префиксом IP-адреса 10.0.75.0/24*  

9. Net start docker  
   *DOCKER будет использовать определенную пользователем сеть NAT в качестве значения по умолчанию для подключения контейнеров Windows.*  

В конце вы должны получить два внутренних коммутатора vSwitch — один с именем DockerNAT, другой с именем nat. При выполнении Get-NetNat выводится только одна подтвержденная сеть NAT (10.0.0.0/17). IP-адреса для контейнеров Windows назначаются сетевой службой узлов Windows (HNS) (HNS) из подсети 10.0.76.0/24. В соответствии с имеющимся сценарием MobyLinux.ps1 IP-адреса для Docker 4 Windows назначаются из подсети 10.0.75.0/24.


## <a name="troubleshooting"></a>Поиск и устранение неисправностей

### <a name="multiple-nat-networks-are-not-supported"></a>Несколько сетей NAT не поддерживается.  
В этом руководстве предполагается, что других NAT на узле нет. Приложениям или службам необходимо использовать NAT, и они могут создать ее в процессе установки. Поскольку Windows (WinNAT) поддерживает только один внутренний префикс подсети NAT, при попытке создать несколько NAT система переходит в неизвестное состояние.

Чтобы понять, является ли это проблемой, убедитесь, что имеется только одна NAT.
``` PowerShell
Get-NetNat
```

Если NAT уже существует, удалите ее.
``` PowerShell
Get-NetNat | Remove-NetNat
```
Убедитесь, что для приложения или компонента (например, для контейнеров Windows) имеется всего один "внутренний" vmSwitch. Запишите имя vSwitch.
``` PowerShell
Get-VMSwitch
```

Проверьте, есть ли частные IP-адреса (например, IP-адрес шлюза NAT по умолчанию обычно имеет значение *.1) старого NAT, по-прежнему назначенные адаптеру.
``` PowerShell
Get-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)"
```

Если используется старый частный IP-адрес, удалите его.
``` PowerShell
Remove-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)" -IPAddress <IPAddress>
```

**Удаление нескольких сетевых адресов**  
Мы встречали сообщения о нескольких случайно созданных сетях NAT. Это вызвано ошибкой, присутствующей в последних сборках (включая Windows Server 2016 Technical Preview 5 и Windows 10 Insider Preview). Если после запуска команд ls или Get-ContainerNetwork сети Docker появится несколько сетей NAT, в командной строке PowerShell с повышенными привилегиями выполните следующее:

```
PS> $KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
PS> $keys = get-childitem $KeyPath
PS> foreach($key in $keys)
PS> {
PS>    if ($key.GetValue("FriendlyName") -eq 'nat')
PS>    {
PS>       $newKeyPath = $KeyPath+"\"+$key.PSChildName
PS>       Remove-Item -Path $newKeyPath -Recurse
PS>    }
PS> }
PS> remove-netnat -Confirm:$false
PS> Get-ContainerNetwork | Remove-ContainerNetwork
PS> Get-VmSwitch -Name nat | Remove-VmSwitch (_failure is expected_)
PS> Stop-Service docker
PS> Set-Service docker -StartupType Disabled
Reboot Host
PS> Get-NetNat | Remove-NetNat
PS> Set-Service docker -StartupType automaticac
PS> Start-Service docker 
```

При необходимости обратитесь к этому [руководству по установке для нескольких приложений, использующих одну NAT](#multiple-applications-using-the-same-nat) для перестройки среды NAT. 

## <a name="references"></a>Ссылок
Дополнительные сведения о [сетях NAT](https://en.wikipedia.org/wiki/Network_address_translation)
