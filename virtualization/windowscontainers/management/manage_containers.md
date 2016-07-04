---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: 2b85875eae1dcf1e50162e69c53dbf1ac7463450
ms.openlocfilehash: 8921cbd910bf657ddc4998e4214c1e9f9c3a01e9

---

# Управление контейнерами Windows Server

**Это предварительное содержимое. Возможны изменения.** 

Жизненный цикл контейнера включает такие действия, как запуск, остановка и удаление. Кроме того, необходимо получать список образов контейнера, управлять его сетевыми подключениями и ограничивать его ресурсы. В этом документе рассматриваются базовые задачи по управлению контейнерами с помощью Docker, а также приводятся ссылки на более подробные статьи. 

## Управление контейнерами

### Создание контейнера

Используйте команду `docker run`, чтобы создать контейнер с помощью Docker.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Дополнительные сведения о команде `run` Docker см. в [справке по команде "run" Docker]( https://docs.docker.com/engine/reference/run/).

### Прекращает работу контейнера.

Используйте команду `docker stop`, чтобы остановить контейнер с помощью Docker.

```none
PS C:\> docker stop tender_panini

tender_panini
```

В этом примере показано, как остановить все запущенные контейнеры с помощью Docker.

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Удаление контейнера

Чтобы удалить контейнер с помощью Docker, используйте команду `docker rm`.

```none
PS C:\> docker rm prickly_pike

prickly_pike
``` 

Чтобы удалить все контейнеры с помощью Docker:

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

Дополнительные сведения о команде Docker rm см. в [справке по команде Docker rm](https://docs.docker.com/engine/reference/commandline/rm/).



<!--HONumber=Jun16_HO4-->


