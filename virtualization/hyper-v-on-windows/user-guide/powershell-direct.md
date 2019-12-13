---
title: Управление виртуальными машинами Windows с помощью PowerShell Direct
description: Управление виртуальными машинами Windows с помощью PowerShell Direct
keywords: windows 10, hyper-v, powershell, integration services, службы интеграции, компоненты интеграции, автоматизация, powershell direct
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: ea6b71200d3115ba3d156b2c133e1be2fa495261
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910924"
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>Автоматизация виртуальных машин и управление ими с помощью PowerShell

С помощью PowerShell Direct можно запускать произвольный код PowerShell на виртуальной машине с Windows 10 или Windows Server 2016 с узла Hyper-V независимо от конфигурации сети и параметров удаленного управления.

Ниже приведены некоторые способы запуска PowerShell Direct.

* [Интерактивный сеанс с помощью командлета Enter-PSSession](#create-and-exit-an-interactive-powershell-session)
* [Как раздел с одним использованием для выполнения одной команды или скрипта с помощью командлета Invoke-Command.](#run-a-script-or-command-with-invoke-command)
* [В качестве постоянного сеанса (сборка 14280 и более поздние версии) с помощью командлетов New-PSSession, Copy-Item и Remove-PSSession.](#copy-files-with-new-pssession-and-copy-item)

## <a name="requirements"></a>Требования
**Требования к операционной системе:**
* Узел: Windows 10, Windows Server 2016 или более поздней версии с Hyper-V.
* Гость/виртуальная машина: Windows 10, Windows Server 2016 или более поздней версии.

Если вы управляете виртуальными машинами более ранних версий, используйте средство "Подключение к виртуальной машине" (VMConnect) или [настройте для этой машины виртуальную сеть](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816585(v=ws.10)). 

**Требования к конфигурации:**    
* Виртуальная машина должна работать на узле локально.
* Виртуальная машина должна быть включена и иметь хотя бы один настроенный профиль пользователя.
* Необходимо войти в учетную запись администратора Hyper-V на хост-компьютере.
* Необходимо указать действительные учетные данные пользователя для виртуальной машины.

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>Создание и завершение интерактивного сеанса PowerShell

Для выполнения команд PowerShell на виртуальной машине проще всего запустить интерактивный сеанс.

При запуске сеанса вводимые команды выполняются на виртуальной машине, как если бы вы вводили их непосредственно в сеансе PowerShell на самой виртуальной машине.

**Чтобы запустить интерактивный сеанс, выполните следующие действия.**

1. На узле Hyper-V откройте PowerShell от имени администратора.

2. Выполните одну из указанных ниже команд, чтобы создать интерактивный сеанс, используя имя или GUID виртуальной машины:  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  Укажите учетные данные для виртуальной машины при отображении соответствующего запроса.

3. Выполните команды на виртуальной машине.
  
  Для вашей командной строки PowerShell должен отображаться префикс VMName, как показано ниже:
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  Любая выполненная команда выполняется на виртуальной машине. Для проверки можно выполнить `ipconfig` или `hostname`, чтобы убедиться, что эти команды выполняются на виртуальной машине.
  
4. После завершения работы выполните следующую команду, чтобы закрыть сеанс:  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> Примечание. Если сеанс не подключается, см. раздел [Диагностика](#troubleshooting) для определения возможных причин. 

Дополнительные сведения об этих командлетах см. в разделах [Enter-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Enter-PSSession?view=powershell-5.1) и [Exit-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Exit-PSSession?view=powershell-5.1). 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>Запуск сценария или команды с помощью командлета Invoke-Command

PowerShell Direct с Invoke-Command идеально подходит для ситуаций, когда нужно выполнить одну команду или один сценарий на виртуальной машине, после чего можно прекратить взаимодействие с виртуальной машиной.

**Для выполнения одной команды выполните следующие действия.**

1. На узле Hyper-V откройте PowerShell от имени администратора.

2. Выполните одну из указанных ниже команд, чтобы создать сеанс, используя имя или GUID виртуальной машины:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   Укажите учетные данные для виртуальной машины при отображении соответствующего запроса.
   
   Команда выполняется на виртуальной машине. При наличии выходных данных они выводятся на вашу консоль.  Подключение будет закрыто автоматически сразу после запуска команды.
   
   
**Чтобы выполнить сценарий, выполните следующие действия.**

1. На узле Hyper-V откройте PowerShell от имени администратора.

2. Выполните одну из указанных ниже команд, чтобы создать сеанс, используя имя или GUID виртуальной машины:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   Укажите учетные данные для виртуальной машины при отображении соответствующего запроса.
   
   Сценарий выполняется на виртуальной машине.  Подключение будет закрыто автоматически сразу после запуска команды.

Дополнительные сведения об этом командлете см. в статье [Invoke-Command](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Invoke-Command?view=powershell-5.1). 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>Копирование файлов с помощью New-PSSession и Copy-Item

> **Примечание.** В сборках Windows 14280 и более поздних версий PowerShell Direct поддерживает только постоянные сеансы.

Постоянные сеансы PowerShell чрезвычайно полезны при написании сценариев, координирующих действия для одного или нескольких удаленных компьютеров.  После создания постоянные сеансы выполняются в фоновом режиме, пока вы не решите удалить их.  Это означает, что можно снова и снова ссылаться на один и тот же сеанс с помощью `Invoke-Command` или `Enter-PSSession` без передачи учетных данных.

По той же причине сеансы сохраняют состояние.  Так как постоянные сеансы сохраняются, все созданные или переданные в них переменные сохраняются между вызовами. Существует несколько средств для работы с постоянными сеансами.  В этом примере мы будем использовать [New-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/New-PSSession?view=powershell-5.1) и [Copy-Item](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Management/Copy-Item?view=powershell-5.1) для перемещения данных с узла в виртуальную машину и из виртуальной машины на узел.

**Чтобы создать сеанс, скопируйте файлы:**  

1. На узле Hyper-V откройте PowerShell от имени администратора.

2. Выполните одну из указанных ниже команд, чтобы создать постоянный сеанс PowerShell для виртуальной машины с помощью `New-PSSession`.
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  Укажите учетные данные для виртуальной машины при отображении соответствующего запроса.
  
  > **Предупреждение:**  
   В сборках, предшествующих 14500, имеется ошибка.  Если учетные данные не указаны явно с помощью флага `-Credential`, служба у гостя завершается со сбоем и потребуется перезапустить ее.  Если вы столкнулись с этой проблемой, инструкции по ее устранению см. [здесь](#error-a-remote-session-might-have-ended).
  
3. Скопируйте файл на виртуальную машину.
  
  Чтобы скопировать `C:\host_path\data.txt` на виртуальную машину с хост-компьютера, выполните следующую команду:
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  Скопируйте файл с виртуальной машины (на узел). 
   
   Чтобы скопировать `C:\guest_path\data.txt` на узел с виртуальной машины, выполните следующую команду:
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. Остановите постоянный сеанс с помощью `Remove-PSSession`.
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>Поиск и устранение неисправностей

В PowerShell Direct выводится небольшой набор сообщений о распространенных ошибках.  Ниже приведены наиболее распространенные ошибки, возможные причины и средства диагностики.

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>Параметры -VMName или -VMID не существуют
**Проблема:**  
`Enter-PSSession`, `Invoke-Command`или `New-PSSession` не имеют параметра `-VMName` или `-VMId`.

**Возможные причины:**  
Скорее всего, проблема состоит в том, что PowerShell Direct не поддерживается операционной системой сервера виртуальных машин.

Проверить сборку Windows можно с помощью следующей команды:

``` PowerShell
[System.Environment]::OSVersion.Version
```

Если применяется поддерживаемая сборка, возможно, ваша версия PowerShell не использует PowerShell Direct.  Для PowerShell Direct и JEA требуется основной номер версии 5 или выше.

Проверить версию PowerShell можно с помощью следующей команды:

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>Ошибка: "Возможно, удаленный сеанс был завершен"
> **Примечание.**  
Для Enter-PSSession в сборках узла с 10240 по 12400 все описанные ниже ошибки регистрируются как "Возможно, удаленный сеанс был завершен".

**Сообщение об ошибке:**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Возможные причины:**
* Виртуальная машина существует, но не выполняется.
* Гостевая ОС не поддерживает PowerShell Direct (см. [требования](#requirements)).
* Оболочка PowerShell еще не доступна на гостевой виртуальной машине.
  * Загрузка операционной системы не завершена.
  * Правильная загрузка операционной системы невозможна.
  * Во время загрузки требуется ввод данных пользователем.

Можно использовать командлет [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps), чтобы узнать, какие виртуальные машины выполняются на узле.

**Сообщение об ошибке:**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Возможные причины:**
* Одна из описанных выше причин — все они одинаково применимы к `New-PSSession`  
* Ошибка в текущих сборках, когда требуется явная передача учетных данных с помощью `-Credential`.  Когда это происходит, в операционной системе на виртуальной машине зависает вся служба и ее требуется перезапустить.  Доступность сеанса можно проверить с помощью Enter-PSSession.

Чтобы обойти эту проблему с учетными данными, войдите на виртуальную машину с помощью VMConnect, откройте PowerShell и перезапустите службу vmicvmsession, используя следующую команду PowerShell:

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>Ошибка: набор параметров не может быть разрешен
**Сообщение об ошибке:**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**Возможные причины:**  
* `-RunAsAdministrator` не поддерживается при подключении к виртуальным машинам.
     
  Флаг `-RunAsAdministrator` позволяет администратору подключиться к контейнеру Windows без явного указания учетных данных.  Поскольку виртуальные машины не предоставляют неявный доступ администратора к узлу, в этом случае необходимо явно предоставить учетные данные.

Можно передать учетные данные администратора на виртуальную машину с помощью параметра `-Credential` либо вручную при поступлении запроса.


### <a name="error-the-credential-is-invalid"></a>Ошибка. Недопустимые учетные данные.

**Сообщение об ошибке:**  
```
Enter-PSSession : The credential is invalid.
```

**Возможные причины:** 
* Не удается проверить учетные данные для гостевой виртуальной машины.
  * Предоставлены неправильные учетные данные.
  * Учетные записи пользователей отсутствуют на гостевой виртуальной машине (операционная система не загружалась)
  * В случае подключения от имени администратора: администратор не был установлен как активный пользователь.  Дополнительные сведения см. [здесь](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh825104(v=win.10)>).
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>Ошибка. Входной параметр VMName не разрешается ни в одну виртуальную машину.

**Сообщение об ошибке:**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**Возможные причины:**  
* Вы не являетесь администратором Hyper-V.  
* Виртуальная машина не существует.

С помощью командлета [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) можно проверить, имеется ли у используемых учетных данных роль администратора Hyper-V, а также просмотреть, какие виртуальные машины запущены локально и загружены.


-------------

## <a name="samples-and-user-guides"></a>Примеры и руководства пользователя

PowerShell Direct поддерживает JEA (Just Enough Administration).  Чтобы оценить эту функцию, воспользуйтесь этим руководством пользователя.

См. примеры на веб-сайте [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93).
