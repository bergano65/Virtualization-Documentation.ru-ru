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
ms.openlocfilehash: 35f35b490ce5aa80068578d78a6427ace7352b73
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574985"
---
# <a name="container-host-deployment-windows-server"></a>Развертывание узла контейнера: Windows Server

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server2016 или Windows Server Core2016 в физической или виртуальной системе.

## <a name="install-docker"></a>Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker.

Чтобы установить Docker, мы будем использовать [модуль PowerShell поставщика OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Поставщик обеспечит функцию контейнеров на вашем компьютере и установит Docker, после чего потребуется перезагрузка.

Откройте сеанс PowerShell с повышенными привилегиями и выполните приведенные ниже командлеты.

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

## <a name="install-a-specific-version-of-docker"></a>Установите это конкретная версия Docker

Доступны в настоящее время два канала для Docker EE для Windows Server:

* `17.06` -Используйте эту версию, если вы используете Docker Enterprise Edition (подсистема Docker, UCP, DTR). `17.06` значение по умолчанию.
* `18.03` -Используете эту версию, если вы используете подсистема Docker EE само по себе.

Чтобы установить это конкретная версия, используйте `RequiredVersion` флаг:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Установка определенных версий Docker EE может потребоваться обновление ранее установленных модулей DockerMsftProvider. Чтобы обновить:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Обновить Docker

Если необходимо обновить подсистема Docker EE из предыдущих канала по такому каналу, более поздней версии, используйте оба `-Update` и `-RequiredVersion` флаги:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Установка базовых образов контейнеров

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве операционной системы контейнера. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

С выпуском Windows Server 2019 образы контейнеров следы Microsoft переходе на новый реестра называется реестр контейнера Microsoft. Образы контейнеров, выпущенные корпорацией Майкрософт и впредь обнаружения с помощью Docker Hub. Для новых образов контейнеров, публикации с помощью Windows Server 2019 и так далее, вам ознакомиться с для извлечения их из MCR. Для публикации до Windows Server 2019 старые образы контейнера следует продолжать извлекать их из реестра Docker.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 и более поздних версий

Чтобы установить базовый образ «Windows Server Core», выполните следующую команду:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Чтобы установить базовый образ «Nano Server», выполните следующую команду:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (версии 1607 1803)

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```PowerShell
docker pull microsoft/windowsservercore
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```PowerShell
docker pull microsoft/nanoserver
```

> Внимательно прочитайте лицензионное соглашение, можно найти здесь: [Лицензионное соглашение](../images-eula.md)для образа ОС контейнеров Windows.

## <a name="hyper-v-isolation-host"></a>Узел изоляции Hyper-V

Необходимо иметь роль Hyper-V для запуска изоляции Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

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
