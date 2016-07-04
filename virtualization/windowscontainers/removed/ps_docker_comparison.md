---
author: scooley
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: e56aa08067fb18caa28224ddc0577a677f09ded3
ms.openlocfilehash: 5a10afe0f0adcfa86fe9776efa45cfb935ca1beb

---


# Сравнение Docker и PowerShell как средств управления контейнерами Windows

Управлять контейнерами Windows можно с помощью средств, поставляемых с Windows (в этой предварительной версии — PowerShell), и средств с открытым исходным кодом, например Docker.  
Отдельные руководства по каждому из этих средств можно найти здесь:
* [Управление контейнерами Windows с помощью Docker](../quick_start/manage_docker.md)
* [Управление контейнерами Windows с помощью PowerShell](../quick_start/manage_powershell.md) 

В этой статье основное внимание уделяется сравнению средств управления Docker и PowerShell.

## PowerShell для контейнеров и виртуальные машины Hyper-V
Создавать и запускать контейнеры Windows, а также взаимодействовать с ними можно с помощью командлетов PowerShell. Все, что вам нужно для работы, уже встроено в систему.

Если вы использовали PowerShell для Hyper-V, то структура командлетов должна быть вам знакома. Рабочий процесс в целом аналогичен управлению виртуальной машиной с помощью модуля Hyper-V. Вместо `New-VM`, `Get-VM`, `Start-VM` и `Stop-VM` у вас есть `New-Container`, `Get-Container`, `Start-Container` и `Stop-Container`.  Существует довольно много командлетов и параметров для контейнеров, но управление контейнером Windows и виртуальной машиной Hyper-V, а также их общий жизненный цикл очень похожи.

## Сравнение средств управления PowerShell и Docker 
Командлеты PowerShell для контейнеров предоставляют API, который немного отличается от API Docker. Как правило, командлеты более специализированы. Некоторые команды Docker имеют довольно явные параллели с командлетами PowerShell:

| Команда Docker |  Командлет PowerShell |
|----|----|
| `docker ps -a`    | `Get-Container` |
| `docker images`   | `Get-ContainerImage` |
| `docker rm`   | `Remove-Container` |
| `docker rmi` | `Remove-ContainerImage` |
| `docker create`   | `New-Container` |
| `docker commit <container ID>` | `New-ContainerImage -Container <container>` |
| `docker load <tarball>` | `Import-ContainerImage <AppX package>` |
| `docker save` |   `Export-ContainerImage` |
| `docker start` |  `Start-Container` |
| `docker stop` |   `Stop-Container` |

Командлеты PowerShell не являются точными аналогами команд Docker. Кроме того, у некоторых команд Docker нет аналогов в PowerShell* (например, `docker build` и `docker cp`). Но сразу бросается в глаза то, что нет однострочного аналога для команды `docker run`.

\* Может измениться.

### Но что делать, если мне нужна команда docker run?  
Мы стремимся сделать модель взаимодействия чуть более привычной для пользователей, которые уже знакомы с работой в PowerShell. Конечно, если вы привыкли к работе в Docker, вам это покажется несколько нестандартным.

1.  Жизненный цикл контейнера в модели PowerShell немного отличается. В модуле "Контейнеры" PowerShell мы предоставляем более специализированные операции: `New-Container` (создает новый остановленный контейнер) и `Start-Container`.
  
  Между созданием и запуском контейнера можно также настроить его параметры. Кроме того, в версии Technical Preview 3 мы планируем предоставить возможность установки сетевого подключения для контейнера с помощью командлетов (Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter.

2.  В настоящее время невозможно передать команду в контейнер при запуске. Тем не менее вы все равно можете создать интерактивный сеанс PowerShell с запущенным контейнером с помощью `Enter-PSSession -ContainerId <ID of a running container>` и выполнить команду внутри контейнера с помощью `Invoke-Command -ContainerId <container id> -ScriptBlock { code to run inside the container }` или `Invoke-Command -ContainerId <container id> -FilePath <path to script>`.  
С обеими командами можно использовать необязательный флаг `-RunAsAdministrator` для повышения привилегий.  


## Предупреждения и известные проблемы
1.  В настоящее время командлеты для контейнеров не работают с контейнерами и образами, созданными с помощью Docker. Так же и команды Docker не взаимодействуют с контейнерами и образами, созданными с помощью PowerShell. Управлять контейнером следует с помощью того средства, в котором он был создан — или Docker, или PowerShell.

2.  Мы запланировали много улучшений, чтобы сделать работу пользователей более удобной. Они коснутся сообщений об ошибках, отчетов о ходе выполнения, недопустимых строк событий и т. д. Если вы столкнулись с ситуацией, когда вам не хватает необходимой информации, оставьте свои предложения на форумах.

## Краткий обзор
Рассмотрим некоторые из стандартных рабочих процессов.

Предполагается, что вы установили образ контейнера ОС с именем ServerDatacenterCore и создали виртуальный коммутатор с именем Virtual Switch (с помощью командлета New-VMSwitch).

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### Создание собственного образца
Для просмотра всех командлетов модуля "Контейнеры" воспользуйтесь командой `Get-Command -Module Containers`.  Некоторые командлеты здесь не описаны. Узнать о них вы можете самостоятельно.    
**Примечание**. Эта команда не возвращает командлеты `Enter-PSSession` и `Invoke-Command`, которые относятся к базовым командлетам PowerShell.

Кроме того, вы можете получить справку по любому командлету с помощью команды `Get-Help [cmdlet name]` или эквивалентной команды `[cmdlet name] -?`.  В настоящее время справка создается автоматически и просто содержит синтаксис команд. Мы будем добавлять дополнительную документацию по мере доработки структуры командлетов.

Нагляднее синтаксис представлен в интегрированной среде сценариев PowerShell, с которой вы, возможно, не сталкивались, если не использовали активно PowerShell. Если ваша система позволяет, запустите интегрированную среду сценариев, откройте панель команд и выберите модуль "Контейнеры", в нем вы найдете графическое представление командлетов и наборов их параметров.

P.S. Чтобы доказать осуществимость такого сценария, приведем функцию PowerShell, которая состоит из некоторых командлетов, уже известных нам, и может заменить команду `docker run`. (Уточним, что этот пример доказывает саму возможность замены, которая пока не находится в активной разработке.)

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker
Контейнерами Windows можно управлять с помощью команд Docker.  Контейнеры Windows и Linux аналогичны и имеют одни и те же принципы управления в Docker, но некоторые команды Docker просто не применимы к контейнерам Windows.  Другие пока даже не тестировались (планируется в ближайшем будущем).

Чтобы не дублировать документацию по API, доступную в Docker, <a href="https://docs.docker.com/engine/reference/commandline/cli/" >приведем</a> ссылку на интерфейсы API управления.  этих API управления.

В настоящее время мы работаем над документом "Проблемы и их решение", в котором рассматриваются возможности и ограничения работы с API Docker.



<!--HONumber=Jun16_HO4-->


