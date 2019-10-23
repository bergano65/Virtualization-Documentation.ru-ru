---
title: Базовые образы контейнера Windows
description: Общие сведения об основных изображениях контейнера Windows и о том, когда их использовать.
keywords: docker, контейнеры, хэши
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254274"
---
# <a name="container-base-images"></a>Базовые образы контейнеров

В Windows есть четыре базовых образа контейнеров, из которых пользователи могут выполнять сборку. Каждый базовый образ является другим разновидностью операционной системы Windows, имеет разный объем места на диске и несет другую сумму набора API Windows.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Поддерживает традиционные приложения .NET Framework.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Сервер Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Создано для основных приложений .NET.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Предоставляет полный набор API Windows.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Предназначены для приложений IoT.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Обнаружение изображений

Все базовые образы контейнера Windows можно обнаружить с помощью [стыковочного](https://hub.docker.com/_/microsoft-windows-base-os-images)узла. Базовые изображения контейнера Windows обслуживаются из [MCR.Microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), реестра Microsoft Container (мкр). Вот почему команды Pull для базовых изображений контейнера Windows выглядят следующим образом:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

МКР не имеет собственного интерфейса и предназначен для поддержки существующих каталогов, таких как Dock Hub. Благодаря своему глобальному разприятию Azure и соединению с Azure CDN, мкр обеспечивает единую и быструю распечатку изображений. Клиенты Azure, работающие с рабочими нагрузками в Azure, получают преимущество от производительности в сети, а также тесная интеграция с мкр (источник для изображений контейнера Microsoft), Azure Marketplace и расширение количества служб в Azure, предлагающее контейнеры в качестве формата пакета развертывания.

## <a name="choosing-a-base-image"></a>Выбор базового изображения

Как выбрать подходящего базового образа для сборки? Для большинства пользователей, `Windows Server Core` и `Nanoserver` это будет наиболее подходящее изображение для использования.

### <a name="guidelines"></a>Требования

 Несмотря на то, что вы не можете ориентироваться на нужные изображения, ниже приведены некоторые рекомендации по выбору из них.

- **Нужна ли приложению полная версия .NET Framework?** Если ответ на этот вопрос равен Да, необходимо выбрать `Windows Server Core`.
- **Вы строите приложение для Windows на основе .NET Core?** Если ответ на этот вопрос равен Да, необходимо выбрать `Nanoserver`.
- **Вы строите приложение IoT?** Если ответ на этот вопрос равен Да, необходимо выбрать `IoT Core`.
- **Является ли образ контейнера основных компонентов Windows Server отсутствующим зависимостям для вашего приложения?** Если ответ на этот вопрос равен Да, вы должны попробовать цель `Windows`. Это изображение значительно больше, чем другие базовые изображения, но оно содержит множество основных библиотек Windows (например, библиотеки GDI).
- **Вы являетесь участником программы предварительной оценки Windows?** Если да, следует использовать версию программы предварительной оценки. Ознакомьтесь с разрешениями "базовые образы для участников программы предварительной оценки Windows" ниже.

> [!TIP]
> Многие пользователи Windows хотят контаинеризе приложения, которые имеют зависимость от .NET. В дополнение к четырем основным изображениям, описанным здесь, корпорация Майкрософт публикует несколько изображений контейнера Windows, которые предварительно настроены для распространенных платформ Microsoft Framework, таких как изображение [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) и изображение [ASP .NET](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) .

### <a name="base-images-for-windows-insiders"></a>Базовые образы для участников программы предварительной оценки Windows

Корпорация Майкрософт предоставляет версии для участников программы предварительной оценки для каждого базового образа контейнера. Эти образы участников программы предварительной оценки включают в себя новейшее и более глубокое развитие функций в наших образах. При запуске основного приложения, которое является версией для участников программы предварительной оценки Windows (программа предварительной оценки Windows или Windows Server Insider), предпочтительнее использовать эти образы. Образы участников программы предварительной оценки доступны в центре стыковочного узла:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Чтобы узнать больше, прочтите сведения о [контейнерах с помощью программы предварительной оценки Windows](../deploy-containers/insider-overview.md) .

### <a name="windows-server-core-vs-nanoserver"></a>Основной сервер Windows Server или VS

`Windows Server Core` и `Nanoserver` это самые распространенные базовые образы для целевого объекта. Основное различие между этими изображениями заключается в том, что у сервера в серверном веб-сервере значительно меньше поверхности API. Оболочки PowerShell, WMI и комплект обслуживания Windows отсутствуют в образе сервера.

Сервер в среде конструирования разработан для предоставления достаточной поверхности API для запуска приложений, которые имеют зависимость на ядре .NET или других современных платформах с открытым исходным кодом. В качестве компромисса для более мелкой поверхности APi на диске сервера с небольшим объемом свободного места, чем в остальных графических устройствах Windows. Имейте в виду, что вы всегда можете добавить другие компоненты поверх Nano Server по своему усмотрению. Пример см. здесь: [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
