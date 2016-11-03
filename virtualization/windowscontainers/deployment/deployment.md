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
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 1392cb13798c96f91b7daa036e529c97e560edbb

---

# Развертывание узла контейнера — Windows Server

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server 2016 или Windows Server Core 2016 в физической или виртуальной системе.

## Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. 

Для установки Docker будет использоваться [модуль PowerShell поставщика OneGet](https://github.com/oneget/oneget). Поставщик обеспечит работу контейнеров на компьютере и установит Docker. После этого потребуется перезагрузка. 

Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

Сначала будет установлен модуль OneGet PowerShell.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Далее при помощи OneGet будет установлена последняя версия Docker.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

После завершения установки перезагрузите компьютер.

```none
Restart-Computer -Force
```

## Установка базовых образов контейнеров

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве базовой операционной системы. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```none
docker pull microsoft/windowsservercore
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```none
docker pull microsoft/nanoserver
```

> Прочтите лицензионное соглашение для образов ОС контейнеров Windows на странице [Лицензионное соглашение](../Images_EULA.md).

## Узел контейнера Hyper-V

Чтобы запустить контейнеры Hyper-V, необходима роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### Вложенная виртуализация

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

### Включение роли Hyper-V

Чтобы включить компонент Hyper-V с помощью PowerShell, выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=Oct16_HO4-->


