---
title: Создание Gmsa для контейнеров Windows
description: Создание групповых управляемых учетных записей служб (Gmsa) для контейнеров Windows.
keywords: DOCKER, контейнеры, Active Directory, gmsa, групповая управляемая учетная запись службы, групповые управляемые учетные записи служб
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910254"
---
# <a name="create-gmsas-for-windows-containers"></a>Создание Gmsa для контейнеров Windows

Сети на основе Windows обычно используют Active Directory (AD) для упрощения проверки подлинности и авторизации между пользователями, компьютерами и другими сетевыми ресурсами. Разработчики корпоративных приложений часто разработают свои приложения как интегрированные в AD и работают на присоединенных к домену серверах, чтобы воспользоваться преимуществами встроенной проверки подлинности Windows, что упрощает пользователям и другим службам автоматически и прозрачно входить в приложение с удостоверениями.

Хотя контейнеры Windows не могут быть присоединены к домену, они по-прежнему могут использовать Active Directory удостоверения доменов для поддержки различных сценариев проверки подлинности.

Для этого можно настроить контейнер Windows для работы с [групповой управляемой учетной записью службы](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA), которая представляет собой специальный тип учетной записи службы, представленной в Windows Server 2012, которая позволяет нескольким компьютерам совместно использовать удостоверение без необходимости знать пароль.

При запуске контейнера с gMSA узел контейнера получает пароль gMSA из контроллера домена Active Directory и передает его экземпляру контейнера. Контейнер будет использовать учетные данные gMSA всякий раз, когда учетной записи компьютера (SYSTEM) требуется доступ к сетевым ресурсам.

В этой статье объясняется, как начать работу с Active Directory групповой управляемыми учетными записями службы с контейнерами Windows.

## <a name="prerequisites"></a>Предварительные условия

Чтобы запустить контейнер Windows с групповой управляемой учетной записью службы, вам потребуется следующее:

- Домен Active Directory с как минимум одним контроллером домена под Windows Server 2012 или более поздней версии. Для использования Gmsa не требуются требования леса или режима работы домена, но пароли gMSA могут распространяться только контроллерами домена под управлением Windows Server 2012 или более поздней версии. Дополнительные сведения см. в статье [Active Directory требования к gmsa](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Разрешение на создание учетной записи gMSA. Чтобы создать учетную запись gMSA, необходимо быть администратором домена или использовать учетную запись, которая делегирована разрешение *CREATE msDS-граупманажедсервицеаккаунт Objects* .
- Доступ к Интернету для загрузки модуля PowerShell Кредентиалспек. При работе в отключенной среде можно [сохранить модуль](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) на компьютере с доступом к Интернету и скопировать его на компьютер разработки или узел контейнера.

## <a name="one-time-preparation-of-active-directory"></a>Однократная подготовка Active Directory

Если вы еще не создали gMSA в своем домене, необходимо создать корневой ключ службы распространения ключей (KDS). KDS отвечает за создание, вращение и освобождение пароля gMSA на полномочных узлах. Если узел контейнера должен использовать gMSA для запуска контейнера, он свяжется с KDS для получения текущего пароля.

Чтобы проверить, был ли уже создан корневой ключ KDS, выполните следующий командлет PowerShell от имени администратора домена на контроллере домена или члене домена с установленными инструментами AD PowerShell:

```powershell
Get-KdsRootKey
```

Если команда возвращает идентификатор ключа, все готово и можно сразу перейти к разделу [Создание групповой управляемой учетной записи службы](#create-a-group-managed-service-account) . В противном случае продолжите, чтобы создать корневой ключ KDS.

В рабочей среде или в тестовой среде с несколькими контроллерами домена выполните следующий командлет в PowerShell с правами администратора домена, чтобы создать корневой ключ KDS.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Хотя команда предполагает, что ключ будет действовать немедленно, необходимо подождать 10 часов, прежде чем корневой ключ KDS будет реплицирован и доступен для использования на всех контроллерах домена.

Если в вашем домене только один контроллер домена, можно ускорить процесс, установив для ключа силу 10 часов назад.

>[!IMPORTANT]
>Не используйте этот метод в рабочей среде.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Создание групповой управляемой учетной записи службы

Для каждого контейнера, использующего встроенную проверку подлинности Windows, требуется по крайней мере одна gMSA. Первичная gMSA используется всякий раз, когда приложения, работающие в качестве системы, или сетевые службы обращаются к сети. Имя gMSA станет именем контейнера в сети независимо от имени узла, назначенного контейнеру. Кроме того, контейнеры можно настроить с дополнительным Gmsa, если вы хотите запустить службу или приложение в контейнере от имени другого удостоверения из учетной записи компьютера-контейнера.

При создании gMSA также создается общее удостоверение, которое можно использовать одновременно на нескольких разных компьютерах. Доступ к паролю gMSA защищается списком управления доступом Active Directory. Рекомендуется создать группу безопасности для каждой учетной записи gMSA и добавить соответствующие узлы контейнера в группу безопасности, чтобы ограничить доступ к паролю.

Наконец, поскольку контейнеры не регистрируют имена субъектов-служб автоматически, необходимо вручную создать по крайней мере имя субъекта-службы узла для учетной записи gMSA.

Как правило, узел или имя участника-службы HTTP зарегистрированы с тем же именем, что и учетная запись gMSA, но может потребоваться использовать другое имя службы, если клиенты обращаются к контейнерному приложению из подсистемы балансировки нагрузки или DNS-имени, которое отличается от имени gMSA.

Например, если учетная запись gMSA называется "WebApp01", но пользователи обращаются к сайту по адресу `mysite.contoso.com`, необходимо зарегистрировать имя субъекта-службы `http/mysite.contoso.com` в учетной записи gMSA.

Некоторым приложениям могут потребоваться дополнительные имена участников-служб для своих уникальных протоколов. Например, для SQL Server требуется `MSSQLSvc/hostname` SPN.

В следующей таблице перечислены необходимые атрибуты для создания gMSA.

|gMSA, свойство | Обязательное значение | Пример |
|--------------|----------------|--------|
|Имя | Любое допустимое имя учетной записи. | `WebApp01` |
|Атрибуты | Имя домена, добавляемое к имени учетной записи. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Задайте по крайней мере имя субъекта-службы узла, при необходимости добавьте другие протоколы. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | Группа безопасности, содержащая узлы контейнера. | `WebApp01Hosts` |

Когда вы решили имя для gMSA, выполните следующие командлеты в PowerShell, чтобы создать группу безопасности и gMSA.

> [!TIP]
> Вам потребуется использовать учетную запись, принадлежащую группе безопасности **Администраторы домена** , или делегировано разрешение **CREATE msDS-граупманажедсервицеаккаунт Objects** для выполнения следующих команд.
> Командлет [New-адсервицеаккаунт](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) входит в состав средств AD PowerShell из [средства удаленного администрирования сервера](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

Мы рекомендуем создавать отдельные учетные записи gMSA для сред разработки, тестирования и рабочей среды.

## <a name="prepare-your-container-host"></a>Подготовка узла контейнера

Каждый узел контейнера, на котором будет выполняться контейнер Windows с gMSA, должен быть присоединен к домену и иметь доступ для получения пароля gMSA.

1. Присоедините компьютер к домену Active Directory.
2. Убедитесь, что узел принадлежит группе безопасности, управляющей доступом к паролю gMSA.
3. Перезагрузите компьютер, чтобы он получит новое членство в группе.
4. Настройте [DOCKER Desktop для Windows 10](https://docs.docker.com/docker-for-windows/install/) или [DOCKER для Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5. Такую Убедитесь, что узел может использовать учетную запись gMSA, выполнив [Test-адсервицеаккаунт](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount). Если команда возвращает **значение false**, следуйте [инструкциям по устранению неполадок](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa).

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Создание спецификации учетных данных

Файл спецификации учетных данных — это документ JSON, содержащий метаданные о gMSA учетных записях, которые вы хотите использовать в контейнере. При хранении конфигурации удостоверения отдельно от образа контейнера можно изменить, какой gMSA контейнер использует, просто заменив файл спецификации учетных данных, не изменяя код.

Файл спецификации учетных данных создается с помощью [модуля PowerShell кредентиалспек](https://aka.ms/credspec) на узле контейнера, присоединенного к домену.
После создания файла его можно скопировать на другие узлы контейнера или в контейнер Orchestrator.
Файл спецификации учетных данных не содержит секретов, например пароль gMSA, так как узел контейнера получает gMSA от имени контейнера.

DOCKER ждет найти файл спецификации учетных данных в каталоге **кредентиалспекс** в каталоге данных DOCKER. В установке по умолчанию эта папка находится по адресу `C:\ProgramData\Docker\CredentialSpecs`.

Чтобы создать файл спецификации учетных данных на узле контейнера, выполните следующие действия.

1. Установка средств RSAT AD PowerShell
    - Для Windows Server выполните команду **Install-WINDOWSFEATURE RSAT-AD-PowerShell**.
    - Для Windows 10, 1809 или более поздней версии, выполните команду **Add-виндовскапабилити-Online-name ' RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0 '** .
    - Более ранние версии Windows 10 см. в разделе <https://aka.ms/rsat>.
2. Выполните следующий командлет, чтобы установить последнюю версию [модуля PowerShell кредентиалспек](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Если у вас нет доступа к Интернету на узле контейнера, запустите `Save-Module CredentialSpec` на компьютере, подключенном к Интернету, и скопируйте папку модуля в `C:\Program Files\WindowsPowerShell\Modules` или в другое расположение в `$env:PSModulePath` на узле контейнера.

3. Выполните следующий командлет, чтобы создать новый файл спецификации учетных данных:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    По умолчанию командлет создаст спецификацию CRED, используя предоставленное имя gMSA в качестве учетной записи компьютера для контейнера. Файл будет сохранен в каталоге DOCKER Кредентиалспекс с использованием домена gMSA и имени учетной записи для имени файла.

    Вы можете создать спецификацию учетных данных, включающую дополнительные учетные записи gMSA, если вы используете службу или процесс в качестве вторичного gMSA в контейнере. Для этого используйте параметр `-AdditionalAccounts`:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Чтобы получить полный список поддерживаемых параметров, выполните `Get-Help New-CredentialSpec`.

4. Вы можете отобразить список всех спецификаций учетных данных и их полный путь с помощью следующего командлета:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили учетную запись gMSA, ее можно использовать в следующих случаях:

- [Настройка приложений](gmsa-configure-app.md)
- [Выполнение контейнеров](gmsa-run-container.md)
- [Управление контейнерами](gmsa-orchestrate-containers.md)

Если во время установки возникнут проблемы, ознакомьтесь с нашим [руководством по устранению неполадок](gmsa-troubleshooting.md) , чтобы получить возможные решения.

## <a name="additional-resources"></a>Дополнительные ресурсы

- Дополнительные сведения о Gmsa см. в разделе [Общие сведения о групповой управляемых учетных записях служб](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Для демонстрации видео просмотрите нашу [записанную демонстрацию](https://youtu.be/cZHPz80I-3s?t=2672) от Ignite 2016.
