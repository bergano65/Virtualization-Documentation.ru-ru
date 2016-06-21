---
title: Создайте собственные службы интеграции
description: Службы интеграции Windows 10.
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &81555379 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
---

# Создайте собственные службы интеграции

Начиная с ОС Windows 10, любой пользователь может создать службу, практически аналогичную встроенным службам интеграции Hyper-V, с помощью нового канала связи на основе сокетов между узлом Hyper-V и запущенными на нем виртуальными машинами. С помощью этих сокетов Hyper-V службы могут работать независимо от сетевого стека и все данные остаются в той же физической памяти.

В этой статье приведены пошаговые инструкции по созданию простого приложения, созданного на сокетах Hyper-V, и началу работы.

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">PowerShell Direct</g><g id="1CapsExtId3" ctype="x-title"></g></g> — это пример приложения (в данном случае службы, поставляемой с Windows), использующего для связи сокеты Hyper-V.

<g id="1" ctype="x-strong">Поддерживаемые операционные системы узла</g>
* Сборка 14290 системы Windows 10 и более поздние сборки
* Windows Server Technical Preview 4 и более поздние версии
* Будущие выпуски (Server 2016 +)

<g id="1" ctype="x-strong">Поддерживаемые гостевые операционные системы</g>
* Windows 10
* Windows Server Technical Preview 4 и более поздние версии
* Будущие выпуски (Server 2016 +)
* Гости Linux со службами интеграции Linux (см. раздел <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Поддерживаемые виртуальные машины Linux и FreeBSD для Hyper-V в Windows</g><g id="2CapsExtId3" ctype="x-title"></g></g>)

<g id="1" ctype="x-strong">Возможности и ограничения</g>
* Поддержка режима ядра или действий в режиме пользователя
* Только поток данных
* Нет блочной памяти (не подходит для резервного копирования и видео)

--------------


## Начало работы

В настоящий момент сокеты Hyper-V доступны в виде машинного кода (C/C++).

Чтобы написать простое приложение, вам понадобится следующее:
* Компилятор C. Если у вас его нет, перейдите в <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">сообщество Visual Studio</g><g id="2CapsExtId3" ctype="x-title"></g></g>.
* Компьютер, на котором работают Hyper-V и виртуальная машина.
  * В качестве гостевой операционной системы (виртуальная машина) и ОС узла должна использоваться Windows 10, Windows Server Technical Preview 3 или более поздней версии.
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Пакет SDK для Windows 10</g><g id="1CapsExtId3" ctype="x-title"></g></g> установлен на узле Hyper-V

<g id="1" ctype="x-strong">Сведения о Windows SDK</g>

Ссылки на пакет SDK для Windows:
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Пакет SDK для Windows 10 — предварительная версия для инсайдеров</g><g id="1CapsExtId3" ctype="x-title"></g></g>
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Пакет SDK для Windows 10</g><g id="1CapsExtId3" ctype="x-title"></g></g>

Интерфейс API для сокетов Hyper-V стал доступен в Windows 10, начиная со сборки 14290, — скачиваемые файлы фокус-тестирования соответствуют последней сборке фокус-тестирования для инсайдеров.  
При странностях в поведении сообщите нам через <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">форум TechNet</g><g id="2CapsExtId3" ctype="x-title">Форумы TechNet</g></g>. В сообщение включите следующее.
* Описание непредвиденного поведения.
* Номера ОС и сборки для узла, виртуальной машины и пакета SDK.

  Номер сборки пакета SDK отображается в заголовке установщика для пакета SDK:  
  <g id="1" ctype="x-linkText"></g>


## Регистрация нового приложения

Чтобы использовать сокеты Hyper-V, необходимо зарегистрировать приложение в реестре узла Hyper-V.

Зарегистрировав службу в реестре, вы получите:
*  Управление WMI для включения, отключения и отображения списка доступных служб
*  Разрешение на прямую связь с виртуальными машинами

Следующая команда PowerShell регистрирует новое приложение с именем HV Socket Demo. Ее необходимо выполнять от имени администратора. Инструкции по регистрации вручную приведены ниже.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID and add it to the services list then add the name as a value

$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```

<g id="1" ctype="x-em">* Расположение и сведения реестра *</g>

``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
В этом расположении реестра есть несколько кодов GUID. Это и есть службы, поставляемые с Windows.

Сведения в реестре для каждой службы:
* <g id="1" ctype="x-code">Service GUID</g>
    * <g id="1" ctype="x-code">ElementName (REG_SZ)</g> — понятное имя службы

Чтобы зарегистрировать свою службу, создайте раздел реестра, используя свой код GUID и понятное имя.

Понятное имя будет связано с новым приложением. Оно будет отображаться в счетчиках производительности и других местах, где невозможно использовать код GUID.

Запись реестра будет выглядеть следующим образом:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

> <g id="1" ctype="x-em">* Совет. *</g> Чтобы создать код GUID в PowerShell и скопировать его в буфер обмена, используйте следующую команду:
``` PowerShell
(New-Guid).Guid | clip.exe
```

## Создание сокета Hyper-V

В большинстве случаев, чтобы определить сокет, требуется семейство адресов, тип соединения и протокол.

Вот простое [определение сокета](
https://msdn.microsoft.com/ru-ru/library/windows/desktop/ms740506(v=vs.85).aspx
)

``` C
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);
```

Для сокета Hyper-V:
* Семейство адресов: <g id="2" ctype="x-code">AF_HYPERV</g>
* Тип — <g id="2" ctype="x-code">SOCK_STREAM</g>
* Протокол: <g id="2" ctype="x-code">HV_PROTOCOL_RAW</g>


Пример объявления или создания экземпляра:
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## Привязка к сокету Hyper-V

Привязка связывает сокет со сведениями о подключении.

Определение данной функции скопировано ниже для удобства. Подробнее о привязке см. <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">здесь</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

``` C
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);
```

В отличие от адреса сокета (sockaddr) для стандартного семейства адресов протокола IP (<g id="2" ctype="x-code">AF_INET</g>), который состоит из IP-адреса хост-компьютера и номера порта на этом узле, адрес сокета для <g id="4" ctype="x-code">AF_HYPERV</g> использует для подключения идентификатор виртуальной машины и описанный выше идентификатор приложения.

Так как сокеты Hyper-V не зависят от сетевого стека, TCP/IP, DNS и т. д., для конечной точки сокета требуется формат, не использующий протокол IP и имя узла, но однозначно описывающий подключение.

Вот определение адреса сокета Hyper-V:

``` C
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};
```

Вместо протокола IP или имени узла конечные точки AF_HYPERV используют два кода GUID:
* ИД ВМ — это уникальный идентификатор виртуальной машины. Его можно узнать с помощью следующего фрагмента кода PowerShell.
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* Идентификатор службы — код GUID, <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">описанный выше</g><g id="2CapsExtId3" ctype="x-title"></g></g>, под которым приложение зарегистрировано в реестре узла Hyper-V.

Кроме того, для идентификатора виртуальной машины доступен ряд подстановочных знаков.

### Подстановочные знаки для идентификатора виртуальной машины

| Название| Код GUID| Описание|
|:-----|:-----|:-----|
| HV_GUID_ZERO| 00000000-0000-0000-0000-000000000000| Прослушиватели необходимо привязать к этому идентификатору виртуальной машины, чтобы принимать подключения от всех разделов.|
| HV_GUID_WILDCARD| 00000000-0000-0000-0000-000000000000| Прослушиватели необходимо привязать к этому идентификатору виртуальной машины, чтобы принимать подключения от всех разделов.|
| HV_GUID_BROADCAST| FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF| |
| HV_GUID_CHILDREN| 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd| Адрес из подстановочных знаков для дочерних элементов.Прослушиватели необходимо привязать к этому идентификатору виртуальной машины, чтобы принимать подключения от дочерних элементов.|
| HV_GUID_LOOPBACK| e0e16197-dd56-4a10-9195-5ee7a155a838| Петлевой адрес.При использовании этого адреса выполняется подключение к разделу соединителя.|
| HV_GUID_PARENT| a42e7cda-d03f-480c-9cc2-a4de20abb878| Адрес родительского элемента.При использовании этого адреса выполняется подключение к родительскому разделу соединителя*.|


<g id="1" ctype="x-strong">* HV_GUID_PARENT</g>  
Родительский элемент виртуальной машины — это ее узел. Родительский элемент контейнера — это узел контейнера.  
При подключении из контейнера, запущенного в виртуальной машине, будет выполнено подключение к виртуальной машине, в которой размещен контейнер.  
Прослушивание на этом идентификаторе виртуальной машины принимает подключение от:  
(В контейнерах): узла контейнера.  
(В виртуальной машине: узел контейнера или без контейнера): узла виртуальной машины.  
(Не в виртуальной машине: узел контейнера или без контейнера): не поддерживается.

## Поддерживаемые команды сокета

Socket()
Bind()
Connect()
Send()
Listen()
Accept()

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Полный API WinSock</g><g id="1CapsExtId3" ctype="x-title"></g></g>

## Проблемы и их решение

Выбор нормального
отключения






<!--HONumber=May16_HO1-->


