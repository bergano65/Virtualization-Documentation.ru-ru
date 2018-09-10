# <a name="linux-containers"></a>Контейнеры Linux

Эта функция использует [изоляцию Hyper-V](../manage-containers/hyperv-container.md) для выполнения ядра Linux с необходимыми компонентами ОС для поддержки контейнеров. Изменения Windows и Hyper-V для реализации этой функции начались с _Windows 10 осенью Creators Update_ и _Windows Server версии 1709_, но для объединения всех возможностей потребовалось поработать [проектом Moby](https://www.github.com/moby/moby) с открытым исходным кодом, на котором основана технология Docker, а также с ядром Linux. 

[Видео о контейнеров Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

Для работы с этой функцией вам понадобятся следующие документы:

- Windows 10 или Windows Server Insider Preview, сборка 16267 или более поздняя версия;
- сборка управляющей программы Docker на базе основной ветви Moby с флагом `--experimental`;
- совместимый образ Linux по вашему выбору.

Для этой предварительной версии доступны руководства по началу работы:

- [Docker Enterprise Edition Preview](https://blog.docker.com/2017/09/docker-windows-server-1709/) включает в себя систему LinuxKit и предварительную версию Docker EE для запуска контейнеров Linux. Дополнительные сведения см. в статье [Предварительная версия контейнеров Linux в Windows с использованием LinuxKit](https://go.microsoft.com/fwlink/?linkid=857061)
- [Запуск контейнеров Ubuntu с изоляцией Hyper-V в Windows 10 и Windows Server](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>Проблемы и их решение

Текущий ход разработки проекта Moby можно отслеживать на портале [GitHub](https://github.com/moby/moby/issues/33850)


### <a name="known-app-issues"></a>Известные проблемы приложений

Для всех этих приложений требуется сопоставление томов, ряд ограничений которых описаны в разделе [Привязки подключений](#Bind-mounts). Они не будут запускаться или будут работать неправильно.

- MySQL
- PostgreSQL
- WordPress
- Jenkins
- MariaDB
- RabbitMQ


### <a name="bind-mounts"></a>Привязки подключений

Тома с привязкой подключения с использованием `docker run -v ...` хранят файлы в файловой системе Windows NTFS, поэтому для операций POSIX потребуется определенная корректировка. Некоторые операции файловой системы в настоящий момент реализованы частично или не реализованы, что может вызывать проблемы с совместимостью для некоторых приложений.

Эти операции в настоящий момент не работают для томов с привязкой подключения:

- MkNod
- XAttrWalk
- XAttrCreate
- Lock
- Getlock
- Auth
- Flush
- INotify

Ряд операций не были полностью реализованы:

- GetAttr— число Nlink всегда равно 2.
- Open— реализованы только флаги ReadWrite, WriteOnly и ReadOnly.
