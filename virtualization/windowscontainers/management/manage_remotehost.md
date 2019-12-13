---
title: Удаленное управление узлом Docker в Windows
description: Безопасное управление удаленным узлом Docker, работающим в Windows Server.
keywords: docker, контейнеры
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
ms.openlocfilehash: b975c593bd5c736ec3e7e1e21b76b2f6a2c8f8a4
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909654"
---
# <a name="remote-management-of-a-windows-docker-host"></a>Удаленное управление узлом Docker в Windows

Даже при отсутствии `docker-machine` можно создать доступный удаленно узел Docker на виртуальной машине с Windows Server 2016.

Процесс очень прост.

* Создайте сертификаты на сервере с помощью [dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/). Если вы создаете сертификаты с IP-адресом, можно использовать статический IP-адрес, чтобы не создавать сертификаты повторно при изменении IP-адреса.

* Перезапустите службу DOCKER `Restart-Service Docker`
* Сделайте TLS-порты 2375 и 2376 Docker доступными, создав правило NSG, разрешающее входящий трафик. Обратите внимание, что для безопасных подключений необходимо разрешить только порт 2376.  
  На портале должна быть указана конфигурация NSG, подобная показанной ниже:  
  ![NGS](media/nsg.png)  
  
* Разрешите входящие подключения через брандмауэр Windows. 
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* Скопируйте файлы `ca.pem`, cert.pem и key.pem из папки Docker на компьютере, например `c:\users\chris\.docker`, на локальный компьютер. Например, вы можете скопировать (CTRL+C) и вставить (CTRL+V) файлы из сеанса RDP. 
* Убедитесь, что вы можете подключиться к удаленному узлу Docker. Запуск
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## <a name="troubleshooting"></a>Поиск и устранение неисправностей
### <a name="try-connecting-without-tls-to-determine-your-nsg-firewall-settings-are-correct"></a>Попытка подключения без TLS для проверки параметров брандмауэра NSG
Ошибки подключения обычно проявляются следующим образом:
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

Разрешите незашифрованный подключения, добавив 
```
{
    "tlsverify":  false,
}
```
в `c"\programdata\docker\config\daemon.json`, а затем перезапустите службу.

Подключитесь к удаленному узлу с помощью следующей команды:
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### <a name="cert-problems"></a>Проблемы с сертификатами
При доступе к узлу Docker с помощью сертификата, который не был создан для IP-адреса или DNS-имени, возникнет ошибка:
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
Убедитесь, что w.x.y.z — это DNS-имя для общего IP-адреса узла и что DNS-имя соответствует [общему имени](https://www.ssl.com/faqs/common-name/) сертификата, которое указано в переменной среды `SERVER_NAME` или соответствует одному из IP-адресов в переменной `IP_ADDRESSES`, предоставленной dockertls.

### <a name="cryptox509-warning"></a>Предупреждение crypto/x509
Вы можете получить предупреждение: 
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
Это предупреждение можно проигнорировать.
