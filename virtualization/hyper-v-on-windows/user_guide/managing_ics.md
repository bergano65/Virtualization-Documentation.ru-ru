---
title: "Управление службами интеграции Hyper-V"
description: "Управление службами интеграции Hyper-V"
keywords: "windows10, hyper-v, службы интеграции, компоненты интеграции"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 9cafd6cb-dbbe-4b91-b26c-dee1c18fd8c2
redirect_url: https://technet.microsoft.com/windows-server-docs/compute/hyper-v/manage/manage-Hyper-V-integration-services
ms.openlocfilehash: 83bcc4c2f47e2a3921be257f45a3a0e22dcba89a
ms.sourcegitcommit: fd6c5ec419aae425af7ce6c6a44d59c98f62502a
ms.translationtype: HT
ms.contentlocale: ru-RU
---
# <a name="managing-hyper-v-integration-services"></a>Управление службами интеграции Hyper-V

Службы интеграции (часто называемые компонентами интеграции) — это службы, позволяющие виртуальной машине связываться с узлом Hyper-V. Многие из этих служб используются для удобства (например, для копирования файлов гостевой ОС), а другие могут быть достаточно важны для правильной работы виртуальной машины (синхронизация времени).

В этой статье подробно рассматривается управление службами интеграции с помощью диспетчера Hyper-V и оболочки PowerShell в Windows 10.  

Дополнительные сведения о каждой отдельной службе интеграции см. в статье [Службы интеграции](../reference/integration-services.md).

## <a name="enable-or-disable-integration-services-using-hyper-v-manager"></a>Включение и отключение служб интеграции с помощью диспетчера Hyper-V

1. Выберите виртуальную машину и откройте параметры.
  
2. В окне параметров виртуальной машины перейдите на вкладку "Службы интеграции" в разделе "Управление".
  
  Здесь указаны все службы интеграции, доступные на этом узле Hyper-V.  Обратите внимание, что гостевая операционная система может поддерживать не все указанные службы интеграции. Чтобы узнать данные о версии для гостевой операционной системы, войдите в нее и выполните следующую команду из командной строки.

REG QUERY "HKLM\Software\Microsoft\Virtual Machine\Auto" /v IntegrationServicesVersion

## <a name="enable-or-disable-integration-services-using-powershell"></a>Включение и отключение служб интеграции с помощью PowerShell

Службы интеграции можно также включить и отключить через PowerShell, выполнив [`Enable-VMIntegrationService`](https://technet.microsoft.com/en-us/library/hh848500.aspx) и [`Disable-VMIntegrationService`](https://technet.microsoft.com/en-us/library/hh848488.aspx).

В следующем примере мы включим, а затем отключим службу копирования файлов гостевой ОС на виртуальной машине demovm, показанной выше.

1. Узнайте, какие службы интеграции запущены.
  
  ``` PowerShell
  Get-VMIntegrationService -VMName "DemoVM"
  ```

  Результат будет выглядеть так:  
  ``` PowerShell
  VMName      Name                    Enabled PrimaryStatusDescription SecondaryStatusDescription
  ------      ----                    ------- ------------------------ --------------------------
  DemoVM      Guest Service Interface False   OK
  DemoVM      Heartbeat               True    OK                       OK
  DemoVM      Key-Value Pair Exchange True    OK
  DemoVM      Shutdown                True    OK
  DemoVM      Time Synchronization    True    OK
  DemoVM      VSS                     True    OK
  ```

2. Включение службы интеграции `Guest Service Interface`

   ``` PowerShell
   Enable-VMIntegrationService -VMName "DemoVM" -Name "Guest Service Interface"
   ```
   
   Если вы запустите командлет `Get-VMIntegrationService -VMName "DemoVM"`, то увидите, что служба интеграции "Интерфейс гостевой службы" будет включена.
 
3. Отключение службы интеграции `Guest Service Interface`

   ``` PowerShell
   Disable-VMIntegrationService -VMName "DemoVM" -Name "Guest Service Interface"
   ```
   
Службы интеграции работают, только если они включены и на узле, и в гостевой операционной системе.  Все службы интеграции в гостевых ОС Windows включены по умолчанию, но их можно отключить.  Инструкции приведены в следующем разделе.


## <a name="manage-integration-services-from-guest-os-windows"></a>Управление службами интеграции из гостевой ОС (Windows)

> **Примечание.** Отключение служб интеграции может серьезно повлиять на способность узла управлять вашей виртуальной машиной.  Службы интеграции работают, только если они включены и на узле, и в гостевой операционной системе.

Службы интеграции отображаются как службы Windows. Чтобы включить или отключить службы интеграции из виртуальной машины, откройте диспетчер служб Windows.

![](../user-guide/media/HVServices.png) 

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

Запуск или остановка служб с помощью [`Start-Service`](https://technet.microsoft.com/en-us/library/hh849825.aspx) или [`Stop-Service`](https://technet.microsoft.com/en-us/library/hh849790.aspx).

Например, чтобы отключить PowerShell Direct, можно запустить командлет `Stop-Service -Name vmicvmsession`.

По умолчанию в гостевой операционной системе все службы интеграции включены.

## <a name="manage-integration-services-from-guest-os-linux"></a>Управление службами интеграции из гостевой ОС (Linux)

Службы интеграции Linux обычно предоставляются через ядро Linux.

Убедитесь, что драйвер и управляющие программы служб интеграции запущены, выполнив указанные ниже команды в гостевой операционной системе Linux.

1. Драйвер служб интеграции Linux называется hv_utils.  Чтобы узнать, загружен ли он, выполните следующую команду:

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
  * **`hv_vss_daemon`**— эта управляющая программа необходима для создания динамических архивных копий виртуальной машины Linux.
  * **`hv_kvp_daemon`**— эта управляющая программа разрешает задание и запрос внутренних и внешних пар "ключ-значение".
  * **`hv_fcopy_daemon`**— эта управляющая программа внедряет службу копирования файлов между узлом и гостевой ОС.

> **Примечание.** Если управляющие программы для служб интеграции выше недоступны, они могут не поддерживаться в вашей системе или не могут быть установлены в ней.  Дополнительные сведения о дистрибутивах см. [здесь](https://technet.microsoft.com/en-us/library/dn531030.aspx).  

В этом примере показаны запуск и остановка управляющей программы KVP `hv_kvp_daemon`.

Остановите процесс управляющей программы с помощью идентификатора процесса, указанного во втором столбце приведенных выше результатов.  Кроме того, нужный процесс можно найти с помощью `pidof`.  Так как управляющие программы Hyper-V запускаются как корень, необходимы разрешения корневой папки.

``` BASH
sudo kill -15 `pidof hv_kvp_daemon`
```

Теперь при повторном запуске `ps -ef | hv` вы обнаружите, что все процессы `hv_kvp_daemon` пропали.

Чтобы запустить управляющую программу снова, запустите ее как корень.

``` BASH
sudo hv_kvp_daemon
``` 

Теперь при повторном запуске `ps -ef | hv` вы обнаружите, что у процесса `hv_kvp_daemon` появился новый идентификатор.


## <a name="integration-service-maintenance"></a>Обслуживание служб интеграции

Обслуживание службы интеграции в Windows10 осуществляется по умолчанию, при условии, что виртуальные машины могут получать важные обновления из Центра обновления Windows.  

Для обеспечения максимальной производительности и функциональности виртуальной машины требуется регулярное обновление служб интеграции.

**Для виртуальных машин, работающих на узлах под управлением Windows 10:**

> **Примечание.** ISO-файл образа, vmguest.iso, больше не требуется для обновления компонентов интеграции. Он не входит в Hyper-V в Windows10.

| Гостевая ОС | Механизм обновления | Примечания |
|:---------|:---------|:---------|
| Windows 10 | Центр обновления Windows | |
| Windows 8.1 | Центр обновления Windows | |
| Windows 8 | Центр обновления Windows | Требуется служба интеграции для обмена данными.* |
| Windows 7 | Центр обновления Windows | Требуется служба интеграции для обмена данными.* |
| WindowsVista с пакетом обновления 2 (SP2) | Центр обновления Windows | Требуется служба интеграции для обмена данными.* |
| - | | |
| Windows Server 2012 R2 | Центр обновления Windows | |
| Windows Server2012 | Центр обновления Windows | Требуется служба интеграции для обмена данными.* |
| Windows Server2008R2 с пакетом обновления 1 (SP1) | Центр обновления Windows | Требуется служба интеграции для обмена данными.* |
| Windows Server2008 с пакетом обновления 2 (SP2) | Центр обновления Windows | Расширенная поддержка предоставляется только по Windows Server2016 ([подробности](https://support.microsoft.com/en-us/lifecycle?p1=12925)). |
| Windows Home Server 2011 | Центр обновления Windows | Не поддерживается в Windows Server2016 ([подробности](https://support.microsoft.com/en-us/lifecycle?p1=15820)). |
| Windows Small Business Server 2011 | Центр обновления Windows | Не в основной фазе поддержки ([подробности](https://support.microsoft.com/en-us/lifecycle?p1=15817)). |
| - | | |
| Гостевые ОС Linux | диспетчер пакетов | Компоненты интеграции для Linux уже встроены в дистрибутив. Могут быть доступны необязательные обновления. ******** |

>  \* Если службу интеграции для обмена данными невозможно включить, компоненты интеграции для этих гостевых ОС можно скачать в виде файла CAB в Центре загрузки [здесь](https://support.microsoft.com/en-us/kb/3071740).  
  Инструкции по применению CAB-файла доступны [здесь](http://blogs.technet.com/b/virtualization/archive/2015/07/24/integration-components-available-for-virtual-machines-not-connected-to-windows-update.aspx).


**Для виртуальных машин, работающих на узлах под управлением Windows 8.1:**

| Гостевая ОС | Механизм обновления | Примечания |
|:---------|:---------|:---------|
| Windows 10 | Центр обновления Windows | |
| Windows 8.1 | Центр обновления Windows | |
| Windows 8 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows 7 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| WindowsVista с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows XP с пакетами обновления 2 и 3 (SP2, SP3) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Windows Server 2012 R2 | Центр обновления Windows | |
| Windows Server2012 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2008 R2 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server2008 с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Home Server 2011 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Small Business Server 2011 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server2003R2 с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2003 с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Гостевые ОС Linux | диспетчер пакетов | Компоненты интеграции для Linux уже встроены в дистрибутив. Могут быть доступны необязательные обновления. ** |


**Для виртуальных машин, работающих на узлах под управлением Windows 8:**

| Гостевая ОС | Механизм обновления | Заметки |
|:---------|:---------|:---------|
| Windows 8.1 | Центр обновления Windows | |
| Windows 8 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows 7 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| WindowsVista с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows XP с пакетами обновления 2 и 3 (SP2, SP3) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Windows Server 2012 R2 | Центр обновления Windows | |
| Windows Server2012 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2008 R2 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4).|
| Windows Server2008 с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Home Server 2011 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Small Business Server 2011 | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server2003R2 с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2003 с пакетом обновления 2 (SP2) | Диск со службами интеграции | Инструкции см. [здесь](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Гостевые ОС Linux | диспетчер пакетов | Компоненты интеграции для Linux уже встроены в дистрибутив. Могут быть доступны необязательные обновления. ** |

 > ** Дополнительные сведения о гостевых ОС Linux см. [здесь](https://technet.microsoft.com/en-us/library/dn531030.aspx). 
