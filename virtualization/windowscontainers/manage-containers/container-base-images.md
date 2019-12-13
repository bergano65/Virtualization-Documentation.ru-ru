---
title: Базовые образы контейнеров Windows
description: Общие сведения об основных образах контейнеров Windows и их использовании.
keywords: docker, контейнеры, хэши
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909784"
---
# <a name="container-base-images"></a>Базовые образы контейнеров

Windows предлагает четыре базовых образа контейнера, из которых пользователи могут выполнять сборку. Каждый базовый образ имеет разную разновидность ОС Windows, имеет разный объем места на диске и несет другой объем набора API Windows.

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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Создано для приложений .NET Core.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT базовая</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Назначение — создано для приложений IoT.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Обнаружение изображений

Все базовые образы контейнера Windows обнаруживаются через [DOCKER Hub](https://hub.docker.com/_/microsoft-windows-base-os-images). Базовые образы контейнера Windows обслуживаются из [MCR.Microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), реестра контейнеров Майкрософт (мкр). Именно поэтому команды извлечения для базовых образов контейнера Windows выглядят следующим образом:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

МКР не имеет собственного каталога и предназначен для поддержки существующих каталогов, таких как DOCKER Hub. Благодаря глобальному объему работы в Azure и в сочетании с Azure CDN мкр обеспечивает единообразную и быструю процедуру извлечения образа. Клиенты Azure, запускающие рабочие нагрузки в Azure, получают преимущества улучшений производительности в сети, а также тесно интегрируются с мкр (источником для образов контейнеров Майкрософт), Azure Marketplace и расширяющимся числом служб в Azure, предлагающим контейнеры в качестве формата пакета развертывания.

## <a name="choosing-a-base-image"></a>Выбор базового образа

Как выбрать правильный базовый образ, на основе которого будет проводиться построение? Для большинства пользователей `Windows Server Core` и `Nanoserver` будут наиболее подходящими для использования образом.

### <a name="guidelines"></a>Руководство

 Хотя вы не можете ориентироваться на какой бы образ, вот некоторые рекомендации по выбору:

- **Требуется ли приложению полноценная платформа .NET Framework?** Если ответ на этот вопрос имеет значение Да, следует ориентироваться `Windows Server Core`.
- **Вы создаете приложение для Windows на основе .NET Core?** Если ответ на этот вопрос имеет значение Да, следует ориентироваться `Nanoserver`.
- **Вы создаете приложение IoT?** Если ответ на этот вопрос имеет значение Да, следует ориентироваться `IoT Core`.
- **В образе контейнера Windows Server Core отсутствует зависимость, необходимая вашему приложению?** Если ответ на этот вопрос положительный, следует попытаться нацелить `Windows`. Этот образ гораздо больше, чем другие базовые образы, но содержит множество основных библиотек Windows (таких как библиотека GDI).
- **Вы являетесь участником программы предварительной оценки Windows?** Если да, рекомендуется использовать предварительную версию образов. См. раздел "базовые образы для участников программы предварительной оценки Windows" ниже.

> [!TIP]
> Многие пользователи Windows хотят контейнеризовать приложения, имеющие зависимость от .NET. В дополнение к четырем базовым образам, описанным здесь, корпорация Майкрософт публикует несколько образов контейнеров Windows, предварительно настроенных с помощью популярных платформ Майкрософт, таких как образ [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) и образ [ASP .NET](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) .

### <a name="base-images-for-windows-insiders"></a>Базовые образы для участников программы предварительной оценки Windows

Корпорация Майкрософт предоставляет "предварительные" версии каждого базового образа контейнера. Эти образы участников программы предварительной оценки содержат новейшие и лучшие функции разработки в наших образах контейнеров. При запуске узла, который является предварительной версией Windows (Предварительная версия Windows или Windows Server Insider Preview), предпочтительнее использовать эти образы. Образы для участников программы предварительной оценки доступны в центре docker:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Дополнительные сведения см. в статье [Использование контейнеров с программой предварительной оценки Windows](../deploy-containers/insider-overview.md) .

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core VS Server

`Windows Server Core` и `Nanoserver` являются наиболее распространенными базовыми образами. Основное различие между этими образами заключается в том, что в сервере существует значительно меньше поверхности API. PowerShell, WMI и стек обслуживания Windows отсутствуют в образе сервера.

Главный сервер был разработан для предоставления достаточной поверхности API для запуска приложений с зависимостью от .NET Core или других современных платформ с открытым исходным кодом. В качестве компромисса к более мелкой поверхности APi, образ сервера на диске значительно меньше, чем в остальных основных образах Windows. Имейте в виду, что вы всегда можете добавить другие компоненты поверх Nano Server по своему усмотрению. Пример см. здесь: [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
