---
title: Выполнение контейнера с Гмса
description: Работа с контейнером Windows с помощью групповой управляемой учетной записи службы (Гмса).
keywords: Dock, Containers, Active Directory, гмса, групповая управляемая учетная запись службы, групповая управляемая учетные записи служб
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b9c0406b5fe9527d88365dabf0cfd10114c34c74
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079763"
---
# <a name="run-a-container-with-a-gmsa"></a>Выполнение контейнера с Гмса

Чтобы выполнить контейнер с групповой управляемой учетной записью службы (Гмса), укажите в качестве файла спецификации `--security-opt` учетных данных параметр [выполнения Dock](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>В Windows Server 2016 версий 1709 и 1803 имя узла контейнера должно совпадать с коротким именем Гмса.

В предыдущем примере именем учетной записи SAM Гмса является "webapp01", поэтому имя узла контейнера также называется "webapp01".

В Windows Server 2019 и более поздних версиях поле HostName не является обязательным, но контейнер будет по-прежнему определяться именем Гмса, а не именем узла, даже если явным образом вы явно задаете его.

Чтобы проверить, правильно ли работает Гмса, выполните в контейнере следующий командлет:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Если состояние подключения доверенного домена и состояние проверки доверия не `NERR_Success`установлены, следуйте инструкциям по [устранению неполадок для устранения](gmsa-troubleshooting.md#check-the-container) проблемы.

Вы можете проверить удостоверение Гмса в контейнере, выполнив следующую команду и проверив имя клиента:

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

Чтобы открыть PowerShell или другое консольное приложение в качестве учетной записи Гмса, вы можете задать для него разрешение на работу с учетной записью Network Service, а не с обычной Контаинерадминистратор (или Контаинерусер для сервера для серверов).

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Если вы используете сетевую службу, вы можете проверить подлинность сети как Гмса, пытаясь подключиться к SYSVOL на контроллере домена.

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>Дальнейшие действия

В дополнение к запущенным контейнерам вы также можете использовать Гмсас, чтобы:

- [Настройка приложений](gmsa-configure-app.md)
- [Контейнеры для оркестрации](gmsa-orchestrate-containers.md)

Если во время установки возникнут проблемы, ознакомьтесь с нашим [руководством по устранению неполадок для поиска](gmsa-troubleshooting.md) возможных решений.
