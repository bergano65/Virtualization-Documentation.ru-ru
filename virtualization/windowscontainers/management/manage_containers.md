---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
---

# Управление контейнерами Windows Server

<g id="1" ctype="x-strong">Это предварительное содержимое. Возможны изменения.</g>

Жизненный цикл контейнера включает такие действия, как запуск, остановка и удаление. Кроме того, необходимо получать список образов контейнера, управлять его сетевыми подключениями и ограничивать его ресурсы. В этом документе рассматриваются базовые задачи по управлению контейнерами с помощью Docker, а также приводятся ссылки на более подробные статьи.

## Управление контейнерами

### Создание контейнера

Используйте команду <g id="2" ctype="x-code">docker run</g>, чтобы создать контейнер с помощью Docker.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Дополнительные сведения о команде Docker <g id="2" ctype="x-code">run</g> см. в <g id="4CapsExtId1" ctype="x-link"><g id="4CapsExtId2" ctype="x-linkText">справке по команде Docker run</g><g id="4CapsExtId3" ctype="x-title"></g></g>.

### Прекращает работу контейнера.

Используйте команду <g id="2" ctype="x-code">docker stop</g>, чтобы остановить контейнер с помощью Docker.

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

Чтобы удалить контейнер с помощью Docker, используйте команду <g id="2" ctype="x-code">docker rm</g>.

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

Дополнительные сведения о команде Docker rm см. в <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">справке по команде Docker rm</g><g id="2CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=Apr16_HO5-->


