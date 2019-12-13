---
title: Развертывание контейнеров Windows в Windows Server
description: Развертывание контейнеров Windows в Windows Server
keywords: docker, контейнеры
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 6e3996af36b4a710f9a12b3a1371138b053a43d8
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909904"
---
# <a name="container-host-deployment-windows-server"></a>Развертывание узла контейнера: Windows Server

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server 2016 или Windows Server Core 2016 в физической или виртуальной системе.

## <a name="install-docker"></a>Установка Docker

Docker необходим для работы с контейнерами Windows. DOCKER состоит из подсистемы DOCKER и клиента DOCKER.

Чтобы установить DOCKER, мы будем использовать [модуль PowerShell поставщика OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Поставщик включит функцию контейнеров на компьютере и установит DOCKER, что потребует перезагрузки.

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

## <a name="install-a-specific-version-of-docker"></a>Установка определенной версии DOCKER

Сейчас для DOCKER EE для Windows Server доступно два канала:

* `17.06` — используйте эту версию, если вы используете DOCKER Enterprise Edition (DOCKER Engine, точка управления служебной программой, DTR). Значение по умолчанию — `17.06`.
* `18.03` — используйте эту версию, если вы используете только ядро DOCKER EE.

Чтобы установить конкретную версию, используйте флаг `RequiredVersion`.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Для установки конкретных версий DOCKER EE может потребоваться обновление ранее установленных модулей Доккермсфтпровидер. Для обновления:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Обновление DOCKER

Если необходимо обновить подсистему DOCKER EE с более раннего канала до более позднего канала, используйте флаги `-Update` и `-RequiredVersion`.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Установка образов базовых контейнеров

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве операционной системы контейнера. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

В выпуске Windows Server 2019 образы контейнеров Microsoft с исходным кодом перемещаются в новый реестр, именуемый реестром контейнеров (Майкрософт). Образы контейнеров, опубликованные корпорацией Майкрософт, продолжают обнаруживаться через центр DOCKER. Для новых образов контейнеров, опубликованных в Windows Server 2019 и более поздних версиях, необходимо извлечь их из мкр. Для старых образов контейнеров, опубликованных до Windows Server 2019, их следует запрашивать из реестра DOCKER.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 и более поздние версии

Чтобы установить базовый образ "Windows Server Core", выполните следующую команду:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (версии 1607-1803)

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> Ознакомьтесь с лицензионным соглашением по образу ОС контейнеров Windows, которое можно найти здесь: [EULA](../images-eula.md).

## <a name="hyper-v-isolation-host"></a>Узел изоляции Hyper-V

Для запуска изоляции Hyper-V необходимо иметь роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).

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

Чтобы включить компонент Hyper-V с помощью PowerShell, выполните следующий командлет в сеансе PowerShell с повышенными привилегиями.

```PowerShell
Install-WindowsFeature hyper-v
```
