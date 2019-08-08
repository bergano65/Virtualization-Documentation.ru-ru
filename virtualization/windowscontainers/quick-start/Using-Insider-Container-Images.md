
# <a name="using-insider-container-images"></a>Использование образов контейнеров программы предварительной оценки

В этом упражнении вы узнаете, как развернуть и использовать контейнеры Windows в последней сборке Windows Server для участников программы предварительной оценки Windows. Во время этого упражнения вы установите роль контейнера и развернете предварительный выпуск базовых образов ОС. Если вам необходимо ознакомиться с контейнерами, изучите раздел [О контейнерах](../about/index.md).

В этом кратком руководстве рассматриваются контейнеры Windows Server в программе предварительной оценки Windows Server. Ознакомьтесь с программой перед продолжением работы с этим кратким руководством.

## <a name="prerequisites"></a>Необходимые условия

- Пользователь должен быть участником [программы предварительной оценки Windows](https://insider.windows.com/GettingStarted) и должен изучить условия ее использования.
- Одна компьютерная система (физическая или виртуальная) с последней сборкой Windows Server из программы предварительной оценки Windows и (или) последней сборкой Windows 10 из программы предварительной оценки Windows.

> [!IMPORTANT]
> Вы должны использовать сборку Windows Server из программы предварительной оценки Windows Server или для сборки Windows 10 из программы предварительной оценки Windows, чтобы использовать базовое изображение, описанное ниже. Если у вас не установлена одна из этих сборок, использование этих базовых образов приведет к сбою запуска контейнера.

## <a name="install-docker-enterprise-edition-ee"></a>Установка Docker Enterprise Edition (EE)

Docker EE необходим для работы с контейнерами Windows. Docker EE состоит из подсистемы Docker и клиента Docker.

Для установки Docker EE будет использоваться модуль PowerShell поставщика OneGet. Поставщик обеспечит работу контейнеров на компьютере и установит Docker EE. После этого потребуется перезагрузка. Откройте сеанс PowerShell с повышенными правами и выполните следующие команды.

> [!NOTE]
> Для установки дока в сборках программы предварительной оценки Windows Server требуется иной поставщик Онежет, чем тот, который используется для сборок, не являющихся участниками программы предварительной оценки. Если Docker EE и поставщик DockerMsftProvider OneGet уже установлены, удалите их перед продолжением.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Установите модуль OneGet PowerShell для использования в сборках программы предварительной оценки Windows.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

С помощью OneGet установите последнюю версию Docker EE Preview.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

После завершения установки перезагрузите компьютер.

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>Установка базового образа контейнера

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Вы как участник программы предварительной оценки Windows также можете протестировать наши последние сборки для базовых образов. Сейчас доступно четыре базовых образа, основанных на Windows Server. Сведения о том, для каких целей следует использовать каждый из них, см. в таблице ниже.

| Базовый образ ОС                       | Использование                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | Рабочая среда и разработка |
| mcr.microsoft.com/windows/nanoserver              | Рабочая среда и разработка |
| mcr.microsoft.com/windows/servercore/insider | Только разработка           |
| mcr.microsoft.com/windows/nanoserver/insider        | Только разработка           |

Чтобы получить базовый образ Nano Server для участников программы предварительной оценки, выполните следующую команду:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Чтобы получить базовый образ Windows Server Core для участников программы предварительной оценки, выполните следующую команду:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Пожалуйста, прочтите [лицензионное соглашение](../EULA.md ) на использование образа ОС для контейнеров Windows и [условия использования](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)программы предварительной оценки Windows.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание и выполнение примера приложения](./Nano-RS3-.NET-Core-and-PS.md)
