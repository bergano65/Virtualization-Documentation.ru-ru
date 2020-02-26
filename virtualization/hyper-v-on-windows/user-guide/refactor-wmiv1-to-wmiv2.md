---
title: Перенос программ с Hyper-V WMI версии 1 на WMI версии 2
description: Узнайте, как переносить программы с Hyper-V WMI версии 1 на WMI версии 2
keywords: windows 10, hyper-v, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: 963a1cc356c34c8d051c427a069c49021e3c0d27
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439341"
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>Перенос программ с Hyper-V WMI версии 1 на WMI версии 2

Инструментарий управления Windows (WMI) представляет собой интерфейс управления, лежащий в основе диспетчера Hyper-V и командлетов PowerShell для Hyper-V.  Несмотря на то что большинство пользователей используют наши командлеты PowerShell или диспетчер Hyper-V, иногда разработчикам требуется использовать WMI напрямую.  

Существует два пространства имен WMI Hyper-V (или две версии API-интерфейса WMI Hyper-V).
* Пространство имен WMI версии 1 (root\virtualization), появившееся в Windows Server 2008. Последним продуктом, в котором оно было доступно, была ОС Windows Server 2012
* Пространство имен WMI версии 2 (root\virtualization\v2), которое было представлено в Windows Server 2012

В этом документе содержатся ссылки на ресурсы для преобразования кода, который обеспечивает взаимодействие старого пространства имен WMI с новым.  Прежде всего, эта статья будет являться ресурсом, содержащим сведения об API, а также примеры кода/сценариев, которые можно использовать для переноса любых программ или сценариев с API-интерфейсами WMI Hyper-V из пространства имен версии 1 в пространство имен версии 2.

## <a name="msdn-samples"></a>Примеры MSDN

[Пример миграции виртуальных машин Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Пример виртуального волоконного канала Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Пример запланированных виртуальных машин Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Пример мониторинга работоспособности приложений Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[Пример управления виртуальными жесткими дисками](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Пример репликации Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Пример метрик Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Пример динамической памяти Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Драйвер фильтра расширений расширенного коммутатора Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Пример сети Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Пример управления пулом ресурсов Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Пример моментального снимка восстановления Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>Примеры в блогах

[Добавление сетевого адаптера в виртуальную машину с помощью пространства имен Hyper-V WMI v2](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Подключение сетевого адаптера виртуальной машины к коммутатору с помощью пространства имен Hyper-V WMI v2](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Изменение MAC-адреса сетевого адаптера с помощью пространства имен WMI версии 2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Удаление сетевого адаптера с виртуальной машины с помощью пространства имен Hyper-V WMI v2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Подключение виртуального жесткого диска к виртуальной машине с помощью пространства имен Hyper-V WMI v2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Удаление виртуального жесткого диска из виртуальной машины с помощью пространства имен Hyper-V WMI v2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Создание виртуальной машины с помощью пространства имен Hyper-V WMI v2](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)

