---
title: Контейнеры для Windows и Linux в Windows 10
description: Краткое руководство по развертыванию контейнеров
keywords: Dock, Containers, ЛКОВ
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288132"
---
# <a name="get-started-run-your-first-windows-container"></a>Начало работы: запуск первого контейнера Windows

В этой статье описано, как выполнить первый контейнер Windows после настройки среды, как описано в разделе Начало [работы с Windows для контейнеров](./set-up-environment.md). Для запуска контейнера сначала необходимо установить базовый образ, который предоставляет основу для создания базового уровня служб операционной системы для контейнера. Затем вы создаете и запускаете изображение контейнера, которое основывается на базовом изображении. Подробнее читайте в статье.

## <a name="install-a-container-base-image"></a>Установка базового образа контейнера

Все контейнеры создаются из изображений контейнера. Корпорация Майкрософт поддерживает несколько начальных изображений, которые называются базовыми изображениями (Дополнительные сведения можно найти в разделе [основные образы контейнера](../manage-containers/container-base-images.md)). Эти процедуры загружают (загрузки и устанавливаются) базового образа облегченного Nano Server.

1. Откройте окно командной строки (например, встроенная командная строка, PowerShell или [терминал Windows](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)), а затем выполните следующую команду, чтобы скачать и установить базовый образ:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Если появляется сообщение об ошибке `no matching manifest for unknown in the manifest list entries`, убедитесь в том, что Dock не настроен для работы с контейнерами Linux.

2. После завершения загрузки изображения прочтите [лицензионное соглашение](../images-eula.md) , пока не дождитесь завершения сеанса, и запросите его в вашем локальном репозитории образов. Выполнение команды `docker images` возвращает список установленных изображений.

   Ниже приведен пример выходных данных, демонстрирующих изображение Nano Server.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Запуск контейнера Windows

В этом простом примере будет создан и развернуто изображение контейнера "Hello World". Для оптимальной работы выполните эти команды в окне командной строки с повышенными привилегиями (но не используйте среду Windows PowerShell ISE — она не работает для интерактивных сеансов с контейнерами, так как контейнеры отображаются для зависания).

1. Запустите контейнер с интерактивным сеансом из `nanoserver` образа, введя в окне командной строки следующую команду:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. После запуска контейнера окно командной строки изменит контекст на контейнер. В контейнере мы создадим простой текстовый файл "Hello World", а затем выполни контейнер, введя следующие команды:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Получите идентификатор контейнера для контейнера, который вы только что вышли, запустив команду [Dock PS](https://docs.docker.com/engine/reference/commandline/ps/) :

   ```console
   docker ps -a
   ```

4. Создайте новый образ HelloWorld, включающий изменения в первом контейнере, который вы запустили. Для этого выполните команду [commit Dock](https://docs.docker.com/engine/reference/commandline/commit/) , заменив `<containerid>` ее идентификатором контейнера.

   ```console
   docker commit <containerid> helloworld
   ```

   После завершения вы получите пользовательский образ, содержащий скрипт "Привет мир". Это может быть показано с помощью команды [Images Dock](https://docs.docker.com/engine/reference/commandline/images/) .

   ```console
   docker images
   ```

   Ниже приведен пример выходных данных.

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Наконец, запустите новый контейнер с помощью команды " [выполнить Dock](https://docs.docker.com/engine/reference/commandline/run/) " с `--rm` параметром, который автоматически удаляет контейнер после того, как Командная строка (cmd. exe) прекратит работу.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   Результат заключается в том, что контейнер создан из изображения "HelloWorld", экземпляр Cmd. exe был запущен в контейнере, который считывает наш файл и выводит содержимое файла в оболочку, а затем контейнер остановлен и удален.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о том, как контаинеризе пример приложения](./building-sample-app.md)
