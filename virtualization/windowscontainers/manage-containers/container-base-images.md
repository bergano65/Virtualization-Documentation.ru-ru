---
title: Журнал базовых образов для контейнеров Windows
description: Список образов для контейнеров Windows, выпущенных с хэшами уровня SHA256
keywords: docker, контейнеры, хэши
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107908"
---
# <a name="container-base-images"></a>Базовые образы контейнеров

## <a name="supported-base-images"></a>Поддерживаемые базовые образы

Контейнеры Windows предлагаются четырьмя базовыми изображениями контейнера: Windows Server Core, Nano Server, Windows и IoT основы. Не все конфигурации поддерживают оба образа ОС. В этой таблице указаны поддерживаемые конфигурации.

|Операционная система узла|Контейнер Windows|Изоляция Hyper-V|
|---------------------|-----------------|-----------------|
|Windows Server 2016 или Windows Server 2019 (Standard или Datacenter)|Server Core, Nano Server, Windows|Server Core, Nano Server, Windows|
|Сервер Nano Server|Сервер Nano Server|Server Core, Nano Server, Windows|
|Windows 10 Pro или Windows 10 корпоративный|Отсутствует|Server Core, Nano Server, Windows|
|IoT Базовая|IoT Базовая|Отсутствует|

> [!WARNING]  
> Начиная с Windows Server версии 1709, Nano Server больше не доступен в качестве узла контейнера.

## <a name="base-image-differences"></a>Отличия базовых изображений

Как можно выбрать подходящего базового образа для сборки? Несмотря на то, что вы можете создавать любые из них, Вот общие рекомендации по каждому изображению:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Если для вашего приложения требуется полная версия .NET Framework, лучше использовать это изображение.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): для приложений, которым требуется только платформа .NET Core, Nano Server предоставит очень облегченнаяное изображение.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): вы можете обнаружить, что приложение зависит от компонента или библиотеки DLL, отсутствующей в серверных ядрах или в изображениях Nano Server, таких как библиотеки GDI. Это изображение содержит полный набор зависимостей Windows.
- [Ядро IOT](https://hub.docker.com/_/microsoft-windows-iotcore): это изображение предназначено для [приложений IOT](https://developer.microsoft.com/windows/iot). Этот контейнер следует использовать при нацеливании на основной узел IoT.

Для большинства пользователей наиболее подходящим будет использование Windows Server Core или Nano Server. Ниже перечислены некоторые моменты, которые необходимо учитывать при создании сервера Nano Server.

- стек обслуживания был удален;
- компонент .NET Core не включен (хотя вы можете использовать [образ .NET Core Nano Server](https://hub.docker.com/r/microsoft/dotnet/));
- компонент PowerShell был удален;
- компонент WMI был удален.
- Начиная с Windows Server версии 1709, приложения запускаются в контексте пользователя, поэтому команды, требующие привилегий администратора, выполняться не будут. Вы можете указать учетную запись администратора контейнера с помощью флага пользователя (например, Контаинерадминистратор), но в будущем мы планируем полностью удалить учетные записи администратора с сервера.

Это самые существенные различия, но не полный список. Существуют другие компоненты, которые также отсутствуют. Имейте в виду, что вы всегда можете добавить другие компоненты поверх Nano Server по своему усмотрению. Пример см. здесь: [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
