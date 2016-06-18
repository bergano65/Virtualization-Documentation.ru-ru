---
author: enderb-ms
redirect_url: https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples
---


# Создание образа контейнера .NET 3.5 Server Core

В этом руководстве рассматривается создание контейнера Windows Server Core, включающего платформу .NET 3.5. Для этого упражнения вам потребуется файл ISO образа Windows Server 2016 или доступ к носителю Windows Server 2016.

## Подготовка носителя

Прежде чем создавать образ контейнера с возможностью использования .NET 3.5, пакет .NET 3.5 необходимо подготовить для использования в контейнере. В этом примере файл `Microsoft-windows-netfx3-ondemand-package.cab` будет скопирован с носителя Windows Server 2016 на узел контейнера.

Создайте на этом узле контейнера каталог с именем `dotnet3.5\source`.

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

Скопируйте файл `Microsoft-windows-netfx3-ondemand-package.cab` в этот каталог. Этот файл можно найти в папке sources\sxs носителя Windows Server 2016.

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
``` 
    
Вместо этого, если узел контейнера работает на виртуальной машине Hyper-V и был развернут с помощью сценария быстрого запуска, можно выполнить следующее. Обратите внимание, что указанные действия выполняются на узле Hyper-V, а не на узле контейнера. 

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

Теперь можно создать образ контейнера, включающий платформу .NET 3.5. Это можно выполнить с помощью PowerShell или Docker. Примеры для обоих случаев приведены ниже.

## Создание образа: PowerShell

Чтобы создать новый образ с помощью PowerShell, следует создать контейнер, внести все необходимые изменения и записать его в новый образ.

Создайте новый контейнер из базового образа Windows Server Core.

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName "Virtual Switch"
```

В новом контейнере создайте общую папку. Она нужна, чтобы CAB-файл .NET 3.5 был доступен внутри контейнера.  Обратите внимание, что при выполнении следующей команды контейнер должен быть остановлен.

```powershell
Add-ContainerSharedFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

Запустите контейнер и выполните следующую команду, чтобы установить .NET 3.5.

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

После завершения установки остановите контейнер.

```powershell
Stop-Container dotnet35
```

Чтобы создать образ из этого контейнера, выполните на узле контейнера следующее.

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

Запустите `Get-ContainerImages` для просмотра нового изображения. Теперь с помощью этого образа можно запускать контейнер с предварительно установленной платформой .NET 3.5.

```powershell
Get-ContainerImages
```

## Создание образа: Docker
 
Чтобы создать новый образ с помощью Docker, следует создать файл dockerfile с инструкциями по созданию нового образа. Затем файл dockerfile запускается, при этом создается новый образ контейнера. Обратите внимание, что следующие команды выполняются на виртуальной машине узла контейнера.

Создайте файл dockerfile и откройте его в блокноте.

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

Скопируйте этот текст в файл dockerfile и сохраните его.

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

Выполните команду `docker build`, которая, используя файл dockerfile, создаст новый образ контейнера.

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

Запустите `docker images` для просмотра нового изображения. Теперь с помощью этого образа можно запускать контейнер с предварительно установленной платформой .NET 3.5.

```powershell
docker images
```


<!--HONumber=May16_HO3-->


