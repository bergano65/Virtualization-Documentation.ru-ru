---
title: "Тома данных контейнера"
description: "Создание томов данных и управление ими с помощью контейнеров Windows."
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
translationtype: Human Translation
ms.sourcegitcommit: 111a4ca9f5d693cd1159f7597110409d670f0f5c
ms.openlocfilehash: b8eca51e347f17e787095b7e4349337cc3ae69a7

---

# Тома данных контейнера

**Это предварительное содержимое. Возможны изменения.** 

При создании контейнеров может потребоваться создать новый каталог данных или добавить в контейнер существующий каталог. Это можно сделать посредством добавления томов данных. Тома данных видимы как контейнеру, так и узлу контейнера, и между ними возможен обмен данными. Кроме того, тома данных могут совместно использоваться несколькими контейнерами на одном узле контейнера. Этот документ подробно описывает создание, просмотр и удаление томов данных.

## Тома данных

### Создание тома данных

Создайте том данных с помощью параметра `-v` команды `docker run`. По умолчанию новые тома данных хранятся на узле в каталоге "C:\ProgramData\Docker\volumes".

Этот пример создает том данных с именем "new-data-volume". Этот том данных будет доступен в выполняющемся контейнере по пути "C:\new-data-volume".

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

Дополнительные сведения о создании томов см. в описании [управления данными в контейнерах на сайте docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes).

### Подключение существующего каталога

Кроме создания тома данных вам может потребоваться передать существующий каталог с узла в контейнер. Это также можно сделать с помощью параметра `-v` команды `docker run`. Все файлы в каталоге узла также доступны в контейнере. Все файлы, созданные контейнером на подключенном томе, будут доступны на узле. Один каталог можно подключить к нескольким контейнерам. В такой конфигурации возможно совместное использование данных контейнерами.

В этом примере исходный каталог "C:\source" подключается к контейнеру как "C:\destination".

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

Дополнительные сведения о подключении каталогов узла см. в описании [управления данными в контейнерах на сайте docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume).

### Подключение отдельных файлов

Для подключения к контейнеру отдельного файла можно явно указать его имя. В этом примере каталог, находящийся в общем доступе, содержит множество файлов, однако внутри контейнера доступен только файл "config.ini". 

```none
docker run -it -v c:\container-share\config.ini windowsservercore cmd
```

Внутри запущенного контейнера виден только файл config.ini.

```none
c:\container-share>dir
 Volume in drive C has no label.
 Volume Serial Number is 7CD5-AC14

 Directory of c:\container-share

04/04/2016  12:53 PM    <DIR>          .
04/04/2016  12:53 PM    <DIR>          ..
04/04/2016  12:53 PM    <SYMLINKD>     config.ini
               0 File(s)              0 bytes
               3 Dir(s)  21,184,208,896 bytes free
```

Дополнительные сведения о подключении отдельных файлов см. в описании [управления данными в контейнерах на сайте docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume).

### Контейнеры томов данных

Тома данных можно наследовать от других запущенных контейнеров с помощью параметра `--volumes-from` команды `docker run`. Благодаря такому наследованию можно создать специальный контейнер, чтобы размещать тома данных для контейнерных приложений. 

Этот пример подключает тома данных из контейнера "cocky_bell" к новому контейнеру. После запуска нового контейнера данные, находящиеся в этом томе, станут доступны для приложений, выполняемых в контейнере.  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

Дополнительные сведения о контейнерах данных см. в описании [управления данными в контейнерах на сайте docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume).

### Проверка общего тома данных

Подключенные тома можно просмотреть с помощью команды `docker inspect`.

```none
docker inspect backstabbing_kowalevski
```

Она возвращает сведения о контейнере, включая раздел "Mounts" (Подключения) со сведениями о подключенных томах, такими как исходный и целевой каталоги.

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

Дополнительные сведения о проверке томов см. в описании [управления данными в контейнерах на сайте docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume).




<!--HONumber=Jun16_HO4-->


