---
author: scooley
redirect_url: ../quick_start/manage_docker
---


# Сравнение Docker и PowerShell как средств управления контейнерами Windows

Управлять контейнерами Windows можно с помощью средств, поставляемых с Windows (в этой предварительной версии — PowerShell), и средств с открытым исходным кодом, например Docker.  
Отдельные руководства по каждому из этих средств можно найти здесь:
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Управление контейнерами Windows с помощью Docker</g><g id="1CapsExtId3" ctype="x-title"></g></g>
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Управление контейнерами Windows с помощью PowerShell</g><g id="1CapsExtId3" ctype="x-title"></g></g>

В этой статье основное внимание уделяется сравнению средств управления Docker и PowerShell.

## PowerShell для контейнеров и виртуальные машины Hyper-V

Создавать и запускать контейнеры Windows, а также взаимодействовать с ними можно с помощью командлетов PowerShell. Все, что вам нужно для работы, уже встроено в систему.

Если вы использовали PowerShell для Hyper-V, то структура командлетов должна быть вам знакома. Рабочий процесс в целом аналогичен управлению виртуальной машиной с помощью модуля Hyper-V. Вместо командлетов <g id="2" ctype="x-code">New-VM</g>, <g id="4" ctype="x-code">Get-VM</g>, <g id="6" ctype="x-code">Start-VM</g> и <g id="8" ctype="x-code">Stop-VM</g> используются командлеты <g id="10" ctype="x-code">New-Container</g>, <g id="12" ctype="x-code">Get-Container</g>, <g id="14" ctype="x-code">Start-Container</g> и <g id="16" ctype="x-code">Stop-Container</g>. Существует довольно много командлетов и параметров для контейнеров, но управление контейнером Windows и виртуальной машиной Hyper-V, а также их общий жизненный цикл очень похожи.

## Сравнение средств управления PowerShell и Docker

Командлеты PowerShell для контейнеров предоставляют API, который немного отличается от API Docker. Как правило, командлеты более специализированы. Некоторые команды Docker имеют довольно явные параллели с командлетами PowerShell:

| Команда Docker| Командлет PowerShell|
|----|----|
| <g id="1" ctype="x-code">docker ps -a</g>| <g id="1" ctype="x-code">Get-Container</g>|
| <g id="1" ctype="x-code">docker images</g>| <g id="1" ctype="x-code">Get-ContainerImage</g>|
| <g id="1" ctype="x-code">docker rm</g>| <g id="1" ctype="x-code">Remove-Container</g>|
| <g id="1" ctype="x-code">docker rmi</g>| <g id="1" ctype="x-code">Remove-ContainerImage</g>|
| <g id="1" ctype="x-code">docker create</g>| <g id="1" ctype="x-code">New-Container</g>|
| <g id="1" ctype="x-code">docker commit <container ID></g>| <g id="1" ctype="x-code">New-ContainerImage -Container &lt;container&gt;</g>|
| <g id="1" ctype="x-code">docker load &lt;tarball&gt;</g>| <g id="1" ctype="x-code">Import-ContainerImage <AppX package></g>|
| <g id="1" ctype="x-code">docker save</g>| <g id="1" ctype="x-code">Export-ContainerImage</g>|
| <g id="1" ctype="x-code">docker start</g>| <g id="1" ctype="x-code">Start-Container</g>|
| <g id="1" ctype="x-code">docker stop</g>| <g id="1" ctype="x-code">Stop-Container</g>|

Командлеты PowerShell не являются точными аналогами. Кроме того, у некоторых команд нет аналогов в PowerShell* (например, <g id="2" ctype="x-code">docker build</g> и <g id="4" ctype="x-code">docker cp</g>). Но сразу бросается в глаза то, что нет однострочного аналога для команды <g id="2" ctype="x-code">docker run</g>.

\* Может измениться.

### Но что делать, если мне нужна команда docker run?

Мы стремимся сделать модель взаимодействия чуть более привычной для пользователей, которые уже знакомы с работой в PowerShell. Конечно, если вы привыкли к работе в Docker, вам это покажется несколько нестандартным.

1.  Жизненный цикл контейнера в модели PowerShell немного отличается. В модуле PowerShell для контейнеров мы предоставляем более специализированные командлеты <g id="2" ctype="x-code">New-Container</g> (создает остановленный контейнер) и <g id="4" ctype="x-code">Start-Container</g>.

  Между созданием и запуском контейнера можно также настроить его параметры. Кроме того, в версии Technical Preview 3 мы планируем предоставить возможность установки сетевого подключения для контейнера с помощью командлетов (Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter.

2.  В настоящее время невозможно передать команду в контейнер при запуске. Однако можно начать интерактивный сеанс PowerShell с запущенным контейнером с помощью команды <g id="2" ctype="x-code">Enter-PSSession -ContainerId <идентификатор запущенного контейнера></g>, а затем выполнить в нем команду с помощью команды <g id="4" ctype="x-code">Invoke-Command -ContainerId <идентификатор контейнера> -ScriptBlock { код, запускаемый внутри контейнера }</g> или <g id="6" ctype="x-code">Invoke-Command -ContainerId <идентификатор контейнера> -FilePath <путь к сценарию></g>.  
С обеими командами можно использовать необязательный флаг <g id="2" ctype="x-code">-RunAsAdministrator</g> для действий с повышенными правами.


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

Все командлеты для контейнеров можно просмотреть с помощью команды <g id="2" ctype="x-code">Get-Command -Module Containers</g>. Некоторые командлеты здесь не описаны. Узнать о них вы можете самостоятельно.    
<g id="1" ctype="x-strong">Примечание.</g> Эта команда не возвращает командлеты <g id="3" ctype="x-code">Enter-PSSession</g> и <g id="5" ctype="x-code">Invoke-Command</g>, которые являются частью базы PowerShell.

Кроме того, справку по командлету можно получить с помощью команды <g id="2" ctype="x-code">Get-Help [имя командлета]</g> или ее эквивалента <g id="4" ctype="x-code">[имя командлета] -?</g>. В настоящее время справка создается автоматически и просто содержит синтаксис команд. Мы будем добавлять дополнительную документацию по мере доработки структуры командлетов.

Нагляднее синтаксис представлен в интегрированной среде сценариев PowerShell, с которой вы, возможно, не сталкивались, если не использовали активно PowerShell. Если ваша система позволяет, запустите интегрированную среду сценариев, откройте панель команд и выберите модуль "Контейнеры", в нем вы найдете графическое представление командлетов и наборов их параметров.

P.S. Чтобы доказать осуществимость такого сценария, приведем функцию PowerShell, которая состоит из некоторых командлетов, уже известных нам, и может заменить команду <g id="2" ctype="x-code">docker run</g>. (Уточним, что этот пример доказывает саму возможность замены, которая пока не находится в активной разработке.)

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

Контейнерами Windows можно управлять с помощью команд Docker. Контейнеры Windows и Linux аналогичны и имеют одни и те же принципы управления в Docker, но некоторые команды Docker просто не применимы к контейнерам Windows. Другие пока даже не тестировались (планируется в ближайшем будущем).

Чтобы не дублировать документацию по API, доступную в Docker, <g id="2" ctype="x-html"></g>приведем<g id="4" ctype="x-html"></g> ссылку на API-интерфейсы управления. этих API управления.

В настоящее время мы работаем над документом "Проблемы и их решение", в котором рассматриваются возможности и ограничения работы с API Docker.






<!--HONumber=Apr16_HO4-->


