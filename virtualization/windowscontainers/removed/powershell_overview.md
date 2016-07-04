---
author: scooley
translationtype: Human Translation
ms.sourcegitcommit: af065ec180f1b5de9e40ef269e7278a16b0c3b7f
ms.openlocfilehash: 5463412d44bd7c657401c55558bb817df4cc1eb2

---

# PowerShell для контейнеров

## Install-ContainerOSImage

**ИМЯ**      
Install-ContainerOSImage

**КРАТКИЙ ОБЗОР**  
Устанавливает WIM-файл как образ ОС контейнера для использования с контейнерами Windows Server или Hyper-V.


**SYNTAX**  
``` PowerShell  
Install-ContainerOSImage [-WimPath] <String> [-Force] [< CommonParameters >]
```

**ОПИСАНИЕ**  
Устанавливает основной образ из WIM-файла в общее центральное хранилище образов для контейнеров Windows Server и Hyper-V.

**PARAMETERS**
``` PowerShell
    -WimPath <String>
        A path to the WIM file that will be installed.

        Required?                    true
        Position?                    1
        Default value
        Accept pipeline input?       false
        Accept wildcard characters?  false

    -Force [<SwitchParameter>]

        Required?                    false
        Position?                    named
        Default value                False
        Accept pipeline input?       false
        Accept wildcard characters?  false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**      
Нет

**ВЫХОДНЫЕ ДАННЫЕ**  
Нет

**ПСЕВДОНИМЫ**  
Нет

-------------------------- ПРИМЕР 1 --------------------------

``` PowerShell
PS C:\>Install-ContainerOSImage c:\baseimage.wim
```

## Uninstall-ContainerOSImage

**ИМЯ**  
Uninstall-ContainerOSImage

**КРАТКИЙ ОБЗОР**  
Удаляет ранее установленный образ ОС контейнера.

**SYNTAX**   
```PowerShell
Uninstall-ContainerOSImage [-ImageName] <string> [-Force]  [< CommonParameters >]

Uninstall-ContainerOSImage [-ContainerImage] <Object> [-Force]  [< CommonParameters >]
```

**PARAMETERS**  
``` PowerShell
    -ContainerImage <Object>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           ByContainerImage
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ImageName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           ByName
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Нет


**ВЫХОДНЫЕ ДАННЫЕ**  
System.Object

**ПСЕВДОНИМЫ**  
Нет

## Add-ContainerNetworkAdapter ##

**ИМЯ**  
Add-ContainerNetworkAdapter

**КРАТКИЙ ОБЗОР**  
Добавляет новый сетевой адаптер в существующий контейнер.

**SYNTAX** 
``` PowerShell  
Add-ContainerNetworkAdapter [-ContainerName] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-SwitchName <string>] [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Add-ContainerNetworkAdapter [-Container] <Container[]> [-SwitchName <string>] [-Name <string>]
    [-DynamicMacAddress] [-StaticMacAddress <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -DynamicMacAddress

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -StaticMacAddress <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```


**ВХОДНЫЕ ДАННЫЕ**  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ПСЕВДОНИМЫ**  
Нет

## Connect-ContainerNetworkAdapter

**ИМЯ**  
Connect-ContainerNetworkAdapter

**КРАТКИЙ ОБЗОР**  
Подключает сетевой адаптер контейнера к виртуальному коммутатору.

**SYNTAX**  
``` PowerShell
    Connect-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string[]>] [-SwitchName] <string>
    [-Passthru] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    Connect-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-SwitchName] <string> [-Passthru]
    [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Object_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Object_SwitchName, Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ПСЕВДОНИМЫ**  
Нет

## Disconnect-ContainerNetworkAdapter

**ИМЯ**  
Disconnect-ContainerNetworkAdapter

**КРАТКИЙ ОБЗОР**  
Отключает сетевой адаптер контейнера от виртуального коммутатора.

**SYNTAX**  
``` PowerShell
    Disconnect-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string[]>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Disconnect-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ПСЕВДОНИМЫ**  
Нет

## Export-ContainerImage

**ИМЯ**  
Export-ContainerImage

**КРАТКИЙ ОБЗОР**  
Копирует образ контейнера из локального хранилища.

**SYNTAX**  
``` PowerShell
    Export-ContainerImage [[-Name] <string>] [-Path] <string> [[-Version] <version>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Export-ContainerImage [-Image] <ContainerImage[]> [-Path] <string> [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARAMETERS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ПСЕВДОНИМЫ**  
Нет

## Get-Container

**ИМЯ**  
Get-Container

**КРАТКИЙ ОБЗОР**  
Перечисляет контейнеры в текущей системе.

**SYNTAX**  
``` PowerShell
    Get-Container [[-Name] <string[]>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>]  [<CommonParameters>]

    Get-Container [[-Id] <guid>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Id, Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Id, Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name, Id
        Aliases                      None
        Dynamic?                     false

    -Id <guid>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue, ByPropertyName)
        Parameter set name           Id
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
System.String\[\]  
System.Guid


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container


**ПСЕВДОНИМЫ**  
Нет

## Get-ContainerHost

**ИМЯ**  
Get-ContainerHost

**КРАТКИЙ ОБЗОР**  
Возвращает объект узла контейнера

**SYNTAX**  
``` PowerShell
    Get-ContainerHost [[-ComputerName] <string[]>] [[-Credential] <pscredential[]>]  [<CommonParameters>]

    Get-ContainerHost [-CimSession] <CimSession[]>  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByPropertyName)
        Parameter set name           CimSession
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ComputerName
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       true (ByValue)
        Parameter set name           ComputerName
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Management.Infrastructure.CimSession\[\]  
System.String\[\]  
System.Management.Automation.PSCredential\[]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerHost


**ПСЕВДОНИМЫ**  
Нет

## Get-ContainerImage

**ИМЯ**  
Get-ContainerImage

**КРАТКИЙ ОБЗОР**  
Отображает список образов контейнеров на узле контейнера.

**SYNTAX**  
``` PowerShell
Get-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-ChildOf <ContainerImage>]
[-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -ChildOf <ContainerImage>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Нет


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ПСЕВДОНИМЫ**  
Нет

## Get-ContainerNetworkAdapter

**ИМЯ**  
Get-ContainerNetworkAdapter

**КРАТКИЙ ОБЗОР**  
Отображает список сетевых адаптеров, связанных с контейнером.

**SYNTAX**  
``` PowerShell
    Get-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>]  [<CommonParameters>]

    Get-ContainerNetworkAdapter [-Container] <Container[]> [[-Name] <string>]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]  


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter  


**ПСЕВДОНИМЫ**  
Нет  

## Import-ContainerImage

**ИМЯ**  
Import-ContainerImage

**КРАТКИЙ ОБЗОР**  
Импортирует образ контейнера, экспортированный с другого компьютера.

**SYNTAX**  
``` PowerShell
    Import-ContainerImage [-Path] <string> [-AsJob] [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
System.String


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ПСЕВДОНИМЫ**  
Нет

## Move-ContainerImageRepository

**ИМЯ**  
Move-ContainerImageRepository

**КРАТКИЙ ОБЗОР**  
Изменяет место, где хранятся образы контейнеров.  Это должно быть место на локальном диске.  Его можно изменить только при отсутствии образов в системе.

**SYNTAX**  
``` PowerShell
    Move-ContainerImageRepository [-Path] <string> [-AsJob] [-Passthru] [-CimSession <CimSession[]>] [-ComputerName
    <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Нет


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.HyperV.PowerShell.VMHost


**ПСЕВДОНИМЫ** — нет

## New-Container

**ИМЯ**  
New-Container

**КРАТКИЙ ОБЗОР**  
Создает контейнер.

**SYNTAX**  
``` PowerShell
    New-Container [[-Name] <string>] -ContainerImageName <string> [-ContainerImagePublisher <string>]
    [-ContainerImageVersion <version>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-MemoryStartupBytes <long>] [-SwitchName <string>] [-Path <string>] [-AsJob] [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    New-Container [[-Name] <string>] -ContainerImage <ContainerImage> [-MemoryStartupBytes <long>] [-SwitchName
    <string>] [-Path <string>] [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerImage <ContainerImage>

        Required?                    true
        Position?                    Named
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -ContainerImageName <string>

        Required?                    true
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ContainerImagePublisher <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ContainerImageVersion <version>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -MemoryStartupBytes <long>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers, Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container


**ПСЕВДОНИМЫ**  
Нет

## New-ContainerImage

**ИМЯ**  
New-ContainerImage

**КРАТКИЙ ОБЗОР**  
Создает образ контейнера на основе существующего контейнера.

**SYNTAX**  
``` PowerShell
    New-ContainerImage [-ContainerName] <string> [-Name] <string> [-Publisher] <string> [-Version] <version>
    [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    New-ContainerImage [-Container] <Container> [-Name] <string> [-Publisher] <string> [-Version] <version> [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    New-ContainerImage [-ContainerId] <guid> [-Name] <string> [-Publisher] <string> [-Version] <version> [-CimSession
    <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Id, Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Id
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerId <guid>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Id
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Id
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    true
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    true
        Position?                    3
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ПСЕВДОНИМЫ**  
Нет

## Remove-Container

**ИМЯ**  
Remove-Container

**КРАТКИЙ ОБЗОР**  
Удаляет существующий контейнер из системы.

**SYNTAX**  
``` PowerShell
    Remove-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-Force] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Remove-Container [-Container] <Container[]> [-Force] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container


**ПСЕВДОНИМЫ**  
Нет

## Remove-ContainerImage

**ИМЯ**  
Remove-ContainerImage

**КРАТКИЙ ОБЗОР**  
Удаляет образ контейнера из узла контейнера.

**SYNTAX**  
``` PowerShell
    Remove-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-CimSession
    <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-Force] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Remove-ContainerImage [-Image] <ContainerImage> [-Force] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ВЫХОДНЫЕ ДАННЫЕ**  
System.Object

**ПСЕВДОНИМЫ**  
Нет

## Remove-ContainerNetworkAdapter

**ИМЯ**  
Remove-ContainerNetworkAdapter

**КРАТКИЙ ОБЗОР**  
Удаляет сетевой адаптер из контейнера.

**SYNTAX**  
``` PowerShell
    Remove-ContainerNetworkAdapter [-ContainerName] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-Name <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Remove-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Remove-ContainerNetworkAdapter [-Container] <Container[]> [-Name <string>] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Object
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ПСЕВДОНИМЫ**  
Нет

## Set-ContainerNetworkAdapter

**ИМЯ**  
Set-ContainerNetworkAdapter

**КРАТКИЙ ОБЗОР**  
Задает MAC-адрес сетевого адаптера в контейнере.

**SYNTAX**  
``` PowerShell
    Set-ContainerNetworkAdapter [-ContainerName] <string> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress <string>] [-Passthru]
    [-WhatIf] [-Confirm]  [<CommonParameters>]

    Set-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter> [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Set-ContainerNetworkAdapter [-Container] <Container> [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -DynamicMacAddress

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           ResourceObject, Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -StaticMacAddress <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           ResourceObject, Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
System.String  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter  
Microsoft.Containers.PowerShell.Objects.Container


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ПСЕВДОНИМЫ**  
Нет

## Start-Container

**ИМЯ**  
Start-Container

**КРАТКИЙ ОБЗОР**  
Запускает контейнер.

**SYNTAX**  
``` PowerShell
    Start-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Start-Container [-Container] <Container[]> [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container


**ПСЕВДОНИМЫ**  
Нет

## Stop-Container

**ИМЯ**  
Stop-Container

**КРАТКИЙ ОБЗОР**  
Прекращает работу контейнера.

**SYNTAX**  
``` PowerShell
    Stop-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-TurnOff] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Stop-Container [-Container] <Container[]> [-TurnOff] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -TurnOff

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.Container


**ПСЕВДОНИМЫ**  
Нет

## Test-ContainerImage

**ИМЯ**  
Test-ContainerImage

**КРАТКИЙ ОБЗОР**  
Проверяет образ контейнера в системе узла контейнера.

**SYNTAX**  
``` PowerShell
    Test-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Test-ContainerImage [-Image] <ContainerImage> [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARAMETERS**  
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ВХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ВЫХОДНЫЕ ДАННЫЕ**  
Microsoft.Containers.PowerShell.Objects.ContainerImageReport


**ПСЕВДОНИМЫ**  
Нет


<!--HONumber=Jun16_HO4-->


