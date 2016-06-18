---
title: Развертывание контейнеров Windows в Windows Server
description: Развертывание контейнеров Windows в Windows Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
---

# Развертывание узла контейнера — Windows Server

**Это предварительное содержимое. Возможны изменения.** 

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server 2016 или Windows Server Core 2016 в физической или виртуальной системе.

## Установка компонента контейнеров

Чтобы начать работу с контейнерами Windows, требуется включить контейнер компонентов. Для этого выполните приведенную ниже команду в сеансе PowerShell с повышенными правами. 

```none
Install-WindowsFeature containers
```

После завершения установки компонента перезагрузите компьютер.

## Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента.

Создайте папку для исполняемых файлов Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Скачайте управляющую программу Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Загрузите клиент Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Добавьте каталог Docker в системный путь.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Перезапустите сеанс PowerShell для распознавания измененного пути.

Чтобы установить Docker в качестве службы Windows, выполните следующую команду:

```none
dockerd --register-service
```

После установки эту службу можно запустить.

```none
Start-Service Docker
```

## Установка базовых образов контейнеров

Перед развертыванием контейнера требуется скачать базовый образ ОС контейнера. Приведенный ниже пример кода скачивает базовый образ ОС Windows Server Core. Эту же процедуру можно использовать и для установки базового образа Nano Server. Эту же процедуру можно использовать и для установки базового образа Nano Server. Дополнительные сведения об образах контейнеров Windows см. в статье [Управление образами контейнеров](../management/manage_images.md).
    
Сначала установите поставщик пакетов образов контейнеров.

```none
Install-PackageProvider ContainerImage -Force
```

Затем установите образ Windows Server Core. Этот процесс может занять некоторое время, поэтому сделайте перерыв и возвращайтесь к работе после завершения скачивания.

```none 
Install-ContainerImage -Name WindowsServerCore    
```

После установки базового образа следует перезапустить службу Docker.

```none
Restart-Service docker
```

Далее этот образ следует пометить как последнюю версию "latest". Для этого выполните следующую команду:

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

## Узел контейнера Hyper-V

Чтобы развернуть контейнеры Hyper-V, необходима роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### Вложенная виртуализация

Приведенный ниже сценарий настраивает вложенную виртуализацию для узла контейнера. Он выполняется на компьютере Hyper-V, где размещается виртуальная машина узла контейнера. Перед запуском сценария убедитесь, что виртуальная машина узла контейнера отключена.

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



<!--HONumber=May16_HO4-->


