---
title: &1445845315 Ресурсы сообщества
description: Ресурсы сообщества
keywords: windows 10, hyper-v, container, docker
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1540477110 virtualization
ms.service: virtualization
ms.assetid: 731ed95a-ce13-4c6e-a450-49563bdc498c
---

# Участие в разработке документации

> <g id="1" ctype="x-strong">Примечание.</g> Для участия в разработке документов необходимо иметь учетную запись <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">GitHub</g><g id="3CapsExtId3" ctype="x-title"></g></g>.

## Редактирование существующего документа

1. Найдите документ, который следует отредактировать.

2. Выберите <g id="2" ctype="x-strong">Contribute to this topic (Изменить этот раздел)</g>  
  <g id="1" ctype="x-linkText"></g>

  Вы будете автоматически перенаправлены в GitHub в файл разметки, связанный с этим файлом.

  Убедитесь, что вы вошли в GitHub. В противном случае выполните вход или создайте учетную запись GitHub.

  <g id="1" ctype="x-linkText"></g>

3. Выберите значок правки для редактирования в редакторе браузера.

  <g id="1" ctype="x-linkText"></g>

4. Внесите соответствующие изменения.

  Возможные действия:
  1. Изменение файла
  2. Просмотр изменений
  3. Переименование файла (крайне маловероятно, что это потребуется)

  <g id="1" ctype="x-linkText"></g>

5. Предложить изменения в виде запроса на включение

  <g id="1" ctype="x-linkText"></g>

6. Проверить изменения

  <g id="1" ctype="x-strong">Что проверяется в запросе на включение</g>
  * Изменение верно — достоверно представляет технологию
  * Правильная орфография и грамматика
  * Логическое расположение в документации

  <g id="1" ctype="x-linkText"></g>

7. Создание <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">запроса на включение внесенных изменений</g><g id="2CapsExtId3" ctype="x-title"></g></g>

## Запрос на включение

Большинство изменений будут внесены по запросу на включение. Запрос на включение — способ проверки набора изменений несколькими рецензентами, а также изменения и комментирования текущего содержимого.


## Разветвление репозитория и локальное редактирование

Для длительной работы с документами клонируйте репозиторий в локальную среду и пользуйтесь им на своем компьютере.

Следующее руководство описывает, как эмулировать мою настройку (автор — Сара Кули (Sarah Cooley)). Существует множество альтернативных настроек, которые работают одинаково хорошо.

> <g id="1" ctype="x-strong">Примечание.</g> Все эти средства для работы с документами функционируют одинаково хорошо на Linux или OSX. Если вам нужны другие руководства, запросите их.

Процедура делится на три этапа:
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Настройка Git</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Установка Git
  * Начальная настройка
  * Разветвление репозитория документации
  * Клонирование копии на локальный компьютер
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Начальное управление учетными данными</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Сведения о задании учетных данных и вспомогательном средстве для них
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Настройка среды документов</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Установка VSCode
  * Пошаговое рассмотрение некоторых удобных функций VSCode для Git
  * Первая фиксация

### Настройка Git

1. Установите Git (в Windows) <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">отсюда</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

  В установке нужно изменить всего одно значение:

  <g id="1" ctype="x-strong">Adjusting your PATH environment</g> (Настройка среды PATH)
  "Use Git from the Windows Command Prompt" (Использовать Git из командной строки Windows)

  <g id="1" ctype="x-linkText"></g>

  Это позволяет использовать команды Git в консоли PowerShell или в любой консоли Windows.

2. Настройте свое удостоверение Git.

  Откройте окно PowerShell и выполните следующую команду:

  ``` PowerShell
  git config --global user.name "User Name"
  git config --global user.email username@microsoft.com
  ```

  Git использует эти значения для обозначения фиксаций.

> Если возникает приведенная ниже ошибка, возможно, Git установлен неправильно или необходимо перезапустить PowerShell.
>    ``` PowerShell
>    git: имя "git" не распознано как имя командлета, функции, файла сценария или выполняемой программы. Проверьте правильность написания имени, а также наличие и правильность пути, после чего повторите попытку.
>    ```

3. Настройте свою среду Git.

   Настройте вспомогательное средство для учетных данных, чтобы ввести имя пользователя и пароль только один раз (по крайней мере на этом компьютере).
   Я использую это базовое <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">вспомогательное средство для учетных данных Windows</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

   После установки выполните следующую команду, чтобы включить вспомогательное средство для учетных данных и задать поведение принудительной отправки:
   ```
   git config --global credential.helper manager
   git config --global push.default simple
   ```

   При первом запуске необходимо выполнить проверку подлинности для GitHub — вам будет предложено ввести свое имя пользователя и код двухфакторной проверки подлинности, если он используется.
   Пример.
   ```
   C:\Users\plang\Source\Repos\Virtualization-Documentation [master]> git pull
   Please enter your GitHub credentials for https://github.com/
   username: plang@microsoft.com
   password:
   authcode (app): 562689
   ```
   При этом автоматически создается <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">личный маркер доступа</g><g id="2CapsExtId3" ctype="x-title"></g></g> с соответствующими разрешениями на GitHub,
   который затем безопасно сохраняется на локальном компьютере. Больше этот запрос выводиться не должен.

4. Выполните разветвление репозитория.

5. Клонируйте репозиторий.

  Клон Git создает локальную копию репозитория с подходящими обработчиками для синхронизации с другими клонами того же репозитория.

  По умолчанию клон создает в текущем каталоге папку, имя которой совпадает с именем репозитория. Я храню все репозитории Git в своем каталоге пользователя. Дополнительные сведения о клоне Git см. <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">здесь</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

  ``` PowerShell
  cd ~
  git clone https://github.com/Microsoft/Virtualization-Documentation.git
  ```

  Если все прошло успешно, теперь у вас есть папка <g id="2" ctype="x-code">Virtualization-Documentation</g>.

  ``` PowerShell
  cd Virtualization-Documentation
  ```

5. [Необязательно] Настройте Posh-Git.

  Posh-Git — это модуль PowerShell, который создан членами сообщества и делает работу с Git в PowerShell немного удобнее. Он добавляет заполнение нажатием клавиши в Git для PowerShell, а также отображение сведений о ветвлении и состоянии файла в командной строке. Дополнительные сведения о нем см. <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">здесь</g><g id="2CapsExtId3" ctype="x-title"></g></g>. Posh-Git можно установить, выполнив приведенную ниже команду в консоли администратора PowerShell.

  ``` PowerShell
  Install-Module -Name posh-git
  ```

  Чтобы автоматически запускать Posh-Git при каждом открытии PowerShell, добавьте следующий код в свой профиль PowerShell (например, <g id="2" ctype="x-code">%UserProfile%\My Documents\WindowsPowerShell\profile.ps1 </g>).

  ``` PowerShell
  Import-Module posh-git

  function global:prompt {
    $realLASTEXITCODE = $LASTEXITCODE

    Write-Host($pwd.ProviderPath) -nonewline

    Write-VcsStatus

    $global:LASTEXITCODE = $realLASTEXITCODE
    return "> "
  }
  ```

### Проверка и задание учетных данных

  Чтобы проверить, правильно ли настроен репозиторий, попробуйте извлечь новое содержимое.

  ``` PowerShell
  git pull
  ```


### Настройка среды редактирования разметки

1. Скачайте VSCode.

6. Выполните тестовую фиксацию. Если учетные данные заданы правильно, все должно работать как нужно.









<!--HONumber=May16_HO1-->


