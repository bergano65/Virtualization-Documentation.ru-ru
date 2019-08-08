---
title: Развертывание контейнеров Windows в Windows Server
description: Развертывание контейнеров Windows в Windows Server
keywords: docker, контейнеры
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: e045539b189eb8cd1594da0784ab0c88e848c948
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998781"
---
# <a name="container-host-deployment-windows-server"></a>Развертывание узла контейнера: Windows Server

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server2016 или Windows Server Core2016 в физической или виртуальной системе.

## <a name="install-docker"></a>Установка Docker

Docker необходим для работы с контейнерами Windows. Dock состоит из подсистемы DOCKER и клиента Dock.

Чтобы установить стыковочный элемент, мы будем использовать [модуль PowerShell поставщика онежет](https://github.com/OneGet/MicrosoftDockerProvider). Поставщик включит функцию контейнеров на вашем компьютере и установит закрепление, которое потребуется перезагрузить.

Откройте сеанс PowerShell с повышенными привилегиями и выполните следующие командлеты.

Установите модуль OneGet PowerShell.

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

С помощью OneGet установите последнюю версию Docker.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

После завершения установки перезагрузите компьютер.

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>Установка определенной версии дока

В настоящее время в Docks EE для Windows Server доступно два канала:

* `17.06` -Используйте эту версию, если вы используете закрепление Enterprise Edition (механизм стыковки, точка управления служебной программой, DTR). `17.06` является значением по умолчанию.
* `18.03` -Используйте эту версию, если вы используете только ядро EE.

Чтобы установить определенную версию, используйте `RequiredVersion` флаг:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Для установки конкретных версий Dock для EE может потребоваться обновление ранее установленных модулей Доккермсфтпровидер. Для обновления:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Обновить закрепление

Если вам нужно обновить модуль Dock EE с более ранней версии канала на более поздний, используйте оба `-Update` `-RequiredVersion` флажка.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Установка изображений базового контейнера

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве операционной системы контейнера. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

При выпуске Windows Server 2019 изображения контейнеров Microsoft Sources перемещаются в новый раздел реестра Microsoft Container. Изображения контейнера, опубликованные корпорацией Microsoft, должны быть обнаружены через стыковочный узел. Для новых изображений контейнера, опубликованных в Windows Server 2019 и более поздних, вы должны найти их в мкр. Для старых изображений контейнера, опубликованных до Windows Server 2019, необходимо по-прежнему получать их из реестра Dock.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 и более поздние версии

Чтобы установить базовый образ "Windows Server Core", выполните указанные ниже действия.

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Чтобы установить базовый образ "Nano Server", выполните указанные ниже действия.

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (версии 1607-1803)

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```PowerShell
docker pull microsoft/windowsservercore
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```PowerShell
docker pull microsoft/nanoserver
```

> Пожалуйста, ознакомьтесь с лицензионным соглашением Windows Containers OS, которое можно найти здесь – [EULA](../images-eula.md).

## <a name="hyper-v-isolation-host"></a>Узел изоляции Hyper-V

Для выполнения изоляции Hyper-V необходима роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).

### <a name="nested-virtualization"></a>Вложенная виртуализация

Приведенный ниже сценарий настраивает вложенную виртуализацию для узла контейнера. Этот сценарий выполняется на родительском компьютере Hyper-V. Перед запуском сценария убедитесь, что виртуальная машина узла контейнера отключена.

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Включение роли Hyper-V

Чтобы включить функцию Hyper-V с помощью PowerShell, выполните следующий командлет в сеансе PowerShell с повышенными привилегиями.

```PowerShell
Install-WindowsFeature hyper-v
```
