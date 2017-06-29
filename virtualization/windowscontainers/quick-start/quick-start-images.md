---
title: "Краткое руководство по развертыванию контейнеров— образы"
description: "Краткое руководство по развертыванию контейнеров"
keywords: "docker, контейнеры"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 355daae1b673f0b05f08d0706664967a825de6f7
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: ru-RU
---
# <a name="container-images-on-windows-server"></a>Образы контейнеров в Windows Server

В предыдущем кратком руководстве по Windows Server мы создали контейнер Windows на основе предварительно созданного примера .Net Core. В этом упражнении подробно рассматриваются следующие аспекты: создание пользовательских образов контейнеров вручную, автоматическое создание образа контейнера изображения с помощью Dockerfile и сохранение образов контейнеров в общедоступном реестре Docker Hub.

Это краткое руководство относится к контейнерам Windows Server в Windows Server2016. В нем будет использоваться базовый образ контейнера Windows Server Core. Дополнительную документацию по быстрому началу работы можно найти в содержании в левой части этой страницы.

**Предварительные требования:**

- Одна компьютерная система (физическая или виртуальная), работающая под управлением Windows Server2016.
- Настройте на компьютере компонент контейнеров Windows и Docker. Пошаговые инструкции по этим этапам см. в статье [Контейнеры Windows в Windows Server](./quick-start-windows-server.md).
- Идентификатор Docker, который будет использоваться для отправки образа контейнера в Docker Hub. Если у вас нет идентификатора Docker, зарегистрируйтесь для его получения в [Docker Cloud](https://cloud.docker.com/).

## <a name="1-container-image---manual"></a>1. Образ контейнера— ручная процедура

Для получения наилучших результатов выполняйте это упражнение из командной оболочки (cmd.exe) Windows.

Первый этап ручного создания образа контейнера заключается в развертывании контейнера. В рамках этого примера разверните контейнер IIS из готового образа IIS. После развертывания контейнера вы будете работать в сеансе оболочки из контейнера. Интерактивный сеанс инициируется с помощью с флага `-it`. Дополнительные сведения о команде Docker Run см. в [справке по команде Docker Run на сайте Docker.com](https://docs.docker.com/engine/reference/run/). 

> Этот шаг может занять некоторое время из-за размера базового образа Windows Server Core.

```none
docker run -d --name myIIS -p 80:80 microsoft/iis
```

Теперь контейнер будет работать в фоновом режиме. Команда по умолчанию `ServiceMonitor.exe`, входящая в состав контейнера, которая отслеживает ход выполнения IIS и автоматически останавливает контейнер при остановке IIS. Дополнительные сведения о том, как был создан этот образ см. в разделе [Microsoft/docker-iis](https://github.com/Microsoft/iis-docker) на GitHub.

Затем запустите интерактивную команду в контейнере. Это позволит выполнять команды в запущенном контейнере без остановки IIS или ServiceMonitor.

```none
docker exec -i myIIS cmd 
```

Затем можно внести изменения в работающий контейнер. Выполните следующую команду, чтобы удалить экран-заставку IIS.

```none
del C:\inetpub\wwwroot\iisstart.htm
```

Выполните приведенную ниже команду, чтобы заменить сайт IIS по умолчанию новым статическим сайтом.

```none
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

Перейдя на другую систему, введите IP-адрес узла контейнера в браузере. Должно открыться приложение "Hello, World".

**Примечание.** Если вы работаете в Azure, потребуется правило группы безопасности сети, разрешающее передачу трафика через порт 80. Дополнительные сведения см. в разделе [Create Rule in a Network Security Group](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg) (Создание правила в группе безопасности сети).

![](media/hello.png)

Вернитесь в контейнер и выйдите из интерактивного сеанса.

```none
exit
```

Теперь измененный контейнер можно записать в новый образ контейнера. Для этого потребуется имя контейнера. Его можно определить с помощью команды `docker ps -a`.

```none
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

Чтобы создать образ контейнера, используйте команду `docker commit`. Команда Docker commit принимает вид "docker commit имя_контейнера имя_нового_образа". Примечание. Замените имя контейнера в данном примере на используемое вами.

```none
docker commit pedantic_lichterman modified-iis
```

Чтобы убедиться, что новый образ создан, используйте команду `docker images`.  

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

Теперь можно развернуть этот образ. Полученный контейнер будет содержать все зафиксированные изменения.

## <a name="2-container-image---dockerfile"></a>2. Образ контейнера— Dockerfile

В последнем упражнении мы вручную создали и изменили контейнер, а затем сохранили его в новом образе контейнера. В Docker есть способ автоматизации этого процесса с помощью файла Dockerfile. В этом упражнении будут почти такие же результаты, как и в последнем, но здесь процесс автоматизирован. Для этого упражнения требуется идентификатор Docker. Если у вас нет идентификатора Docker, зарегистрируйтесь для его получения в [Docker Cloud]( https://cloud.docker.com/).

На узле контейнера создайте каталог `c:\build`, а в нем— файл с именем `Dockerfile`. Обратите внимание, что этот файл не должен иметь расширение.

```none
powershell new-item c:\build\Dockerfile -Force
```

Откройте файл Dockerfile в блокноте.

```none
notepad c:\build\Dockerfile
```

Скопируйте в него приведенный ниже текст и сохраните файл. Эти команды дают Docker указание создать новый образ, приняв `microsoft/iis` в качестве основы. После этого файл Dockerfile выполняет команды, заданные в инструкции `RUN`. В этом случае обновляется содержимое файла index.html. 

Дополнительные сведения о файлах Dockerfile см. в статье [Файлы Dockerfile в Windows](../manage-docker/manage-windows-dockerfile.md).

```none
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

Команда `docker build` запускает процесс создания образа. Параметр `-t` дает процессу указание присвоить новому образу имя `iis-dockerfile`. **Замените "user" на имя пользователя учетной записи Docker**. Если у вас нет учетной записи Docker, зарегистрируйтесь для ее получения в [Docker Cloud](https://cloud.docker.com/).

```none
docker build -t <user>/iis-dockerfile c:\Build
```

После завершения можно убедиться, что образ создан, используя команду `docker images`.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Теперь разверните контейнер с помощью следующей команды, снова заменив имя пользователя на свой идентификатор Docker.

```none
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

После создания контейнера введите IP-адрес узла контейнера в браузере. Должно открыться приложение "Hello, World!".

![](media/dockerfile2.png)

Вернитесь на узел контейнера и выполните команду `docker ps`, чтобы получить имя контейнера, и команду `docker rm`, чтобы удалить контейнер. Примечание. Замените имя контейнера в данном примере на используемое вами.

Получите имя контейнера.

```none
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Удалите контейнер.

```none
docker rm -f <container name>
```

## <a name="3-docker-push"></a>3. Операция отправки в Docker

Образы контейнеров Docker можно хранить в реестре контейнеров. Если образ хранится в реестре, его можно извлечь для последующего использования на нескольких различных узлах контейнеров. Docker предоставляет открытый реестр для хранения образов контейнеров в [Docker Hub](https://hub.docker.com/).

В этом упражнении мы отправим пользовательский образ "Hello world" в вашу учетную запись на Docker Hub.

Войдите в учетную запись Docker с помощью `docker login command`.

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

После входа в систему можно отправить образ контейнера в Docker Hub. Для этого используйте команду `docker push`. **Замените "user" на идентификатор Docker**. 

```none
docker push <user>/iis-dockerfile
```

Теперь можно скачать образ контейнера из Docker Hub на любой узел контейнера Windows с помощью `docker pull`. В этом учебнике мы удалим существующий образ и затем извлечем его из Docker Hub. 

```none
docker rmi <user>/iis-dockerfile
```

Команда `docker images` подтвердит, что образ был удален.

```none
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

Наконец, с помощью команды "docker pull" можно поместить образ обратно в узел контейнера. Замените "user" на имя пользователя учетной записи Docker. 

```none
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>Дальнейшие действия

[Контейнеры Windows в Windows10](./quick-start-windows-10.md)
