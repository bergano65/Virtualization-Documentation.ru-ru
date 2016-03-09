



# Общие папки контейнера

**Это предварительное содержимое. Возможны изменения.**

Общие папки позволяют передавать данные из узла контейнера в контейнер и обратно. После создания общая папка будет доступна в контейнере. Данные, помещенные в общую папку с узла, будут доступны в контейнере. Данные, помещенные в общую папку из контейнера, будут доступны на узле. Одна папка на узле может быть общей для многих контейнеров. В этой конфигурации данные могут передаваться между запущенными контейнерами.

## Управление данными — PowerShell

### Создание общей папки

Чтобы создать общую папку, используйте команду `Add-ContainerSharedFolder`. В примере ниже создается каталог в контейнере `c:\shared_data`, который сопоставлен с каталогом на узле `c:\data_source`.

> При добавлении общей папки контейнер должен быть остановлен.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### Общая папка только для чтения

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### Список общих папок

Чтобы просмотреть список общих папок для конкретного контейнера, используйте команду `Get-ContainerSharedFolder`.

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### Изменение общей папки

Чтобы изменить конфигурацию существующей общей папки, используйте команду `Set-ContainerSharedFolder`.

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### Удаление общей папки

Чтобы удалить общую папку, используйте команду `Remove-ContainerSharedFolder`.

> При удалении общей папки контейнер должен быть остановлен.

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## Управление данными — Docker

### Подключение томов

При управлении контейнерами Windows с помощью Docker тома можно подключать с помощью параметра `-v`.

В примере ниже исходная папка — c:\source, а папка назначения — c:\destination.

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

Дополнительные сведения об управлении данными в контейнерах с помощью Docker см. в разделе ["Тома Docker" на сайте Docker.com](https://docs.docker.com/userguide/dockervolumes/).

## Видеоруководство

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-3-Shared-Folders/player" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>



<!--HONumber=Feb16_HO3-->
