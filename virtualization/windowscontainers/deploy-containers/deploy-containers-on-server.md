---
title: Развертывание контейнеров Windows в Windows Server
description: Развертывание контейнеров Windows в Windows Server
keywords: docker, контейнеры
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: b80dd0d231d0f9435b7cc1c5e2b35bbf5a59d793
ms.sourcegitcommit: a287211a0ed9cac7ebfe1718e3a46f0f26fc8843
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/14/2018
ms.locfileid: "2748890"
---
# <a name="container-host-deployment---windows-server"></a>Развертывание узла контейнера— Windows Server

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server2016 или Windows Server Core2016 в физической или виртуальной системе.

## <a name="install-docker"></a>Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. 

Для установки Docker будет использоваться [модуль PowerShell поставщика OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Поставщик обеспечит работу контейнеров на компьютере и установит Docker. После этого потребуется перезагрузка. 

Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

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

## <a name="install-a-specific-version-of-docker"></a>Установка определенной версии Docker

Доступны в настоящее время два канала для EE Docker Windows Server:

* `17.06` -Используйте эту версию, если вы используете Docker Enterprise Edition (Docker модуля, UCP, DTR). `17.06` значение по умолчанию.
* `18.03` -Используйте эту версию, если вы используете модуль EE Docker сам по себе он.

Чтобы установить определенной версии, используйте `RequiredVersion` флаг:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Для установки определенных версий Docker EE может потребоваться обновление ранее установленных модулей DockerMsftProvider. Чтобы обновить:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Обновление Docker

Если вам потребуется обновить модуль EE Docker из более ранних канала к каналу более поздней версии, используйте их вместе `-Update` и `-RequiredVersion` флаги:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Установка базовых образов контейнеров

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве операционной системы контейнера. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```PowerShell
docker pull microsoft/windowsservercore
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```PowerShell
docker pull microsoft/nanoserver
```

> Прочтите лицензионное соглашение для образов ОС контейнеров Windows на странице [Лицензионное соглашение](../images-eula.md).

## <a name="hyper-v-container-host"></a>Узел контейнера Hyper-V

Чтобы запустить контейнеры Hyper-V, необходима роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

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

Чтобы включить компонент Hyper-V с помощью PowerShell, выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```PowerShell
Install-WindowsFeature hyper-v
```
