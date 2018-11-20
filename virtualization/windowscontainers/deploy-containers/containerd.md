---
title: Создание контейнера стека
description: Дополнительные сведения о новых контейнера сборке блоков, доступных в Windows.
keywords: LCOW, контейнеры linux, docker, контейнеры, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 970de62c9a0011fa09d6741b2665479efd394313
ms.sourcegitcommit: 166aa2430ea47d7774392e65a9875501f86dd5ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/20/2018
ms.locfileid: "7460580"
---
# <a name="container-platform-tools-on-windows"></a>Инструменты для платформы контейнера в Windows

Развернув платформе контейнера Windows!  Docker была первая часть путешествия контейнера, теперь мы создаете другие средства платформы контейнера.

1. [containerd/cri](https://github.com/containerd/cri) - новое в Windows Server 2019 или Windows 10 1809.
1. [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - аналогом узел контейнера Windows runc.
1. [hcs](https://docs.microsoft.com/virtualization/api/) - службы контейнера узлов + удобной оболочками, чтобы упростить использование.

    * [hcsshim](https://github.com/microsoft/hcsshim)
    * [DotNet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

В этой статье поговорим о платформе контейнера Windows и Linux, а также каждого инструмента платформы контейнера.

## <a name="windows-and-linux-container-platform"></a>Контейнер платформы Windows и Linux

В средах, Linux контейнер средств управления, таких как Docker встроены в более тщательный набор средств контейнера — [runc](https://github.com/opencontainers/runc) и [containerd](https://containerd.io/).

![Архитектура docker в Linux](media/docker-on-linux.png)

`runc` — Это средство командной строки Linux для создания и запущенные контейнеры согласно [спецификации среды выполнения контейнера OCI](https://github.com/opencontainers/runtime-spec).

`containerd` является управляющей программы, который управляет жизненного цикла контейнера загружать и распаковке образа контейнера с помощью выполнения контейнера и контроля со стороны клиентов.

В Windows мы воспользовались другой подход.  Когда мы начали работать с Docker для поддержки контейнеров Windows, мы создали непосредственно на HCS (службы контейнера узлов).  [В этой записи блога](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/) заполнен информации о том, почему мы создали HCS и почему мы воспользовались этот подход к контейнерам изначально.

![Начальный архитектура подсистема Docker в Windows](media/hcs.png)

На этом этапе Docker по-прежнему вызывает непосредственно в HCS. В перспективе, тем не менее, средства управления контейнера, развернув контейнеров Windows и Windows, узел контейнера может вызвать containerd и runhcs так, как они вызывают containerd и runc в Linux.

## <a name="runhcs"></a>runhcs

`runhcs` — это компонент из `runc`.  Например, `runc`, `runhcs` — клиент командной строки для запуска приложений, упакованных в соответствии с форматом инициативе Open Initiative (Container) и является реализацией требованиям спецификации откройте Initiative контейнера.

Функциональные различия между runc и runhcs включают в себя:

* `runhcs` выполняется в Windows.  Обменивается данными с [HCS](containerd.md#hcs) для создания и управления контейнерами.
* `runhcs` можно выполнить различные типы другой контейнер.

  * Windows и Linux [контейнеры Hyper-V](../manage-containers/hyperv-container.md)
  * Windows обработать контейнеры (образа контейнера должна соответствовать узлу контейнера)

**Использование:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` — Это имя для экземпляра контейнера, которые вы начинаете работу. Имя должно быть уникальным в узле контейнера.

Каталога набора (с помощью `-b bundle`) не является обязательным.  
Как и в случае с runc, контейнеры настраиваются с помощью пакетов приложений. Пакет контейнера — это каталог с файлом спецификации OCI контейнера, «config.json».  Значение по умолчанию для «bundle» — это текущий каталог.

Этот файл спецификации OCI, «config.json», должен иметь два поля для правильного запуска:

1. Путь к пространству контейнера временных файлов
1. Путь к каталогу уровня контейнера

Контейнер команды, доступные в runhcs включают:

* Средства для создания и запуска контейнера
  * **запустите** создает и запускает контейнера
  * **Создание** создать контейнер

* Средства для управления процессы, выполняемые в контейнере:
  * **запустите** выполняет процесс определяется пользователем в созданный контейнер
  * **exec** запускает новый процесс внутри контейнера
  * Пауза **Приостановить** приостанавливает все процессы внутри контейнера
  * **Возобновление работы** возобновляет все процессы, которые ранее были приостановлены
  * **PS** ps Отображает процессы, выполняемые в контейнере

* Средства для управления состоянием контейнера
  * **состояние** выводит состояние контейнера
  * **завершить** отправляет указанного сигнала (по умолчанию: сигнал SIGTERM) для процесса инициализации контейнера
  * **Удалить** удаляет все ресурсы, занятые контейнера, часто используется с отсоединенной контейнера

**Список**является единственным команда, которая можно рассматривать как несколькими контейнерами.  В нем перечислены работы (или приостановки) контейнеров, запущено с помощью runhcs с заданным корнем.

### <a name="hcs"></a>СЛУЖБЫ HCS

У нас есть две программы-оболочки доступны на GitHub для интерфейса с HCS. Так как HCS C API, программы-оболочки облегчают вызова HCS из верхнего уровня языков.  

* [hcsshim](https://github.com/microsoft/hcsshim) - HCSShim записываются в Go и он является основой для runhcs.
Взять последнюю версию из AppVeyor или выполнять ее сборку самостоятельно.
* [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization является оболочкой для HCS C#.

Если вы хотите использовать HCS (напрямую или через программу-оболочку), или вы хотите сделать нет ли ржавчины или/Haskell/InsertYourLanguage оболочки HCS, оставьте комментарий.

Для более подробно рассмотрим HCS презентация [John Stark DockerCon](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>containerd/cri

> ! CRI Примечание поддерживается только в Server 2019 или Windows 10 1809 и более поздних версиях.

Хотя OCI спецификации определяет один контейнер, [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (интерфейс среды выполнения контейнера) описывает контейнеры как workload(s) в песочнице в общей среде под названием pod.  Модули POD может содержать один или несколько рабочих нагрузок контейнера.  Модули позволяют оркестратора контейнеров, такие как Kubernetes и сетки структуры службы обработки сгруппированных рабочих нагрузок, которые должны находиться на одном узле с некоторыми общими ресурсами, например памяти и vNETs.

Ссылки в спецификации CRI:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - спецификации модуля
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - спецификации рабочей нагрузки

![Средах контейнера на основе Containerd](media/containerd-platform.png)

Хотя runHCS и containerd можно управлять в любой системе Windows Server 2016 или более поздней версии, поддержка модули POD (группы контейнеров) требуется критические изменения контейнером средств в Windows.  Поддержка CRI доступны в Windows Server 2019 или Windows 10 1809 и более поздних версиях.