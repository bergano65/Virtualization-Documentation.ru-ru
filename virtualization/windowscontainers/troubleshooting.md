---
title: Устранение неполадок при работе с контейнерами Windows
description: Рекомендации по устранению неполадок, автоматические сценарии и запись в журнал сведений о контейнерах Windows и Docker
keywords: docker, контейнеры, устранение неполадок, журналы
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 652b1a8e0ab12ac67dd2754051e36c523e3de509
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882947"
---
# <a name="troubleshooting"></a>Диагностика

Возникли проблемы при настройке компьютера или запуске контейнера? Для PowerShell был создан скрипт для проверки наличия распространенных проблем. Попробуйте использовать его и поделитесь своими результатами.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
Список всех запускаемых тестов вместе с распространенными решениями находится в [файле сведений](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) скрипта.

Если это не поможет найти источник проблемы, опубликуйте результаты из скрипта на [форуме по контейнерам](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers). Это наилучший способ получить помощь от сообщества, в том числе от участников программы предварительной оценки Windows и разработчиков.


## <a name="finding-logs"></a>Нахождение журналов
Существует несколько служб, используемых для управления контейнерами Windows. В следующих разделах показано, где найти журналы каждой из служб.

# <a name="docker-engine"></a>Подсистема Docker
Подсистема Docker записывает сообщения в журнал событий приложений Windows, а не в файл журнала. Эти журналы можно легко прочитать, отсортировать и отфильтровать с помощью Windows PowerShell.

Например, следующая команда выведет записи журнала подсистемы Docker за последние 5минут, начиная с самой ранней.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Эти записи легко перенаправить в CSV-файл, чтобы открыть их в другой программе или редакторе электронных таблиц.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

## <a name="enabling-debug-logging"></a>Включение ведения журналов отладки
Вы можете также включить ведение журналов на уровне отладки в подсистеме Docker. Это может помочь в устранении проблем, если в обычных журналах недостаточно подробностей.

Сначала откройте командную строку с повышенными привилегиями, а затем запустите `sc.exe qc docker`, чтобы отобразилась текущая командная строка службы Docker.
Пример.
```
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

После запустите `sc.exe config docker binpath=`, добавив новую строку. Пример. 
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Теперь перезапустите службу Docker.
```
sc.exe stop docker
sc.exe start docker
```

В журнале событий приложений появится гораздо больше записей, поэтому рекомендуется удалить параметр `-D` после завершения устранения проблем. Выполните действия, указанные выше, без `-D` и перезапустите службу, чтобы отключить ведение журналов отладки.

Альтернативным решением является запуск управляющей программы Docker в режиме отладки из командной строки PowerShell с правами администратора и записью данных вывода непосредственно в файл.
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

## <a name="obtaining-stack-dump"></a>Получение дампа стека.

Как правило, это полезно, только если вы явным образом запросили разработчики службы поддержки Майкрософт или закрепления. Она может использоваться для диагностики ситуации, когда Dock зависла. 

Скачайте файл [docker-signal.exe](https://github.com/jhowardmsft/docker-signal).

Использование:
```PowerShell
Get-Process dockerd
# Note the process ID in the `Id` column
docker-signal -pid=<id>
```

Выходной файл будет находиться в закрепление корневого каталога данных, в котором он находится. Каталогом по умолчанию является `C:\ProgramData\Docker`. Фактическое расположение можно проверить, выполнив команду `docker info -f "{{.DockerRootDir}}"`.

Файл будет `goroutine-stacks-<timestamp>.log`.

Обратите `goroutine-stacks*.log` внимание, что не содержит персональных данных.


# <a name="host-compute-service"></a>Служба вычисления узлов
Подсистема Docker зависит от службы контейнера узлов конкретной версии Windows. Она содержит отдельные журналы: 
- Microsoft-Windows-Hyper-V-Compute-Admin.
- Microsoft-Windows-Hyper-V-Compute-Operational.

Они отображаются в компоненте "Просмотр событий". Кроме того, их можно запросить через PowerShell.

Пример.
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

## <a name="capturing-hcs-analyticdebug-logs"></a>Запись журналов аналитики/отладки службы HCS

Включите запись журналов аналитики/отладки для вычисления Hyper-V с их сохранением в файле `hcslog.evtx`.

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

## <a name="capturing-hcs-verbose-tracing"></a>Запись подробной трассировки службы HCS

Как правило, эти данные запрашивает только служба поддержки Майкрософт. 

Скачайте файл [HcsTraceProfile.wprp](https://gist.github.com/jhowardmsft/71b37956df0b4248087c3849b97d8a71)

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

Предоставьте файл `HcsTrace.etl` специалисту службы поддержки.
