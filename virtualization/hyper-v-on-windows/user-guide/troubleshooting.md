---
title: Устранение неполадок с Hyper-V в Windows 10
description: Устранение неполадок с Hyper-V в Windows 10
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
ms.openlocfilehash: 03bbb4494bbbd790f16c4b6afef387905f7c6c83
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910894"
---
# <a name="troubleshoot-hyper-v-on-windows-10"></a>Устранение неполадок с Hyper-V в Windows 10

## <a name="i-updated-to-windows-10-and-now-i-cant-connect-to-my-downlevel-windows-81-or-server-2012-r2-host"></a>После обновления до Windows 10 не удается подключиться к узлу нижнего уровня (Windows 8.1 или Server 2012 R2)
В Windows 10 диспетчер Hyper-V перемещен в WinRM для удаленного управления.  Это значит, что теперь для управления удаленным узлом Hyper-V с помощью диспетчера Hyper-V на нем необходимо включить удаленное управление.

Дополнительные сведения см. в статье [Управление удаленными узлами Hyper-V с помощью диспетчера Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Remotely-manage-Hyper-V-hosts)

## <a name="i-changed-the-checkpoint-type-but-it-is-still-taking-the-wrong-type-of-checkpoint"></a>Создается неправильный тип контрольной точки даже после его изменения
При создании контрольной точки в программе "Подключение к виртуальной машине" используется тип, который был указан на момент ее открытия, даже если вы изменили его в диспетчере Hyper-V.

Закройте и снова откройте программу "Подключение к виртуальной машине", чтобы она создала правильный тип контрольной точки.

## <a name="when-i-try-to-create-a-virtual-hard-disk-on-a-flash-drive-an-error-message-is-displayed"></a>При попытке создать виртуальный жесткий диск на устройстве флэш-памяти отображается сообщение об ошибке
Hyper-V не поддерживает диски в формате FAT или FAT32, так как эти файловые системы не предоставляют списки управления доступом (ACL) и не поддерживает файлы размером более 4 ГБ. Диски в формате ExFAT имеют ограниченную функциональность ACL, поэтому также не поддерживаются из соображений безопасности.
В PowerShell отображается сообщение об ошибке "Системе не удалось создать "\[путь к VHD\]": запрошенная операция не может быть завершена из-за ограничения файловой системы (0x80070299)".

Используйте диск с файловой системой NTFS. 

## <a name="i-get-this-message-when-i-try-to-install-hyper-v-cannot-be-installed-the-processor-does-not-support-second-level-address-translation-slat"></a>При попытке установки появляется сообщение: "Не удается установить Hyper-V: процессор не поддерживает преобразование адресов второго уровня (SLAT)".
Для запуска виртуальных машин с помощью Hyper-V требуется поддержка SLAT. Если ваш компьютер не поддерживает SLAT, размещение на нем виртуальных машин невозможно.

Если вы просто хотите установить средства управления, снимите флажок **Платформа Hyper-V** в разделе **Программы и компоненты** > **Включение или отключение компонентов Windows**.
