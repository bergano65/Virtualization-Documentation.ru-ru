---
title: Контейнеры для Windows и Linux в Windows 10
description: Краткое руководство по развертыванию контейнеров
keywords: Dock, Containers, ЛКОВ
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129386"
---
# <a name="get-started-run-your-first-container"></a>Начало работы: выполнение первого контейнера

В [предыдущем сегменте](./set-up-environment.md)мы настроили среду для выполнения контейнеров. В этом упражнении показано, как извлечь изображение контейнера и запустить его.

## <a name="install-container-base-image"></a>Базовый образ контейнера установки

Создаются экземпляры всех контейнеров `container images`. Microsoft поддерживает несколько начальных изображений (называемых `base images`) для выбора из них. Приведенная ниже команда извлекает базовый образ Nano Server.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> Если появляется сообщение об ошибке `no matching manifest for unknown in the manifest list entries`, убедитесь, что на панели Dock не настроено выполнение контейнеров Linux.

После того как изображение будет извлечено, вы можете убедиться, что оно установлено на вашем компьютере, запросите репозиторий локального образа стыковочного узла. При выполнении команды `docker images` возвращается список установленных изображений (в данном случае — образ Nano Server).

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Пожалуйста, прочтите [лицензионное соглашение](../images-eula.md)для образа ОС Windows Containers.

## <a name="run-your-first-windows-container"></a>Запуск первого контейнера Windows

В этом простом примере будет создан и развернуто изображение контейнера "Hello World". Для оптимальной работы выполните эти команды в оболочке Windows CMD или PowerShell с повышенными привилегиями.

> Интегрированная среда сценариев Windows PowerShell не работает в интерактивных сеансах с контейнерами. Даже если контейнер запускается, он зависает.

Сначала запустите контейнер с помощью интерактивного сеанса из образа `nanoserver`. После начала работы над контейнером в контейнере будет представлена Командная оболочка.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

В контейнере будет создан простой текстовый файл "Здравствуй, мир".

```cmd
echo "Hello World!" > Hello.txt
```   

После завершения выйдите из контейнера.

```cmd
exit
```

Создание нового изображения контейнера из измененного контейнера. Чтобы просмотреть список контейнеров, которые выполняются или вышли из приложения, выполните указанные ниже действия и запишите идентификатор контейнера.

```console
docker ps -a
```

Выполните следующую команду для создания нового образа "Привет мир". Замените `<containerid>` на идентификатор контейнера.

```console
docker commit <containerid> helloworld
```

После завершения вы получите пользовательский образ, содержащий скрипт "Привет мир". Чтобы убедиться в этом, используйте следующую команду.

```console
docker images
```

Наконец, запустите контейнер с помощью `docker run` команды.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

В результате выполнения `docker run` команды контейнер был создан на основе изображения "HelloWorld", экземпляр Cmd был запущен в контейнере и выполнил чтение нашего файла (вывод Echo в оболочке), а затем контейнер остановлен и удален.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о том, как контаинеризе пример приложения](./building-sample-app.md)
