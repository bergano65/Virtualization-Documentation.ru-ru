---
title: "Управление удаленными узлами Hyper-V с помощью диспетчера Hyper-V"
description: "Управление удаленными узлами Hyper-V с помощью диспетчера Hyper-V"
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 2d34e98c-6134-479b-8000-3eb360b8b8a3
translationtype: Human Translation
ms.sourcegitcommit: eb6c8e904b9cd2c5d1ed25583ffbcdbdf0b9139c
ms.openlocfilehash: fee2b24469b45efae982f4dfca4afb9f737b5bdf

---

# Управление удаленными узлами Hyper-V с помощью диспетчера Hyper-V

Диспетчер Hyper-V — это встроенное средство для управления локальным узлом Hyper-V, а также небольшим количеством удаленных узлов, а также их диагностики.  В этой статье приведены инструкции по подключению к узлам Hyper-V с помощью диспетчера Hyper-V во всех поддерживаемых конфигурациях.

> Диспетчер Hyper-V доступен в разделе **Программы и компоненты** под названием **Средства управления Hyper-V** в [любой ОС Windows с Hyper-V](../quick_start/walkthrough_compatibility.md#OperatingSystemRequirements).  Включать платформу Hyper-V для управления удаленными компьютерами необязательно.

Чтобы подключиться к узлу Hyper-V в диспетчере Hyper-V, убедитесь, что на панели слева выбран **Диспетчер Hyper-V**, и выберите пункт **Подключение к серверу...** на панели справа.

![](media/HyperVManager-ConnectToHost.png)

## Сочетания узлов Hyper-V, поддерживаемые диспетчером Hyper-V
Диспетчер Hyper-V в Windows 10 позволяет управлять следующими узлами Hyper-V:
* Windows 10
* Windows 8.1
* Windows 8
* Windows Server 2016 с Windows Server Core, Nano Server и Hyper-V Server
* Windows Server 2012 R2 с Windows Server Core, Datacenter и Hyper-V Server
* Windows 2012 с Windows Server Core, Datacenter и Hyper-V Server

Диспетчер Hyper-V в Windows 8.1 и Windows Server 2012 R2 позволяет управлять следующими узлами:
* Windows 8.1
* Windows 8
* Windows Server 2012 R2 с Windows Server Core, Datacenter и Hyper-V Server
* Windows 2012 с Windows Server Core, Datacenter и Hyper-V Server

Диспетчер Hyper-V в Windows 8 и Windows Server 2012 позволяет управлять следующими узлами:
* Windows 8
* Windows 2012 с Windows Server Core, Datacenter и Hyper-V Server

Компонент Hyper-V появился в Windows 8.  До Windows 8.1 и Windows Server 2012 диспетчер Hyper-V позволял управлять только соответствующими версиями Hyper-V.

> **Примечание.** Функциональность диспетчера Hyper-V соответствует функциональности управляемой версии.  Другими словами, если вы управляете удаленным узлом Server 2012 из Server 2012 R2, то новые возможности диспетчера Hyper-V версии 2012 R2 будут недоступны.

## Управление локальным узлом ##
Чтобы добавить локальный узел в диспетчер Hyper-V в качестве узла Hyper-V, выберите пункт **Локальный компьютер** в диалоговом окне **Выбор компьютера**.

![](media/HyperVManager-ConnectToLocalHost.png)

Если не удается установить подключение:
*  Убедитесь, что включена роль платформы Hyper-V.  
  Сведения о поддержке Hyper-V см. в [этом разделе пошагового руководства о проверке совместимости](../quick_start/walkthrough_compatibility.md).
*  Убедитесь, что ваша учетная запись пользователя принадлежит к группе администраторов Hyper-V.


## Управление другим узлом Hyper-V в одном домене ##

Чтобы добавить удаленный узел Hyper-V в диспетчер Hyper-V, выберите пункт **Другой компьютер** в диалоговом окне **Выбор компьютера** и в текстовом поле введите имя узла, NetBIOS-имя или полное доменное имя удаленного узла.

![](media/HyperVManager-ConnectToRemoteHost.png)

Чтобы управлять удаленными узлами Hyper-V, необходимо включить удаленное управление на локальном компьютере и удаленном узле.

Для этого выберите пункты `System Properties -> Remote Management Settings` или выполните от имени администратора следующую команду PowerShell:  

``` PowerShell
winrm quickconfig
```

Если ваша текущая учетная запись пользователя соответствует учетной записи администратора Hyper-V на удаленном узле, нажмите кнопку **ОК**, чтобы подключиться.  

> Это единственный способ управления удаленным узлом в диспетчере Hyper-V в Windows 8 или Windows 8.1.


В Windows 10 возможные сочетания типов удаленного подключения значительно расширены.  
Теперь подключиться к удаленному узлу Windows 10 или более поздней версии можно с помощью имени или IP-адреса узла.  Теперь диспетчер Hyper-V поддерживает альтернативные учетные данные пользователя.  


### Подключение к удаленному узлу от имени другого пользователя
> Эта возможность доступна только при подключении к удаленному узлу Windows 10 или Windows Server 2016 Technical Preview 3 или более поздней версии.

В Windows 10, если ваша учетная запись пользователя не позволяет управлять удаленным узлом, вы можете подключиться от имени другого пользователя с альтернативными учетными данными.

Чтобы указать учетные данные для удаленного узла Hyper-V, установите флажок **Подключиться от имени другого пользователя: ** в диалоговом окне **Выбор компьютера**, а затем нажмите кнопку **Выбрать пользователя...**.

![](media/HyperVManager-ConnectToRemoteHostAltCreds.png)


### Подключение к удаленному узлу с помощью IP-адреса
> Эта возможность доступна только при подключении к удаленному узлу Windows 10 или Windows Server 2016 Technical Preview 3 или более поздней версии.

Иногда проще подключиться, используя IP-адрес, а не имя узла.  В Windows 10 это возможно.

Чтобы подключиться с помощью IP-адреса, введите IP-адрес в текстовое поле **Другой компьютер**.


## Управление узлом Hyper-V за пределами домена (или без домена) ##
> Эта возможность доступна только при подключении к удаленному узлу Windows 10 или Windows Server 2016 Technical Preview 3 или более поздней версии.

На управляемом узле Hyper-V выполните следующие команды от имени администратора:

1.  [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx)
  * Команда [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx) создает необходимые правила брандмауэра для *частных* зон сети. Чтобы разрешить такой доступ для общедоступных зон, нужно включить правила для CredSSP и WinRM.
2.  [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role server

На управляющем компьютере запустите следующую команду от имени администратора:

1. Set-Item WSMan:\localhost\Client\TrustedHosts -value "полное_доменное_имя_узла_hyper-v"
  * Кроме того, вы можете разрешить управление всеми узлами с помощью такой команды:
  * Set-Item WSMan:\localhost\Client\TrustedHosts -value * -force
2. [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer "полное_доменное_имя_узла_hyper-v"
  * Кроме того, вы можете разрешить управление всеми узлами с помощью такой команды:
  * [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer *
3. Кроме того, вам может потребоваться настроить следующую групповую политику: ** Конфигурация компьютера | Административные шаблоны | Система | Делегирование учетных данных | Разрешить использование новых учетных данных с проверкой подлинности сервера только для NTLM **
    * Щелкните **Включить** и добавьте *wsman/полное_доменное_имя_узла_hyper-v*
    * Кроме того, вы можете разрешить управление всеми узлами, добавив _wsman/*_



<!--HONumber=Jun16_HO5-->


