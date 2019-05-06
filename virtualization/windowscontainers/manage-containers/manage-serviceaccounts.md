---
title: Групповые управляемые службы учетных записей для контейнеров Windows
description: Групповые управляемые службы учетных записей для контейнеров Windows
keywords: docker, контейнеры, active directory, gmsa
author: rpsqrd
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: a2fc3b74a7be109caf078553e471d1c3743f217a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610314"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Групповые управляемые службы учетных записей для контейнеров Windows

Сетей на основе Windows обычно используют Active Directory (AD) для упрощения проверки подлинности и авторизации пользователей, компьютеров и другим сетевым ресурсам. Корпоративные приложения разработчики часто проектирования AD-интегрированные и выполнения на серверах, присоединенный к домену, чтобы воспользоваться преимуществами проверки подлинности Windows, который облегчает пользователям и другим службам автоматически и прозрачно войти в свои приложения приложение со своими учетными данными.

Несмотря на то, что контейнеры Windows невозможно присоединить к домену, они могут по-прежнему использовать удостоверения домена Active Directory для поддержки различных сценариях проверки подлинности.

Чтобы добиться этого, можно настроить контейнера Windows для запуска с [групповой управляемой учетной записью службы](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA), который — это специальный тип учетной записи службы, введены в Windows Server 2012 для нескольких компьютеров к общему идентификатору без необходимости Чтобы узнать свой пароль.

При запуске контейнера с gMSA узла контейнера извлекает gMSA пароль от контроллера домена Active Directory и переводит на него экземпляр контейнера. Контейнер будет использовать учетные данные gMSA всякий раз, когда его учетная запись компьютера (SYSTEM) необходим доступ к сетевым ресурсам.

В этой статье объясняется, как начать работу с Active Directory групповых управляемых учетных записей служб с контейнерами Windows.

## <a name="prerequisites"></a>Что вам понадобится

Для запуска контейнера Windows с помощью групповой управляемой учетной записи службы, необходимо следующее:

- Домен Active Directory с помощью по крайней мере один контроллер домена под управлением Windows Server 2012 или более поздней версии. Нет леса или домена функционального уровня требований для использования gMSAs, но пароли gMSA может быть только распределенных контроллерами домена под управлением Windows Server 2012 или более поздней версии. Дополнительные сведения см. в разделе [требования к Active Directory для gMSAs](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Разрешение на создание управляемой учетной записи. Создание управляемой учетной записи службы, необходимо иметь права администратора домена или использовать учетную запись, которая была делегированные разрешения *Создание msDS-GroupManagedServiceAccount объектов* .
- Доступ к Интернету, чтобы загрузить модуль credentialspec среды PowerShell. Если вы работаете в изолированной среде, вы можете [Сохранить модуль](https://docs.microsoft.com/en-us/powershell/module/powershellget/save-module?view=powershell-5.1) на компьютере с internet доступ и скопировать его в узел компьютера или контейнер разработки.

## <a name="one-time-preparation-of-active-directory"></a>Одноразовый подготовки Active Directory

Если вы еще не создали gMSA в вашем домене, необходимо создать корневой раздел ключей ((kds) — служба распространения на ключ). Ключей (kds) отвечает за создание, поворот и освобождение пароль gMSA, авторизованных узлов. Когда узел контейнеров требуется использование для запуска контейнера, он будет связываться с ключей (kds), чтобы получить текущий пароль.

Как проверить наличие ключей (kds) корневой ключ уже создан, выполните следующий командлет PowerShell от имени администратора домена на контроллере домена или член домена и средствами AD PowerShell:

```powershell
Get-KdsRootKey
```

Если команда вернет ключ идентификатора, вы все набор и можно пропустить раздел [Создание групповой управляемой учетной записью службы](#create-a-group-managed-service-account) . В противном случае выполните на создание корневого ключа ключей (kds).

В рабочей среде или тестовой среде с несколькими контроллерами домена выполните следующий командлет в PowerShell учетной записью администратора домена на создание корневого ключа ключей (kds).

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Несмотря на то, что команда означает, что ключ вступит в силу немедленно, необходимо будет 10 часов, прежде чем корневой раздел ключей (kds) репликации и доступны для использования на всех контроллерах домена.

Если имеется только один контроллер домена в домене, можно ускорить процесс, установив ключ для эффективной 10 часов назад.

>[!IMPORTANT]
>Не используйте этот метод в рабочей среде.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Создание группы управляемый учетной записи службы

Каждый контейнер, который использует встроенную проверку подлинности Windows требуется по крайней мере один gMSA. Основной gMSA будет использоваться для всех приложений, работающих как System или Network Service доступ к ресурсам в сети. Имя gMSA станет имя контейнера в сети, независимо от того, имя узла, назначенные в контейнер. Контейнеров можно также настроить с помощью дополнительных gMSAs на случай, если вы хотите запустить службы или приложения в контейнере как разные учетные записи из учетной записи компьютера контейнера.

При создании gMSA, можно также создать общего удостоверение, которое можно использовать одновременно на нескольких разных компьютерах. Доступ к паролю gMSA защищен системой список управления доступом Active Directory. Рекомендуется создать группу безопасности для каждой учетной записи gMSA и добавить соответствующие контейнера узлов в группу безопасности для ограничения доступа к паролю.

Наконец так как контейнеры не автоматически регистрироваться все имена (Участника службы), необходимо вручную создать по крайней мере узла SPN для учетной записи gMSA.

Как правило узла или http SPN зарегистрировано имя совпадает с именем учетной записи gMSA, но вам может потребоваться использовать другое имя службы, если клиенты получают доступ к контейнерного приложения из-за балансировщика нагрузки или DNS-имя, которое отличается от имени gMSA.

Например, если учетной записи gMSA называется «WebApp01», но пользователям доступ к узлу в `mysite.contoso.com`, вам следует зарегистрировать `http/mysite.contoso.com` SPN для учетной записи gMSA.

Некоторые приложения могут потребоваться дополнительные SPN для их уникальные протоколы. Например, SQL Server требуется `MSSQLSvc/hostname` поставщика услуг.

В следующей таблице перечислены необходимые атрибуты для создания gMSA.

|Свойство gMSA | Требуемое значение | Пример. |
|--------------|----------------|--------|
|ФИО | Любое имя действительной учетной записи. | `WebApp01` |
|DnsHostName | Имя домена, к имени учетной записи. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Задать по крайней мере основное имя поставщика услуг, добавить другие протоколы при необходимости. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | В группе безопасности, содержащий узлов контейнера. | `WebApp01Hosts` |

После того как вы решили по имени для вашего gMSA, выполните приведенные ниже командлеты в PowerShell, чтобы создать группу безопасности и gMSA.

> [!TIP]
> Вам потребуется использовать учетной записи, которая входит в группу безопасности **Администраторов домена** или была делегированные разрешения **msDS-GroupManagedServiceAccount создание объектов** на выполните следующие команды.
> Командлет [New-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) является частью средства AD PowerShell из [Средства удаленного администрирования сервера](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

Мы рекомендуем создать учетные записи отдельных gMSA для разработки, тестирования и рабочей сред.

## <a name="prepare-your-container-host"></a>Подготовка узла контейнера

Каждый узел контейнера будет работать контейнера Windows с помощью gMSA должен быть подключено к домену и иметь доступ к получить gMSA пароль.

1. Присоединение компьютера к домену Active Directory.
2. Убедитесь, что ваш узел входит в группу безопасности управления доступом к gMSA пароль.
3. Перезагрузите компьютер, поэтому он получает его новый членство в группе.
4. Настройка [Docker рабочего стола для Windows 10](https://docs.docker.com/docker-for-windows/install/) или [Docker для Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5. (Рекомендуется) Убедитесь, что основное приложение может использовать учетной записи gMSA, выполнив [Тест-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount). Если команда возвращает **значение False**, смотрите в разделе " [Устранение неполадок](#troubleshooting) " для диагностики действия.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Создание спецификации учетных данных

Файл спецификации учетных данных — это документ JSON, содержащий метаданные о учетным записям gMSA, что вы хотите использовать контейнер. Сохраняя конфигурации удостоверение отдельно от образа контейнера, вы можете изменить какие gMSA по просто учетных данных спецификации файла подкачки, код не изменяет необходимые использует контейнера.

Будет создан файл спецификации учетных данных, с помощью [credentialspec среды PowerShell](https://aka.ms/credspec) на узле контейнера присоединенный к домену.
После создания файла, вы можете скопировать других узлах контейнеров или вашей оркестратор контейнера.
Спецификации файла с учетными данными не содержит все конфиденциальных данных, таких как пароль gMSA, так как узел контейнера извлекает gMSA от имени контейнера.

Docker ожидает найти файл спецификации учетных данных в каталоге **CredentialSpecs** в каталоге Docker данных. По умолчанию при установке, вы найдете эту папку в `C:\ProgramData\Docker\CredentialSpecs`.

Чтобы создать файл спецификации учетных данных на узле контейнера:

1. Установите средства удаленного администрирования сервера AD PowerShell
    - Windows Server, запустите **Установки-WindowsFeature RSAT-AD-PowerShell**.
    - Windows 10, версия 1809 или более поздней версии, запустите **установки WindowsCapability-сети "Rsat.ActiveDirectory.DS-LDS.Tools~~~0.0.1.0"**.
    - Для более ранних версий Windows 10, см. в разделе <https://aka.ms/rsat>.
2. Выполните следующий командлет установить последнюю версию [credentialspec среды PowerShell](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    При отсутствии доступа к Интернету на узле контейнера, запустите `Save-Module CredentialSpec` на компьютере, подключенных к Интернету и скопируйте папку модуля для `C:\Program Files\WindowsPowerShell\Modules` или в другом месте в `$env:PSModulePath` на узле контейнера.

3. Выполните следующий командлет для создания нового файла спецификации учетных данных:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    По умолчанию командлет создает спецификацию cred, предоставленный gMSA имя с учетную запись компьютера для контейнера. Файл будет сохранен в каталоге Docker CredentialSpecs, используя имя домена или учетной записи gMSA для имени файла.

    Вы можете создать спецификацию учетных данных, содержащее дополнительные gMSA организации, если вы используете службы или процесса управляемой вспомогательной в контейнере. Чтобы сделать это, используйте `-AdditionalAccounts` параметр:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Полный список поддерживаемых параметров, запустите `Get-Help New-CredentialSpec`.

4. Можно отобразить список всех спецификации учетных данных и их полный путь, с помощью следующего командлета:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>Настройте приложение на использование

В обычной конфигурации контейнера предоставляется только одной учетной записи gMSA, который используется всякий раз, когда учетная запись компьютера контейнера пытается выполнить проверку подлинности к сетевым ресурсам. Это означает, что ваше приложение необходимо будет запустить как **Local System** или **Network Service** , если оно должно использовать удостоверение gMSA.

### <a name="run-an-iis-app-pool-as-network-service"></a>Запустить пул приложений IIS как сетевая служба

Если вы размещаете веб-сайт IIS в контейнер, все, что нужно сделать для использования в gMSA задается ваше удостоверение пула приложений к **Сетевой службе**. Что можно сделать в вашем файле Dockerfile, добавив следующую команду:

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

Если ранее вы использовали статические учетные данные для пула приложений IIS, рассмотрите возможность gMSA качестве замены эти учетные данные. Вы можете изменить gMSA разработки, тестирования и рабочей средами и IIS автоматически получают текущего удостоверения без изменения образа контейнера.

### <a name="run-a-windows-service-as-network-service"></a>Запуск службы Windows как сетевая служба

Если контейнерного приложения выполняется как служба Windows, вы можете настроить службу для запуска в качестве **Сетевой службы** в вашем Dockerfile:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>Запускать произвольный консольные приложения как сетевая служба

Для универсального консольные приложения, которые не размещаются в или диспетчер служб IIS часто бывает проще всего запустить контейнер как **Сетевая служба** , чтобы приложение автоматически наследует контекст gMSA. Эта функция доступна начиная с Windows Server версии 1709.

Добавьте следующую строку для вашего файла Dockerfile, чтобы она выполнялась как сетевая служба по умолчанию:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Можно также подключаться к контейнеру как сетевая служба на основе однократных с `docker exec`. Это особенно полезно, если Устранение неполадок подключения в работающем контейнере, когда контейнер не обычно выполняется как сетевая служба.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>Запуска контейнера с gMSA

Для запуска контейнера с gMSA, предоставить файле спецификации учетных данных для `--security-opt` параметр [запуска docker](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>В версиях Windows Server 2016 1709 и 1803 имя узла контейнера должна соответствовать gMSA короткое имя.

В предыдущем примере gMSA имя учетной записи SAM — «webapp01», поэтому имя узла контейнера также называется «webapp01».

В Windows Server 2019 и более поздних версиях поле имени узла не является обязательным, но контейнер будет по-прежнему идентифицировать себя имя gMSA вместо имени сервера, даже если явным образом указать другое.

Чтобы проверить gMSA, правильно ли работает, выполните следующий командлет в контейнере:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Если доверенного состояния подключения контроллера домена и доверять состояние проверки не `NERR_Success`, обратитесь к разделу [Устранение неполадок](#troubleshooting) советы о том, как выполнить отладку проблемы.

Вы можете проверить удостоверение gMSA из контейнера, выполнив следующую команду и проверьте имя клиента:

```powershell
PS C:\> klist get krbtgt

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

Чтобы открыть PowerShell или другое приложение консоли в качестве учетной записи gMSA, можно задать контейнер для запуска учетную запись вместо обычного ContainerAdministrator (или ContainerUser для NanoServer) сетевая служба:

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Если вы используете как сетевая служба, вы можете тестировать проверку подлинности на сетевом управляемой учетной записи, пытаясь подключиться к SYSVOL на контроллере домена:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>Планировать контейнеры с gMSA

В производственной среде вы часто используете оркестратора контейнеров для развертывания и управления приложениями и службами. Каждый orchestrator имеет собственный парадигм управления и отвечает за принятие спецификации учетных данных для предоставления к платформе контейнеров Windows.

Если Вы управляющий контейнеры с gMSAs, убедитесь, что:

> [!div class="checklist"]
> * Все узлы контейнеров, которые могут быть запланированы для запуска контейнеров с gMSAs подключены к домену
> * Контейнера узлов есть доступ получить пароли для всех gMSAs, используемые контейнеров
> * Файлы спецификации учетных данных создаются и загруженные оркестратора или скопирована в каждом узле контейнера, в зависимости от того, как оркестратора предпочитает их обработки.
> * Разрешить контейнеров для связи с контроллерами домена Active Directory для получения запросов в службу gMSA сетей контейнера

### <a name="how-to-use-gmsa-with-service-fabric"></a>Как на использование с использованием Service Fabric

Service Fabric поддерживает контейнеров под управлением Windows с помощью gMSA, при указании спецификации расположение учетных данных в манифесте приложения. Необходимо создать файл спецификации учетных данных и поместить в подкаталог **CredentialSpecs** каталога данных Docker на каждом узле таким образом, чтобы Service Fabric можно найти его. Можно запустить командлет **Get-CredentialSpec** частью [credentialspec среды PowerShell](https://aka.ms/credspec), необходимо убедиться, если ваш спецификации учетных данных в нужное место.

См. в разделе [Краткое руководство: развертывание контейнеров Windows на Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-quickstart-containers) и [Настройка gMSA для контейнеров Windows, работающих на Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) Дополнительные сведения о том, как настроить приложение.

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Использование gMSA с Docker Swarm

На использование с контейнерами, управляется режима мелких объектов Docker, запустите команду [создать службу docker](https://docs.docker.com/engine/reference/commandline/service_create/) с помощью `--credential-spec` параметр:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

См. в [примере режима мелких объектов Docker](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) Дополнительные сведения о том, как использовать спецификации учетных данных с помощью службы Docker.

### <a name="how-to-use-gmsa-with-kubernetes"></a>Использование gMSA с Kubernetes

Поддержка для планирования контейнеры Windows с помощью gMSAs в Kubernetes доступна как альфа-компонент в Kubernetes 1.14. См. в разделе [Настройка gMSA POD в Windows и контейнерах](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) последние сведения об этой функции и проверка его в дистрибутива Kubernetes.

## <a name="example-uses"></a>Примеры использования

### <a name="sql-connection-strings"></a>Строки подключения SQL

Если служба выполняется в контейнере как Local System или Network Service, она может использовать встроенную проверку подлинности Windows для подключения к Microsoft SQL Server.

Вот пример строки подключения, которое использует идентификатор контейнера для проверки подлинности в SQL Server:

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

На Microsoft SQL Server создайте имя для входа с использованием домена и имени групповой управляемой учетной записи службы, после которых указан символ "$". После создания имени для входа, можно добавить его пользователям в базе данных и присвойте ему нужные разрешения на доступ.

Пример.

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

Чтобы увидеть это в действии, ознакомьтесь с [запись демонстрации](https://youtu.be/cZHPz80I-3s?t=2672) доступны из Microsoft Ignite 2016 в сеансе «стека путь к контейнеризации — преобразование рабочих нагрузок в контейнеры».

## <a name="troubleshooting"></a>Диагностика

### <a name="known-issues"></a>Известные проблемы

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Имя узла контейнера должно совпадать с именем gMSA для Windows Server 2016 и Windows 10 версии 1709 и 1803

Если вы используете Windows Server 2016, версия 1709 или 1803, имя узла контейнера должна соответствовать ваше gMSA имя учетной записи SAM.

Если имя узла не соответствует имени gMSA, входящих запросов проверки подлинности NTLM и преобразование имени/SID (используется многие библиотеки, например роль поставщика членства ASP.NET) не удастся. Kerberos будет продолжать работать нормально, даже если имя узла и gMSA не совпадают.

Это ограничение была устранена в Windows Server 2019, где контейнер будет теперь всегда использовать его имя gMSA в сети, независимо от того, ограниченного имени узла.

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Одновременное использование gMSA с несколькими контейнерами может привести к временные сбои в Windows Server 2016 и Windows 10 версии 1709 и 1803

Поскольку все контейнеры, должны использовать то же имя узла, вторая проблема затрагивает версии Windows до Windows Server 2019 и Windows 10, версия 1809. При назначении несколько контейнеров тем же удостоверением и имя узла гонки происходит, если два вида контейнеров одновременно обращаться к контроллеру домена. Когда другой контейнер презентации на один контроллер домена, она отменит обмен данными с все предыдущие контейнеры, использующие тем же удостоверением. Это может привести к Периодические сбои проверки подлинности и иногда может настраиваться как сбой доверия при запуске `nltest /sc_verify:contoso.com` внутри контейнера.

Мы изменили этого поведения в Windows Server 2019 для разделения идентификатор контейнера на основе имени компьютера, позволяя несколько контейнеров на использование одного одновременно.

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Нельзя использовать gMSAs с контейнерами изоляцией Hyper-V в Windows 10 версии 1703 и 1709, 1803

Инициализация контейнера будет зависание или завершиться с ошибкой при попытке использование с изолированным контейнером Hyper-V в Windows 10 и Windows Server версии 1803, 1703 и 1709.

Эта ошибка была исправлена в Windows Server 2019 и Windows 10, версия 1809. Вы также можете запустить контейнеры Hyper-V изолированными с gMSAs Windows Server 2016 и Windows 10 версии 1607.

### <a name="general-troubleshooting-guidance"></a>Общие рекомендации по устранению неполадок

Если возникновение ошибок при запуске контейнера с gMSA, приведенные ниже инструкции помогут вам определить причину.

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>Убедитесь, что основное приложение может использовать gMSA

1. Убедитесь, что узел является подключено к домену и может связаться с контроллером домена.
2. Установите средства AD PowerShell из средства удаленного администрирования сервера и выполните [Тест-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount) имеет ли компьютер доступа для получения доступным последней. Если командлет возвращает **значение False**, компьютер не имеет доступа к паролю gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Если **Тест-ADServiceAccount** возвращает **значение False**, убедитесь, что узел относится к группе безопасности, можно получить доступ к gMSA пароль.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Если ваш узел входит в группу безопасности, авторизован для получения пароля gMSA, но по-прежнему не удается выполнить **Тест-ADServiceAccount**, может потребоваться перезагрузите компьютер, чтобы получить новый запрос в службу отражающее его текущего членства в группах.

#### <a name="check-the-credential-spec-file"></a>Проверьте файл спецификации учетных данных

1. Запустите **Get-CredentialSpec** из [credentialspec среды PowerShell](https://aka.ms/credspec) , чтобы найти все технические учетных данных на компьютере. Технические характеристики учетных данных должны храниться в папке «CredentialSpecs» в корневом каталоге Docker. Чтобы узнать Docker корневой каталог, работающего сведения docker **-f «{{. DockerRootDir}}»**.
2. Откройте файл CredentialSpec и убедитесь, что правильно заполнено следующие поля:
    - **SID**: SID учетной записи gMSA
    - **MachineAccountName**: gMSA имя учетной записи SAM (не включает полное доменное имя или доллара)
    - **DnsTreeName**: полное доменное имя леса Active Directory
    - **DnsName**: полное доменное имя домена, к которому принадлежит gMSA
    - **NetBiosName**: имя NETBIOS для домена, к которому принадлежит gMSA
    - **GroupManagedServiceAccounts или имя**: имя учетной записи gMSA SAM (не включайте полное доменное имя или доллара)
    - **GroupManagedServiceAccounts/области**: одна запись для полное доменное имя домена, а другой — для NETBIOS

    Ввод должен выглядеть как в следующем примере спецификацию полный учетных данных:

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. Проверьте правильность пути к файлу спецификации учетных данных для вашего решения по управлению. Если вы используете Docker, убедитесь, что включает в себя контейнера, выполните команду `--security-opt="credentialspec=file://NAME.json"`, где «NAME.json» заменяется с вывод имени **Get-CredentialSpec**. Имя — это имя плоский файл, относительно папки CredentialSpecs в корневом каталоге Docker.

#### <a name="check-the-container"></a>Проверьте контейнера

1. Если вы используете версию Windows до Windows Server 2019 или Windows 10, версия 1809, ваше имя контейнера узла должно совпадать с именем gMSA. Обеспечить `--hostname` параметр соответствует gMSA короткое имя (не компонент домена; например, «webapp01» вместо «webapp01.contoso.com»).

2. Проверьте конфигурацию сети контейнера для проверки контейнера может обрабатывать и доступ к контроллеру домена для домена gMSA. Неправильно настроенные DNS-серверы в контейнере являются общие затруднений удостоверение проблем.

3. Установите флажок, если контейнер допустимое подключение к домену, выполнив командлет в контейнере (с помощью `docker exec` или эквивалент):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    Проверка доверия необходимо возвращать `NERR_SUCCESS` Если gMSA и подключения к сети позволяет контейнеру обращаться к домену. Если не удается, проверьте конфигурацию сети контейнера и узла. Оба проекта должны иметь возможность связаться с контроллером домена.

4. Убедитесь, что ваше приложение будет [настроить на использование](#configure-your-application-to-use-the-gmsa). Учетная запись пользователя в контейнере не изменяется, когда вы использование. Вместо системной учетной записи использует gMSA, когда оно обращается к другим сетевым ресурсам. Это означает, что ваше приложение будет необходимо запустить как сетевая служба или локальной системы использовать удостоверение gMSA.

    > [!TIP]
    > При повторном запуске `whoami` или использовать другое средство для определения текущего контекста пользователя в контейнере, вы не увидите само имя gMSA. Это связано с всегда входа в контейнере как локальный пользователь вместо удостоверения домена. GMSA используемую учетную запись компьютера всякий раз, когда он обращается к сетевым ресурсам, именно поэтому ваше приложение должно для запуска в качестве Local System или Network Service.

5. Наконец, если контейнер кажется настроены правильно, но пользователи или другие службы не удается автоматически проверки подлинности контейнерного приложения, проверьте SPN в вашей учетной записи gMSA. Клиенты будут находить учетной записи gMSA по имени, в котором они достигнут приложения. Это может означать, что вам потребуется дополнительных `host` SPN для вашей gMSA, если, например, клиенты подключаться к приложение с помощью балансировки нагрузки или другое имя DNS.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Общие сведения о группе управляемых учетных записей служб](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
