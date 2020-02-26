# <a name="build-and-run-an-application-with-or-without-net-core-20-or-powershell-core-6"></a>Сборка и запуск приложения с компонентами .NET Core 2.0 и PowerShell Core 6 или без них

Компоненты .NET Core и PowerShell были удалены из этого выпуска базового образа контейнера ОС Nano Server, однако .NET Core и PowerShell поддерживаются как дополнительный контейнер многоуровневый контейнер поверх базового контейнера Nano Server.  

Если контейнер будет запускать неуправляемый код или открытые платформы, такие как Node.js, Python, Ruby и т. д, базового контейнера Nano Server будет достаточно.  Одна из особенностей состоит в том, что определенный неуправляемый код может не выполняться из-за [экономии занимаемого места](https://docs.microsoft.com/windows-server/get-started/nano-in-semi-annual-channel) в этом выпуске по сравнению с Windows Server 2016. Если вы заметили какие-либо проблемы, сообщите нам на [форумах](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers). 

Чтобы создать контейнер из файла Dockerfile, используйте команду docker build, а чтобы запустить его — docker run.  Следующая команда скачивает базовый образ ОС контейнера Nano Server, что может занять несколько минут, а также выводит текст «Hello World»! в консоли узла.

```
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

Вы можете создавать более сложные приложения с помощью [файлов Dockerfile в Windows](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile), используя такой синтаксис Dockerfile, как FROM, RUN, COPY, ADD, CMD и т. д. Хотя выполнить определенные команды сразу из этого базового образа не удастся, теперь вы сможете создавать образы контейнеров, которые содержат только то, что необходимо для работы вашего приложения.

Так как компоненты .NET Core PowerShell недоступны в базовом образе ОС контейнера Nano Server, одна из проблем заключается в создании контейнера с содержимым в сжатом формате ZIP. С помощью функции [многоэтапной сборки](https://docs.docker.com/engine/userguide/eng-image/multistage-build/), доступной в Docker 17.05, вы можете использовать PowerShell в другом контейнере, чтобы извлечь содержимое и скопировать его в контейнер Nano. Этот подход можно применять для создания контейнера .NET Core и PowerShell. 

Вы можете извлечь образ контейнера PowerShell с помощью этой команды:

```
docker pull microsoft/nanoserver-insider-powershell
```

Извлечь образ контейнера .NET Core можно с помощью следующей команды:

```
docker pull microsoft/nanoserver-insider-dotnet
```

Ниже приведены некоторые примеры использования многоэтапных сборок для создания этих образов контейнеров.

## <a name="deploy-apps-based-on-net-core-20"></a>Развертывание приложений на основе .NET Core 2.0
Вы можете использовать образ контейнера .NET Core для запуска приложений .NET Core для участников программы предварительной оценки, если сборка приложения .NET Core выполнена в другом месте, и вы хотите запустить его в контейнере.  Дополнительные сведения о запуске приложения .NET Core с образами контейнеров .NET Core см. в разделе [.NET Core GitHub](https://github.com/dotnet/dotnet-docker-nightly).  Если вы создаете приложение внутри контейнера, следует использовать пакет .NET Core SDK.  Опытные пользователи могут создать собственный контейнер .NET Core 2.0 с помощью версии .NET Core 2.0, Dockerfile и URL-адреса, указанного в свойстве [dotnet-docker-nightly](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0). Для этого можно использовать контейнер Windows Server Core для скачивания и распаковки данных.  Пример файла Dockerfile — такой же, как в разделе [Файл Dockerfile среды выполнения .NET Core](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/amd64/Dockerfile).


С помощью этого файла Dockerfile контейнер .NET Core 2.0 можно создать, выполнив следующую команду.

```
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## <a name="run-powershell-core-6-in-a-container"></a>Запуск PowerShell Core 6 в контейнере
С помощью такого же метода [многоэтапной сборки](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) можно создать контейнер PowerShell Core 6, используя [этот файл PowerShell Dockerfile](https://github.com/PowerShell/PowerShell-Docker/blob/master/release/stable/nanoserver/docker/Dockerfile).


Затем выполните команду docker build, чтобы создать образ контейнера PowerShell.

``` 
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

Дополнительные сведения см. в разделе [PowerShell GitHub](https://github.com/PowerShell/PowerShell-Docker/tree/master/release).  Стоит отметить, что ZIP-файл PowerShell содержит подмножество .NET Core 2.0, необходимое для построения PowerShell Core 6.  Если ваши модули PowerShell зависят от .NET Core 2.0, можно создать контейнер PowerShell поверх контейнера Nano .NET Core вместо базового контейнера Nano, т. е. 

## <a name="next-steps"></a>Следующие шаги
- Используйте один из новых образов контейнеров на основе Nano Server, доступных в Docker Hub, т. е. базовый образ Nano Server, Nano с .NET Core 2.0 и Nano с PowerShell Core 6
- Создайте собственный образ контейнера на основе нового базового образа ОС контейнера Nano Server, используя пример файла Dockerfile из этого руководства 
