---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/02/2017
---
# Windows Containers

## Что такое контейнеры?

Контейнеры— это способ разместить приложение в собственной изолированной «коробке». У приложения в контейнере нет сведений о других приложениях или процессах, размещенных за пределами этой «коробки». Все необходимое приложению для успешной работы также находится в этом контейнере.  Куда бы контейнер не переместить, приложение всегда будет работать, так как оно получит все необходимое для запуска.

Представьте себе кухню. Здесь мы размещаем все устройства и мебель, кастрюли и сковородки, моющее средство и полотенца. Это наш контейнер.

<center style="margin: 25px">![](media/box1.png)</center>

Теперь мы возьмем этот контейнер и поместим его в любую квартиру, при этом кухня никак не изменится. Нам нужно только подключить электричество и воду, и мы сразу сможем начать готовить (так как у нас есть все необходимые устройства).

<center style="margin: 25px">![](media/apartment.png)</center>

Контейнеры похожи на такую кухню. Могут существовать различные виды комнаты, а также множество комнат одного типа. Главное здесь— то, что контейнеры содержат все необходимое для приложения.

Краткий обзор приведен в видео [Контейнеры на основе Windows: разработка современных приложений с корпоративным уровнем управления](https://youtu.be/Ryx3o0rD5lY).

## Принципы работы контейнера

Контейнеры— это изолированная и переносимая среда выполнения с контролируемыми ресурсами, которая работает на хост-компьютере или виртуальной машине. Приложения или процессы, которые запускаются в контейнере, поставляются вместе со всеми необходимыми компонентами и файлами конфигурации. Они не имеют представления о других процессах, выполняющихся вне контейнера.

Узел контейнера предоставляет набор ресурсов, и контейнер будет использовать только их. Контейнер считает, что других ресурсов, помимо предоставленных, не существует, поэтому он не может взаимодействовать с ресурсами, выделенными соседнему контейнеру.

При создании контейнеров Windows и последующей работе с ними пригодятся перечисленные ниже основные понятия.

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. На узле контейнера работает один или несколько контейнеров Windows.

**Образ контейнера.** По мере того как в файловую систему или реестр контейнера вносятся изменения (например, при установке программного обеспечения), они регистрируются в "песочнице". Во многих случаях может потребоваться зарегистрировать это состояние, чтобы применить внесенные изменения при создании новых контейнеров. That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. Этот образ представляет собой среду операционной системы. Образ ОС контейнера невозможно изменить.

**Репозиторий контейнера.** При каждом создании образа контейнера этот образ и его зависимости сохраняются в локальном репозитории. Эти образы можно использовать повторно много раз на узле контейнера. Образы контейнеров также можно хранить в открытом или закрытом реестре (например, Docker Hub), чтобы использовать на многих других узлах контейнеров.

<center>![](media/containerfund.png)</center>

Всем, кто работал с виртуальными машинами, контейнеры может показаться чем-то знакомым. A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. При этом у контейнеров и виртуальных машин совершенно разные технология и принцип работы.

Марк Руссинович (Mark Russinovich), гуру Microsoft Azure, написал [прекрасную статью в блоге](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/), в которой подробно описаны различия.

## Типы контейнеров Windows

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. Эти контейнеры не обеспечивают защиту от неблагоприятных сред, поэтому их не следует использовать для изоляции ненадежного кода. Из-за общего пространства ядра для этих контейнеров требуется одинаковая версия и конфигурация ядра.

**Изоляция Hyper-V**— открывает более широкие возможности изоляции по сравнению с контейнерами Windows Server, так как каждый контейнер запускается в виртуальной машине с высоким уровнем оптимизации. In this configuration, the kernel of the container host is not shared with other containers on the same host. Эти контейнеры предназначены для неблагоприятных сред с несколькими арендаторами и обеспечивают уровень безопасности, аналогичный уровню защиты, обеспечиваемой виртуальной машиной. Так как эти контейнеры не используют то же ядро, что и узел или другие контейнеры в узле, они могут запускать ядра с другими версиями и конфигурациями (в поддерживаемых версиях). Например, все контейнеры Windows в Windows 10 используют изоляцию Hyper-V, чтобы применять версию ядра и конфигурацию Windows Server.

Решение о запуске контейнера в Windows с изоляцией Hyper-V или без нее принимается во время выполнения. Вы можете изначально создать контейнер с изоляцией Hyper-V, а затем во время выполнения запустить его как контейнер Windows Server.

## Что такое Docker?

Читая о контейнерах, вы неизбежно услышите о Docker. Docker— это среда для создания и доставки пакетов образов контейнеров. Этот автоматический процесс создает образы (т. е. шаблоны), которые затем можно запускать в любом месте — в локальной среде, в облаке или на персональном компьютере — в виде контейнера.

<center>![](media/docker.png)</center>

Контейнерами Windows Server, как и любыми другими, можно управлять с помощью [Docker](https://www.docker.com).

## Контейнеры для разработчиков ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## Попробуйте контейнеры Windows Server

Готовы воспользоваться всеми преимуществами контейнеров? Перейдите по ссылкам ниже, чтобы получить инструкции по развертыванию вашего первого контейнера: <br/>
Для пользователей Windows Server: перейдите по ссылке [Краткое руководство для Windows Server](../quick-start/quick-start-windows-server.md). <br/>
Для пользователей Windows 10: перейдите по ссылке [Краткое руководство для Windows 10](../quick-start/quick-start-windows-10.md).

