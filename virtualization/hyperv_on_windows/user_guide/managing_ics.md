---
title: &136673742 Управление службами интеграции Hyper-V
description: Управление службами интеграции Hyper-V
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1795869236 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 9cafd6cb-dbbe-4b91-b26c-dee1c18fd8c2
---

# Управление службами интеграции Hyper-V

Службы интеграции (часто называемые компонентами интеграции) — это службы, позволяющие виртуальной машине связываться с узлом Hyper-V. Многие из этих служб используются для удобства (например, копирования файлов гостевой ОС), а другие могут быть достаточно важны для правильной работы гостевой ОС (синхронизация времени).

В этой статье подробно рассматривается управление службами интеграции с помощью диспетчера Hyper-V и оболочки PowerShell в Windows 10. Дополнительные сведения о каждой отдельной службе интеграции см. в статье <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Службы интеграции</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

## Включение и отключение служб интеграции с помощью диспетчера Hyper-V

1. Выберите виртуальную машину и откройте параметры.
  <g id="1" ctype="x-linkText"></g>

2. В окне параметров виртуальной машины перейдите на вкладку "Службы интеграции" в разделе "Управление".

  <g id="1" ctype="x-linkText"></g>

  Здесь указаны все службы интеграции, доступные на этом узле Hyper-V. Обратите внимание, что гостевая операционная система может поддерживать не все указанные службы интеграции.

## Включение и отключение служб интеграции с помощью PowerShell

Службы интеграции также можно включать и отключать в PowerShell с помощью командлетов [<g id="2" ctype="x-code">Enable-VMIntegrationService</g>](https://technet.microsoft.com/en-us/library/hh848500.aspx) и [<g id="4" ctype="x-code">Disable-VMIntegrationService</g>](https://technet.microsoft.com/en-us/library/hh848488.aspx).

В следующем примере мы включим, а затем отключим службу копирования файлов гостевой ОС на виртуальной машине demovm, показанной выше.

1. Узнайте, какие службы интеграции запущены.

  ``` PowerShell
  Get-VMIntegrationService -VMName "demovm"
  ```

  Результат будет выглядеть так:
  ``` PowerShell
  VMName      Name                    Enabled PrimaryStatusDescription SecondaryStatusDescription
  ------      ----                    ------- ------------------------ --------------------------
  demovm      Guest Service Interface False   OK
  demovm      Heartbeat               True    OK                       OK
  demovm      Key-Value Pair Exchange True    OK
  demovm      Shutdown                True    OK
  demovm      Time Synchronization    True    OK
  demovm      VSS                     True    OK
  ```

2. Включите службу интеграции <g id="2" ctype="x-code">Интерфейс гостевой службы</g>.

   ``` PowerShell
   Enable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

   Служба интеграции "Интерфейс гостевой службы" включается с помощью команды <g id="2" ctype="x-code">Get-VMIntegrationService -VMName "demovm"</g>.

3. Отключите службу интеграции <g id="2" ctype="x-code">Интерфейс гостевой службы</g>.

   ``` PowerShell
   Disable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

Службы интеграции работают, только если они включены и на узле, и в гостевой операционной системе. Все службы интеграции в гостевых ОС Windows включены по умолчанию, но их можно отключить. Инструкции приведены в следующем разделе.


## Управление службами интеграции из гостевой ОС (Windows)

> <g id="1" ctype="x-strong">Примечание.</g> Отключение служб интеграции может серьезно повлиять на возможность узлов управлять вашей виртуальной машиной. Службы интеграции работают, только если они включены и на узле, и в гостевой операционной системе.

Службы интеграции отображаются как службы Windows. Чтобы включить или отключить службы интеграции из виртуальной машины, откройте диспетчер служб Windows.

<g id="1" ctype="x-linkText"></g>

Найдите службы, в имени которых есть слово Hyper-V. Щелкните правой кнопкой мыши службу, которую требуется включить или отключить, а затем запустите или остановите ее.

Чтобы просмотреть все службы интеграции с помощью PowerShell, выполните следующую команду:

```PowerShell
Get-Service -Name vm*
```

Отобразится примерно такой список:

```PowerShell
Status   Name               DisplayName
------   ----               -----------
Running  vmicguestinterface Hyper-V Guest Service Interface
Running  vmicheartbeat      Hyper-V Heartbeat Service
Running  vmickvpexchange    Hyper-V Data Exchange Service
Running  vmicrdv            Hyper-V Remote Desktop Virtualizati...
Running  vmicshutdown       Hyper-V Guest Shutdown Service
Running  vmictimesync       Hyper-V Time Synchronization Service
Stopped  vmicvmsession      Hyper-V VM Session Service
Running  vmicvss            Hyper-V Volume Shadow Copy Requestor
```

Запустите или остановите службы с помощью командлетов [<g id="2" ctype="x-code">Start-Service</g>](https://technet.microsoft.com/en-us/library/hh849825.aspx) и [<g id="4" ctype="x-code">Stop-Service</g>](https://technet.microsoft.com/en-us/library/hh849790.aspx).

Например, чтобы отключить PowerShell Direct, можно запустить <g id="2" ctype="x-code">Stop-Service -Name vmicvmsession</g>.

По умолчанию в гостевой операционной системе все службы интеграции включены.

## Управление службами интеграции из гостевой ОС (Linux)

Службы интеграции Linux обычно предоставляются через ядро Linux.

Убедитесь, что драйвер и управляющие программы служб интеграции запущены, выполнив указанные ниже команды в гостевой операционной системе Linux.

1. Драйвер служб интеграции Linux называется hv_utils. Чтобы узнать, загружен ли он, выполните следующую команду:

  ``` BASH
  lsmod | grep hv_utils
  ```

  Результат должен выглядеть примерно так:

  ``` BASH
  Module                  Size   Used by
  hv_utils               20480   0
  hv_vmbus               61440   8 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc
  ```

2. Чтобы проверить, запускаются ли необходимые управляющие программы, в гостевой операционной системе Linux выполните следующую команду:

  ``` BASH
  ps -ef | grep hv
  ```

  Результат должен выглядеть примерно так:

  ``` BASH
  root       236     2  0 Jul11 ?        00:00:00 [hv_vmbus_con]
  root       237     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  ...
  root       252     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  root      1286     1  0 Jul11 ?        00:01:11 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9333     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9365     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_vss_daemon
  scooley  43774 43755  0 21:20 pts/0    00:00:00 grep --color=auto hv          
  ```

  Чтобы отобразить все доступные управляющие программы, выполните следующую команду:
  ``` BASH
  compgen -c hv_
  ```

  Результат должен выглядеть примерно так:

  ``` BASH
  hv_vss_daemon
  hv_get_dhcp_info
  hv_get_dns_info
  hv_set_ifconfig
  hv_kvp_daemon
  hv_fcopy_daemon     
  ```

  Вы можете увидеть такие управляющие команды:
  * **<g id="2" ctype="x-code">hv_vss_daemon</g>**— необходима для создания динамических резервных копий виртуальной машины Linux.
  * **<g id="2" ctype="x-code">hv_kvp_daemon</g>** — позволяет задавать и запрашивать внутренние и внешние пары "ключ-значение".
  * **<g id="2" ctype="x-code">hv_fcopy_daemon</g>** — реализует службу копирования файлов между узлом и гостевой операционной системой.

> <g id="1" ctype="x-strong">Примечание.</g> Если указанные выше управляющие программы служб интеграции недоступны, возможно, они не поддерживаются вашей операционной системой или не установлены. Дополнительные сведения о дистрибутивах Linux можно найти <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">здесь</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

В следующем примере мы остановим и запустим управляющую программу KVP — <g id="2" ctype="x-code">hv_kvp_daemon</g>.

Остановите процесс управляющей программы с помощью идентификатора процесса, указанного во втором столбце приведенных выше результатов. Кроме того, нужный процесс можно найти с помощью программы <g id="2" ctype="x-code">pidof</g>. Так как управляющие программы Hyper-V запускаются как корень, необходимы разрешения корневой папки.

``` BASH
sudo kill -15 `pidof hv_kvp_daemon`
```

Теперь, если выполнить команду <g id="2" ctype="x-code">ps -ef | hv</g> снова, вы обнаружите, что процесса <g id="4" ctype="x-code">hv_kvp_daemon</g> больше нет.

Чтобы запустить управляющую программу снова, запустите ее как корень.

``` BASH
sudo hv_kvp_daemon
```

Теперь, если выполнить команду <g id="2" ctype="x-code">ps -ef | hv</g> снова, вы обнаружите <g id="4" ctype="x-code">hv_kvp_daemon</g> с новым идентификатором процесса.


## Обслуживание служб интеграции

Для максимальной производительности и функциональности виртуальной машины требуется регулярное обновление служб интеграции.

<g id="1" ctype="x-strong">Для виртуальных машин, работающих на узлах под управлением Windows 10:</g>

> <g id="1" ctype="x-strong">Примечание.</g> Файл ISO-образа vmguest.iso больше не требуется для обновления компонентов интеграции. Он не входит в Hyper-V в Windows 10.

| Гостевая ОС| Механизм обновления| Заметки|
|:---------|:---------|:---------|
| Windows 10| Центр обновления Windows| |
| Windows 8.1| Центр обновления Windows| |
| Windows 8| Центр обновления Windows| Требуется служба интеграции обмена данными<g id="2" ctype="x-strong">*</g>.|
| Windows 7| Центр обновления Windows| Требуется служба интеграции обмена данными<g id="2" ctype="x-strong">*</g>.|
| Windows Vista с пакетом обновления 2 (SP2)| Центр обновления Windows| Требуется служба интеграции обмена данными<g id="2" ctype="x-strong">*</g>.|
| —| | |
| Windows Server 2012 R2| Центр обновления Windows| |
| Windows Server 2012| Центр обновления Windows| Требуется служба интеграции обмена данными<g id="2" ctype="x-strong">*</g>.|
| Windows Server 2008 R2 с пакетом обновления 1 (SP1)| Центр обновления Windows| Требуется служба интеграции обмена данными<g id="2" ctype="x-strong">*</g>.|
| Windows Server 2008 с пакетом обновления 2 (SP 2)| Центр обновления Windows| Расширенная поддержка только для Server 2016 (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Подробнее</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| Windows Home Server 2011| Центр обновления Windows| Не будет поддерживаться в Server 2016 (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Подробнее</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| Windows Small Business Server 2011| Центр обновления Windows| Не распространяется базовая поддержка (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Подробнее</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| —| | |
| Гостевые ОС Linux| диспетчер пакетов| Компоненты интеграции для Linux уже встроены в дистрибутив. Могут быть доступны необязательные обновления.<g id="1" ctype="x-strong">****</g>|

<g id="1" ctype="x-strong">\*</g> Если служба интеграции обмена данными не включается, компоненты интеграции для этих гостевых систем можно найти в центре загрузки <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">здесь</g><g id="3CapsExtId3" ctype="x-title"></g></g> в формате CAB-файла. Инструкции по применению CAB-файлов можно найти <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">здесь</g><g id="2CapsExtId3" ctype="x-title"></g></g>.


<g id="1" ctype="x-strong">Для виртуальных машин, работающих на узлах под управлением Windows 8.1:</g>

| Гостевая ОС| Механизм обновления| Заметки|
|:---------|:---------|:---------|
| Windows 10| Центр обновления Windows| |
| Windows 8.1| Центр обновления Windows| |
| Windows 8| Диск со службами интеграции| |
| Windows 7| Диск со службами интеграции| |
| Windows Vista с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| Windows XP с пакетами обновления 2 и 3 (SP2, SP3)| Диск со службами интеграции| |
| —| | |
| Windows Server 2012 R2| Центр обновления Windows| |
| Windows Server 2012| Диск со службами интеграции| |
| Windows Server 2008 R2| Диск со службами интеграции| |
| Windows Server 2008 с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| Windows Home Server 2011| Диск со службами интеграции| |
| Windows Small Business Server 2011| Диск со службами интеграции| |
| Windows Server 2003 R2 с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| Windows Server 2003 с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| —| | |
| Гостевые ОС Linux| диспетчер пакетов| Компоненты интеграции для Linux уже встроены в дистрибутив. Могут быть доступны необязательные обновления.<g id="1" ctype="x-strong">****</g>|


<g id="1" ctype="x-strong">Для виртуальных машин, работающих на узлах под управлением Windows 8:</g>

| Гостевая ОС| Механизм обновления| Заметки|
|:---------|:---------|:---------|
| Windows 8.1| Центр обновления Windows| |
| Windows 8| Диск со службами интеграции| |
| Windows 7| Диск со службами интеграции| |
| Windows Vista с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| Windows XP с пакетами обновления 2 и 3 (SP2, SP3)| Диск со службами интеграции| |
| —| | |
| Windows Server 2012 R2| Центр обновления Windows| |
| Windows Server 2012| Диск со службами интеграции| |
| Windows Server 2008 R2| Диск со службами интеграции| |
| Windows Server 2008 с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| Windows Home Server 2011| Диск со службами интеграции| |
| Windows Small Business Server 2011| Диск со службами интеграции| |
| Windows Server 2003 R2 с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| Windows Server 2003 с пакетом обновления 2 (SP2)| Диск со службами интеграции| |
| —| | |
| Гостевые ОС Linux| диспетчер пакетов| Компоненты интеграции для Linux уже встроены в дистрибутив. Могут быть доступны необязательные обновления.<g id="1" ctype="x-strong">****</g>|


Инструкции по обновлению с помощью диска со службами интеграции для Windows 8 и Windows 8.1 можно найти <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">здесь</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

 <g id="1" ctype="x-strong">\****</g> Дополнительные сведения о гостевых ОС Linux см. <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">здесь</g><g id="3CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=May16_HO1-->


