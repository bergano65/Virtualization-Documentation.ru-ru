---
title: "Развертывание контейнеров Windows в Windows Server"
description: "Развертывание контейнеров Windows в Windows Server"
keywords: "docker, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: b01c1112ed119908bdabfa0eeee16f4ba11a47f6
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: ru-RU
---
# <a name="container-host-deployment---windows-server"></a>Развертывание узла контейнера— Windows Server

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server2016 или Windows Server Core2016 в физической или виртуальной системе.

## <a name="install-docker"></a>Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. 

Для установки Docker будет использоваться [модуль PowerShell поставщика OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Поставщик обеспечит работу контейнеров на компьютере и установит Docker. После этого потребуется перезагрузка. 

Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

Установите модуль OneGet PowerShell.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

С помощью OneGet установите последнюю версию Docker.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

После завершения установки перезагрузите компьютер.

```none
Restart-Computer -Force
```

## <a name="install-base-container-images"></a>Установка базовых образов контейнеров

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве операционной системы контейнера. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```none
docker pull microsoft/windowsservercore
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```none
docker pull microsoft/nanoserver
```

> Прочтите лицензионное соглашение для образов ОС контейнеров Windows на странице [Лицензионное соглашение](../images-eula.md).

## <a name="hyper-v-container-host"></a>Узел контейнера Hyper-V

Чтобы запустить контейнеры Hyper-V, необходима роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### <a name="nested-virtualization"></a>Вложенная виртуализация

Приведенный ниже сценарий настраивает вложенную виртуализацию для узла контейнера. Этот сценарий выполняется на родительском компьютере Hyper-V. Перед запуском сценария убедитесь, что виртуальная машина узла контейнера отключена.

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Включение роли Hyper-V

Чтобы включить компонент Hyper-V с помощью PowerShell, выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```none
Install-WindowsFeature hyper-v
```
