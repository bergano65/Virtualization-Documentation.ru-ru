---
title: Контейнеры Windows и Linux в Windows 10
description: Краткое руководство по развертыванию контейнеров
keywords: DOCKER, контейнеры, ЛКОВ
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909574"
---
# <a name="get-started-run-your-first-windows-container"></a>Начало работы: запуск первого контейнера Windows

В этом разделе описывается, как запустить первый контейнер Windows после настройки среды, как описано в статье [Начало работы: подготовка окон для контейнеров](./set-up-environment.md). Чтобы запустить контейнер, сначала необходимо установить базовый образ, который предоставляет фундаментальный уровень служб операционной системы для контейнера. Затем создается и запускается образ контейнера, основанный на базовом образе. Дополнительные сведения см. в статье.

## <a name="install-a-container-base-image"></a>Установка базового образа контейнера

Все контейнеры создаются из образов контейнеров. Корпорация Майкрософт предлагает несколько начальных образов, называемых базовыми образами (Дополнительные сведения см. в разделе [базовые образы контейнера](../manage-containers/container-base-images.md)). Эти процедуры запрашивают (скачивает и устанавливает) базовый образ сервера облегченного Nano Server.

1. Откройте окно командной строки (например, встроенную командную строку, PowerShell или [терминал Windows](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)), а затем выполните следующую команду, чтобы скачать и установить базовый образ:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Если отображается сообщение об ошибке с текстом `no matching manifest for unknown in the manifest list entries`, убедитесь, что DOCKER не настроен для запуска контейнеров Linux.

2. После завершения загрузки образа прочтите [лицензионное соглашение](../images-eula.md) во время ожидания — проверьте его наличие в вашей системе, запросив локальный репозиторий образов DOCKER. Выполнение команды `docker images` возвращает список установленных образов.

   Ниже приведен пример выходных данных, демонстрирующих образ Nano Server.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Запуск контейнера Windows

В этом простом примере будет создан и развернут образ контейнера "Hello World". Для наилучшего удобства выполните эти команды в окне командной строки с повышенными привилегиями (но не используйте интегрированную среду сценариев Windows PowerShell), так как они не работают для интерактивных сеансов с контейнерами, так как контейнеры зависает.

1. Запустите контейнер с интерактивным сеансом из образа `nanoserver`, введя следующую команду в окне командной строки:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. После запуска контейнера окно командной строки изменит контекст на контейнер. Внутри контейнера мы создадим простой текстовый файл "Hello World", а затем выберем контейнер, введя следующие команды:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Получите идентификатор контейнера для контейнера, который вы только что вышли, выполнив команду [DOCKER PS](https://docs.docker.com/engine/reference/commandline/ps/) :

   ```console
   docker ps -a
   ```

4. Создайте новый образ HelloWorld, содержащий изменения в первом контейнере, который вы запустили. Для этого выполните команду [DOCKER Commit](https://docs.docker.com/engine/reference/commandline/commit/) , заменив `<containerid>` идентификатором контейнера:

   ```console
   docker commit <containerid> helloworld
   ```

   После завершения вы получите пользовательский образ, содержащий скрипт "Привет мир". Это можно увидеть с помощью команды [DOCKER Images](https://docs.docker.com/engine/reference/commandline/images/) .

   ```console
   docker images
   ```

   Ниже приведен пример выходных данных.

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Наконец, запустите новый контейнер с помощью команды [DOCKER Run](https://docs.docker.com/engine/reference/commandline/run/) с параметром `--rm`, который автоматически удаляет контейнер после остановки командной строки (cmd. exe).

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   В результате создается контейнер из образа HelloWorld, в контейнере был запущен экземпляр Cmd. exe, который считывает наш файл и выводит содержимое файла в оболочку, а затем контейнер останавливается и удаляется.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Узнайте, как контейнеризовать пример приложения.](./building-sample-app.md)
