---
title: &488045633 Управление удаленными узлами Hyper-V с помощью диспетчера Hyper-V
description: Управление удаленными узлами Hyper-V с помощью диспетчера Hyper-V
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &943375296 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 2d34e98c-6134-479b-8000-3eb360b8b8a3
---

# Управление удаленными узлами Hyper-V с помощью диспетчера Hyper-V

Диспетчер Hyper-V — это встроенное средство для управления локальным узлом Hyper-V, а также небольшим количеством удаленных узлов, а также их диагностики. В этой статье приведены инструкции по подключению к узлам Hyper-V с помощью диспетчера Hyper-V во всех поддерживаемых конфигурациях.

> Диспетчер Hyper-V доступен в разделе <g id="2" ctype="x-strong">Программы и компоненты</g> под названием <g id="4" ctype="x-strong">Средства управления Hyper-V</g> в <g id="6CapsExtId1" ctype="x-link"><g id="6CapsExtId2" ctype="x-linkText">любой ОС Windows с Hyper-V</g><g id="6CapsExtId3" ctype="x-title"></g></g>. Включать платформу Hyper-V для управления удаленными компьютерами необязательно.

Чтобы подключиться к узлу Hyper-V в диспетчере Hyper-V, убедитесь, что на панели слева выбран <g id="2" ctype="x-strong">Диспетчер Hyper-V</g>, и выберите пункт <g id="4" ctype="x-strong">Подключение к серверу...</g> на панели справа.

<g id="1" ctype="x-linkText"></g>

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

Компонент Hyper-V появился в Windows 8. До Windows 8.1 и Windows Server 2012 диспетчер Hyper-V позволял управлять только соответствующими версиями Hyper-V.

> <g id="1" ctype="x-strong">Примечание.</g> Функциональность диспетчера Hyper-V соответствует функциональности управляемой версии. Другими словами, если вы управляете удаленным узлом Server 2012 из Server 2012 R2, то новые возможности диспетчера Hyper-V версии 2012 R2 будут недоступны.

## Управление локальным узлом

Чтобы добавить локальный узел в диспетчер Hyper-V в качестве узла Hyper-V, выберите пункт <g id="2" ctype="x-strong">Локальный компьютер</g> в диалоговом окне <g id="4" ctype="x-strong">Выбор компьютера</g>.

<g id="1" ctype="x-linkText"></g>

Если не удается установить подключение:
*  Убедитесь, что включена роль платформы Hyper-V.  
  Сведения о поддержке Hyper-V см. в <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">разделе руководства о проверке совместимости</g><g id="2CapsExtId3" ctype="x-title"></g></g>.
*  Убедитесь, что ваша учетная запись пользователя принадлежит к группе администраторов Hyper-V.


## Управление другим узлом Hyper-V в одном домене

Чтобы добавить удаленный узел Hyper-V в диспетчер Hyper-V, выберите пункт <g id="2" ctype="x-strong">Другой компьютер</g> в диалоговом окне <g id="4" ctype="x-strong">Выбор компьютера</g> и в текстовом поле введите имя, NetBIOS-имя или полное доменное имя удаленного узла.

<g id="1" ctype="x-linkText"></g>

Чтобы управлять удаленными узлами Hyper-V, необходимо включить удаленное управление на локальном компьютере и удаленном узле.

Для этого выберите пункты <g id="2" ctype="x-code">Свойства системы > Параметры удаленного управления</g> или выполните от имени администратора следующую команду PowerShell:

``` PowerShell
winrm quickconfig
```

Если ваша текущая учетная запись пользователя соответствует учетной записи администратора Hyper-V на удаленном узле, нажмите кнопку <g id="2" ctype="x-strong">ОК</g>, чтобы подключиться.

> Это единственный способ управления удаленным узлом в диспетчере Hyper-V в Windows 8 или Windows 8.1.


В Windows 10 возможные сочетания типов удаленного подключения значительно расширены.  
Теперь подключиться к удаленному узлу Windows 10 или более поздней версии можно с помощью имени или IP-адреса узла. Теперь диспетчер Hyper-V поддерживает альтернативные учетные данные пользователя.


### Подключение к удаленному узлу от имени другого пользователя

> Эта возможность доступна только при подключении к удаленному узлу Windows 10 или Windows Server 2016 Technical Preview 3 или более поздней версии.

В Windows 10, если ваша учетная запись пользователя не позволяет управлять удаленным узлом, вы можете подключиться от имени другого пользователя с альтернативными учетными данными.

Чтобы указать учетные данные для удаленного узла Hyper-V, установите флажок <g id="2" ctype="x-strong">Подключиться от имени другого пользователя: ** в диалоговом окне **Выбор компьютера</g>, а затем нажмите кнопку <g id="4" ctype="x-strong">Выбрать пользователя...</g>

<g id="1" ctype="x-linkText"></g>


### Подключение к удаленному узлу с помощью IP-адреса

> Эта возможность доступна только при подключении к удаленному узлу Windows 10 или Windows Server 2016 Technical Preview 3 или более поздней версии.

Иногда проще подключиться, используя IP-адрес, а не имя узла. В Windows 10 это возможно.

Чтобы подключиться с помощью IP-адреса, введите IP-адрес в текстовое поле <g id="2" ctype="x-strong">Другой компьютер</g>.


## Управление узлом Hyper-V за пределами домена (или без домена)

> Эта возможность доступна только при подключении к удаленному узлу Windows 10 или Windows Server 2016 Technical Preview 3 или более поздней версии.

На управляемом узле Hyper-V выполните следующие команды от имени администратора:

1.  <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-PSRemoting</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Команда <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-PSRemoting</g><g id="1CapsExtId3" ctype="x-title"></g></g> создает необходимые правила брандмауэра для <g id="3" ctype="x-em">частных</g> зон сети. Чтобы разрешить такой доступ для общедоступных зон, нужно включить правила для CredSSP и WinRM.
2. Set-Item WSMan:\localhost\Client\TrustedHosts -value "fqdn-of-managing-pc"
  * Кроме того, вы можете разрешить управление всеми узлами с помощью такой команды:
  * Set-Item WSMan:\localhost\Client\TrustedHosts -value * -force
3. <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-WSManCredSSP</g><g id="1CapsExtId3" ctype="x-title"></g></g> -Role client -DelegateComputer "fqdn-of-managing-pc"
  * Кроме того, вы можете разрешить управление всеми узлами с помощью такой команды:
  * <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-WSManCredSSP</g><g id="1CapsExtId3" ctype="x-title"></g></g> -Role client -DelegateComputer *






<!--HONumber=May16_HO1-->


