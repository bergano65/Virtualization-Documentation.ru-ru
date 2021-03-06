---
title: Часто задаваемые вопросы о контейнерах Windows
description: Вопросы и ответы по контейнерам Windows Server
keywords: docker, контейнеры
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910804"
---
# <a name="frequently-asked-questions-about-containers"></a>Часто задаваемые вопросы о контейнерах

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>В чем разница между контейнерами Linux и Windows Server?

В Linux и Windows Server реализованы аналогичные технологии в ядре и в основных операционных системах. Отличие заключается в платформе и рабочих нагрузках, выполняющихся в контейнерах.  

Когда клиент использует контейнеры Windows Server, они могут интегрироваться с существующими технологиями Windows, такими как .NET, ASP.NET и PowerShell.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Каковы предварительные требования для запуска контейнеров в Windows?

Контейнеры были введены для платформы с Windows Server 2016. Для использования контейнеров требуется Windows Server 2016 или годовщина обновления Windows 10 (версия 1607) или более поздней версии. Дополнительные сведения [см](../deploy-containers/system-requirements.md) . в статье требования к системе.

## <a name="what-are-wcow-and-lcow"></a>Что такое ВКОВ и ЛКОВ?

ВКОВ является коротким для "контейнеров Windows в Windows". ЛКОВ является коротким для "контейнеров Linux в Windows".

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>Как осуществляется лицензирование контейнеров? Существует ли ограничение на количество контейнеров, которые я могу запустить?

[Лицензионное соглашение](../images-eula.md) для образа контейнера Windows описывает использование, которое зависит от пользователя, имеющего действительную лицензированную ОС узла. Количество контейнеров, которые разрешено запускать пользователю, зависит от выпуска ОС узла и [режима изоляции](../manage-containers/hyperv-container.md) , с которым выполняется контейнер, а также от того, выполняются ли эти контейнеры для разработки и тестирования или в рабочей среде.

|ОС узла                                                         |Ограничение изолированных контейнеров процессов                   |Ограничение изолированных контейнеров Hyper-V                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard;                                         |Без ограничений                                          |2                                                  |
|Windows Server Datacenter.                                       |Без ограничений                                          |Без ограничений                                          |
|Windows 10 профессиональная и Корпоративная                                   |Без ограничений *(только для тестов или целей разработки)*|Без ограничений *(только для тестов или целей разработки)*|
|Windows 10 IoT базовая и Корпоративная                             |Неограниченное                                         |Неограниченное                                          |

Использование образа контейнера Windows Server определяется путем считывания количества гостевых виртуальных машин, поддерживаемых для этого [выпуска](/windows-server/get-started-19/editions-comparison-19.md). <br/>

>[!NOTE]
>\*использование рабочих контейнеров в выпуске Windows IoT Edition зависит от того, вы согласились с коммерческими условиями использования образов среды выполнения Windows 10 Core или лицензии на Windows 10 IoT Корпоративная ("коммерческое соглашение Windows IoT"). Дополнительные условия и ограничения для коммерческих соглашений по Windows IoT применяются к использованию образа контейнера в рабочей среде. Ознакомьтесь с [лицензионным соглашением для образа контейнера](../images-eula.md) , чтобы понять, что разрешено и что не так.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Мне как разработчику нужно переписать приложение для каждого типа контейнера?

Нет. Образы контейнеров Windows обычно используются как в контейнерах Windows Server, так и в изоляции Hyper-V. Вы выбираете тип при запуске контейнера. С точки зрения разработчика контейнеры Windows Server и изоляция Hyper-V представляют собой две разновидности одной и той же вещи. Они предлагают одну и ту же среду разработки, программирования и управления, а также являются открытыми и расширяемыми и включают тот же уровень интеграции и поддержки с DOCKER.

Разработчик может создать образ контейнера с помощью контейнера Windows Server и развернуть его в изоляции Hyper-V или наоборот без каких-либо изменений, кроме указания соответствующего флага времени выполнения.

Контейнеры Windows Server обладают большей плотностью и производительностью, когда важна скорость, например, меньшее время вращения и более высокая производительность среды выполнения по сравнению с вложенными конфигурациями. Изоляция Hyper-V, имеющая имя, обеспечивает более широкие возможности изоляции, гарантируя, что код, выполняемый в одном контейнере, не сможет повлиять на работу операционной системы узла или других контейнеров, работающих на том же узле. Это полезно для многоклиентских сценариев с требованиями к размещению ненадежного кода, включая приложения SaaS и размещение вычислений.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Можно ли запускать контейнеры Windows в режиме изолированной обработки в Windows 10?

Начиная с обновления Windows 10 октября с обновлением 2018 можно запустить контейнер Windows с изоляцией процесса, но сначала необходимо напрямую запрашивать изоляцию процесса с помощью флага `--isolation=process` при запуске контейнеров с `docker run`. Изоляция процессов совместима в Windows 10 Pro, Windows 10 Корпоративная, Windows 10 IoT Core и Windows 10 IoT Корпоративная.

Если вы хотите выполнять контейнеры Windows таким образом, необходимо убедиться, что узел работает под управлением Windows 10 Build 17763 + и у вас установлена версия DOCKER с ядром 18,09 или более поздней.

> [!WARNING]
> Эта функция предназначена только для разработки и тестирования, но не на узлах Интернета вещей Core и Интернета вещей предприятия (после принятия дополнительных условий и ограничений). Следует продолжать использовать Windows Server в качестве узла для развертывания в рабочей среде. С помощью этой функции необходимо также убедиться, что теги версии узла и контейнера совпадают, в противном случае контейнер может не запуститься или открыть неопределенное поведение.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Разделы справки сделать образы контейнеров доступными на машинах гаппед?

Базовые образы контейнера Windows содержат артефакты, распределение которых ограничено лицензией. Когда вы создаете эти образы и отправляете их в закрытый или общедоступный реестр, вы заметите, что базовый уровень никогда не будет отправлен. Вместо этого мы используем концепцию внешнего слоя, который указывает на реальный базовый уровень, находящийся в облачном хранилище Azure.

Это может усложнить работу при наличии компьютера Air гаппед, который может получать образы только из адреса закрытого реестра контейнеров. В этом случае попытки выполнить внешний слой для получения базового образа не будут работать. Чтобы переопределить поведение внешнего слоя, можно использовать флаг `--allow-nondistributable-artifacts` в управляющей программе DOCKER.

> [!IMPORTANT]
> Использование этого флага не исключает обязательства в соответствии с условиями лицензии на базовый образ контейнера Windows; не следует публиковать содержимое Windows для распространения общедоступного или стороннего производителя. Использование в собственной среде разрешено.

## <a name="additional-feedback"></a>Дополнительный отзыв

Хотите добавить что-нибудь в часто задаваемые вопросы? Откройте новую ошибку отзыва в разделе комментариев или настройте запрос на включение внесенных изменений для этой страницы с помощью GitHub.
