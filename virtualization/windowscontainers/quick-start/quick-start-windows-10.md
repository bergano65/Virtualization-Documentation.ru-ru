---
title: Контейнеры для Windows и Linux в Windows 10
description: Краткое руководство по развертыванию контейнеров
keywords: Dock, Containers, ЛКОВ
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: aa7a20d914fdb65597c0f31ef6d53b91f6497be4
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/20/2019
ms.locfileid: "10044964"
---
# <a name="windows-containers-on-windows-10"></a>Контейнеры Windows в Windows10

С помощью этого упражнения вы сможете создавать и запускать контейнеры Windows в Windows 10.

Это краткое руководство поможет вам сделать следующее:

1. Установка стыковочного компьютера
2. Выполнение простого контейнера Windows

Это краткое руководство применимо только к Windows 10. Дополнительные краткие руководства по началу работы можно найти в оглавлении в левой части страницы.

## <a name="prerequisites"></a>Что вам понадобится
Убедитесь, что вы отвечаете на следующие требования:
- Одна физическая компьютерная система под управлением Windows 10 профессиональная или Корпоративная с обновлением юбилея (версия 1607) или более поздней. 
- Убедитесь, что включена [технология Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) .

***Изоляция Hyper-V:*** Для предоставления разработчикам одной и той же версии ядра и конфигурации, которая будет использоваться в производстве, для контейнеров Windows Server требуется изоляция Hyper-V, дополнительные сведения об изоляции Hyper-V можно найти на странице " [о программе](../about/index.md) " в контейнере Windows.

> [!NOTE]
> В выпуске Windows Октябрь Update 2018 мы больше не запрещают пользователям запускать контейнер Windows в режиме изоляции процессов в Windows 10 Корпоративная или профессиональная для разработки и тестирования. Дополнительные сведения можно найти в разделе " [часто задаваемые вопросы](../about/faq.md) ".

## <a name="install-docker-desktop"></a>Установка стыковочного компьютера

Скачайте закрепление для настольного [компьютера](https://store.docker.com/editions/community/docker-ce-desktop-windows) и запустите установщик (вам потребуется выполнить вход. Создайте учетную запись, если она еще не установлена. [Подробные инструкции по установке](https://docs.docker.com/docker-for-windows/install) доступны в документации по Docker.

## <a name="switch-to-windows-containers"></a>Переключение на контейнеры Windows

После установки закрепления по умолчанию для рабочего стола используется контейнеры Linux. Переключитесь на контейнеры Windows с помощью закрепляемого меню или в командной строке PowerShell, выполнив следующую команду:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>Установка базовых образов контейнеров

Контейнеры Windows создаются из базовых образов. Приведенная ниже команда извлекает базовый образ Nano Server.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!NOTE]
> Если появляется сообщение об ошибке `no matching manifest for unknown in the manifest list entries`, убедитесь, что вы не ожидаете вытягивания контейнера Linux.

После его получения выполните команду `docker images`, чтобы вывести список установленных образов. В данном случае будет отображен образ Nano Server.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Пожалуйста, прочтите [лицензионное соглашение](../images-eula.md)для образа ОС Windows Containers.

## <a name="run-your-first-windows-container"></a>Запуск первого контейнера Windows

В этом простом примере будет создан и развернуто изображение контейнера "Hello World". Для получения наилучших результатов выполните следующие команды в командной строке Windows или PowerShell с повышенными привилегиями.

> Интегрированная среда сценариев Windows PowerShell не работает в интерактивных сеансах с контейнерами. Даже если контейнер запускается, он зависает.

Сначала запустите контейнер с помощью интерактивного сеанса из образа `nanoserver`. После запуска контейнера в его рамках появится командная оболочка.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Внутри контейнера мы создадим простой текстовый файл "Hello World".

```cmd
echo "Hello World!" > Hello.txt
```   

После завершения выйдите из контейнера.

```cmd
exit
```

Теперь создайте новый образ контейнера из измененного контейнера. Чтобы просмотреть список контейнеров, выполните следующую команду и запишите идентификатор контейнера.

```console
docker ps -a
```

Выполните следующую команду для создания нового образа "Привет мир". Замените <containerid> на идентификатор контейнера.

```console
docker commit <containerid> helloworld
```

После завершения вы получите пользовательский образ, содержащий скрипт "Привет мир". Чтобы убедиться в этом, используйте следующую команду.

```console
docker images
```

Наконец, чтобы запустить контейнер, используйте команду `docker run`.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

Результатом выполнения `docker run` команды является то, что контейнер, работающий под управлением Hyper-V, был создан на основе изображения "HelloWorld", экземпляр Cmd был запущен в контейнере и выполнил чтение нашего файла (вывод получен в оболочке), а затем контейнер остановлены и удалены.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о том, как создать пример приложения](./building-sample-app.md)
