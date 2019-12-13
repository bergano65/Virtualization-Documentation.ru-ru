---
title: Запуск контейнера с помощью gMSA
description: Как запустить контейнер Windows с групповой управляемой учетной записью службы (gMSA).
keywords: DOCKER, контейнеры, Active Directory, gmsa, групповая управляемая учетная запись службы, групповые управляемые учетные записи служб
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 52625517748356251aa41115caebd7801ec3cdaf
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909764"
---
# <a name="run-a-container-with-a-gmsa"></a>Запуск контейнера с помощью gMSA

Чтобы запустить контейнер с групповой управляемой учетной записью службы (gMSA), предоставьте файл спецификации учетных данных в параметре `--security-opt` [выполнения DOCKER](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>В Windows Server 2016 версии 1709 и 1803 имя узла контейнера должно соответствовать короткому имени gMSA.

В предыдущем примере имя учетной записи SAM gMSA — «webapp01», поэтому имя узла контейнера также называется «webapp01».

В Windows Server 2019 и более поздних версиях поле HostName не является обязательным, но контейнер по-прежнему будет идентифицировать себя по имени gMSA, а не имени узла, даже если явно указать другое имя.

Чтобы проверить, правильно ли работает gMSA, выполните следующий командлет в контейнере:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Если состояние подключения к доверенному контроллеру домена и состояние проверки доверия не `NERR_Success`, следуйте [инструкциям по устранению неполадок](gmsa-troubleshooting.md#check-the-container) , чтобы отладить проблему.

Удостоверение gMSA можно проверить в контейнере, выполнив следующую команду и проверив имя клиента:

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

Чтобы открыть PowerShell или другое консольное приложение в качестве учетной записи gMSA, можно попросить его запустить под учетной записью сетевой службы вместо обычной учетной записи Контаинерадминистратор (или Контаинерусер for Server):

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

При работе в качестве сетевой службы можно проверить сетевую проверку подлинности в качестве gMSA, пытаясь подключиться к SYSVOL на контроллере домена:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>Дальнейшие действия

В дополнение к работе с контейнерами можно также использовать Gmsa для:

- [Настройка приложений](gmsa-configure-app.md)
- [Управление контейнерами](gmsa-orchestrate-containers.md)

Если во время установки возникнут проблемы, ознакомьтесь с нашим [руководством по устранению неполадок](gmsa-troubleshooting.md) , чтобы получить возможные решения.
