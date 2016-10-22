# Диагностика

Возникли проблемы при настройке компьютера или запуске контейнера? Для PowerShell был создан скрипт для проверки наличия распространенных проблем. Попробуйте использовать его и поделитесь своими результатами.

```PowerShell
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/windows-server-container-tools/Debug-ContainerHost/Debug-ContainerHost.ps1 | Invoke-Expression
```
Список всех запускаемых тестов вместе с распространенными решениями находится в [файле сведений](https://github.com/Microsoft/Virtualization-Documentation/blob/master/windows-server-container-tools/Debug-ContainerHost/README.md) скрипта.

Если это не поможет найти источник проблемы, опубликуйте результаты из скрипта на [форуме по контейнерам](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). Это наилучший способ получить помощь от сообщества, в том числе от участников программы предварительной оценки Windows и разработчиков.


## Нахождение журналов
Существует ряд служб, которые используются для управления контейнерами Windows. В следующих разделах показано, где найти журналы каждой из служб.

### Подсистема Docker
Подсистема Docker записывает сообщения в журнал событий приложений Windows, а не в файл журнала. Эти журналы можно легко прочитать, отсортировать и отфильтровать с помощью Windows PowerShell.

Например, следующая команда выведет записи журнала подсистемы Docker за последние 5 минут, начиная с самой ранней.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Эти записи легко перенаправить в CSV-файл, чтобы открыть их в другой программе или редакторе электронных таблиц.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### Включение ведения журналов отладки
Вы можете также включить ведение журналов на уровне отладки в подсистеме Docker. Это может помочь в устранении проблем, если в обычных журналах недостаточно подробностей.

Сначала откройте командную строку с повышенными привилегиями, а затем запустите `sc.exe qc docker`, чтобы отобразилась текущая командная строка службы Docker.
Пример.
```none
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Используйте текущую команду, `BINARY_PATH_NAME`, и измените ее:
- Добавьте в конец "-D".
- Добавьте экранирующий символ "\" для каждой кавычки (").
- Заключите всю команду в кавычки (").

После запустите `sc.exe config docker binpath= `, добавив новую строку. Пример. 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Теперь перезапустите службу Docker.
```none
sc.exe stop docker
sc.exe start docker
```

В журнале событий приложений появится гораздо больше записей, поэтому рекомендуется удалить параметр `-D` после завершения устранения проблем. Выполните действия, указанные выше, без `-D` и перезапустите службу, чтобы отключить ведение журналов отладки.


### Служба контейнера узлов
Подсистема Docker зависит от службы контейнера узлов конкретной версии Windows. Она содержит отдельные журналы: 
- Microsoft-Windows-Hyper-V-Compute-Admin.
- Microsoft-Windows-Hyper-V-Compute-Operational.

Они отображаются в компоненте "Просмотр событий". Кроме того, их можно запросить через PowerShell.

Пример.
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```



<!--HONumber=Oct16_HO3-->


