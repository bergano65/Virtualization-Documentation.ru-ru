---
title: Создайте собственные службы интеграции
description: Службы интеграции Windows10.
keywords: windows 10, hyper-v, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: 966ca3ff267e03e8c380391281c8dde723e4b1dd
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575334"
---
# <a name="make-your-own-integration-services"></a>Создайте собственные службы интеграции

Начиная с выхода юбилейного обновления Windows 10 любой пользователь может создавать приложения, которые взаимодействуют с узлом Hyper-V и его виртуальными машинами с помощью сокетов Hyper-V — сокетов Windows с новым семейством адресов и специализированной конечной точкой для работы с виртуальными машинами.  Вся связь через сонеты Hyper-V происходит без использования сети, а все данные остаются в одной и той же физической памяти.   Приложения, использующие сонеты Hyper-V, похожи на службы интеграции Hyper-V.

В этом документе показано, как создать простую программу на основе сокетов Hyper-V.

**Поддерживаемые операционные системы узла**
* Windows 10 и более поздние версии
* Windows Server 2016 и более поздние версии

**Поддерживаемые гостевые операционные системы**
* Windows 10 и более поздние версии
* Windows Server 2016 и более поздние версии
* Гостевые ОС Linux со службами интеграции Linux Integration Services (см. статью [Поддерживаемые виртуальные машины Linux и FreeBSD для Hyper-V в Windows](https://technet.microsoft.com/library/dn531030.aspx))
> **Примечание.** Ядро поддерживаемой гостевой ОС Linux должно поддерживать следующее:
> ```bash
> CONFIG_VSOCKET=y
> CONFIG_HYPERV_VSOCKETS=y
> ```

**Возможности и ограничения**
* Поддержка режима ядра или действий в режиме пользователя
* Только поток данных
* Нет блочной памяти (не подходит для резервного копирования и видео)

--------------

## <a name="getting-started"></a>Начало работы

Требования:
* компилятор C/C++  (если у вас его нет, перейдите в [сообщество Visual Studio](https://aka.ms/vs));
* [пакет SDK для Windows10](https://developer.microsoft.com/windows/downloads/windows-10-sdk) — устанавливается вместе с Visual Studio 2015 с обновлением 3 и более поздними версиями;
* компьютер под управлением одной из операционной систем сервера виртуальных машин, указанных выше, по крайней мере с одной виртуальной машиной (необходимо для тестирования приложения).

> **Примечание.** API для сокетов Hyper-V стал общедоступным в Windows 10 немного позднее.  Приложения, использующие HVSocket, будут работать на любом хосте и гостевом узле с Widnows 10, но могут создаваться только с помощью пакета SDK для Windows сборки более поздней, чем 14290.

## <a name="register-a-new-application"></a>Регистрация нового приложения
Чтобы использовать сокеты Hyper-V, необходимо зарегистрировать приложение в реестре узла Hyper-V.

Зарегистрировав службу в реестре, вы получите:
*  Управление WMI для включения, отключения и отображения списка доступных служб
*  Разрешение на прямую связь с виртуальными машинами

Следующая команда PowerShell регистрирует новое приложение с именем HV Socket Demo.  Ее необходимо выполнять от имени администратора.  Инструкции по регистрации вручную приведены ниже.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**Расположение и данные реестра**
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
В этом разделе реестра есть несколько кодов GUID.  Это и есть службы, поставляемые с Windows.

Сведения в реестре для каждой службы:
* `Service GUID`
    * `ElementName (REG_SZ)` — понятное имя службы

Чтобы зарегистрировать свою службу, создайте раздел реестра, используя свой код GUID и понятное имя.

Понятное имя будет связано с новым приложением.  Оно будет отображаться в счетчиках производительности и других местах, где невозможно использовать код GUID.

Запись реестра будет выглядеть следующим образом:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

> **Примечание.** Идентификатор GUID службы для гостевой ОС Linux использует протокол VSOCK, который отправляет запросы через `svm_cid` и `svm_port`, а не через идентификатор GUID. Для компенсации этого несоответствия в Windows хорошо известный идентификатор GUID используется в качестве шаблона службы на узле, который преобразуется в порт в гостевой ОС. Для настройки идентификатора GUID службы измените первое значение "00000000" на необходимый номер порта. Пример: "00000ac9" обозначает порт 2761.
> ```C++
> // Hyper-V Socket Linux guest VSOCK template GUID
> struct __declspec(uuid("00000000-facb-11e6-bd58-64006a7986d3")) VSockTemplate{};
>
>  /*
>   * GUID example = __uuidof(VSockTemplate);
>   * example.Data1 = 2761; // 0x00000AC9
>   */
> ```
>

> **Совет.** Чтобы создать GUID в PowerShell и скопировать его в буфер обмена, используйте следующую команду:
>``` PowerShell
>(New-Guid).Guid | clip.exe
>```

## <a name="create-a-hyper-v-socket"></a>Создание сокета Hyper-V

В большинстве случаев, чтобы определить сокет, требуется семейство адресов, тип соединения и протокол.

Вот простое [определение сокета](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
)

``` C
// Windows
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);

// Linux guest
int socket(int domain, int type, int protocol);
```

Для сокета Hyper-V:
* Семейство адресов— `AF_HYPERV` (Windows) или `AF_VSOCK` (гостевая ОС Linux)
* Тип— `SOCK_STREAM`
* Протокол— `HV_PROTOCOL_RAW` (Windows) или `0` (гостевая ОС Linux)


Пример объявления или создания экземпляра:
``` C
// Windows
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);

// Linux guest
int sock = socket(AF_VSOCK, SOCK_STREAM, 0);
```

## <a name="bind-to-a-hyper-v-socket"></a>Привязка к сокету Hyper-V

Привязка связывает сокет со сведениями о подключении.

Определение этой функции скопировано ниже для удобства. Подробнее о привязке см. [здесь](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx).

``` C
// Windows
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);

// Linux guest
int bind(int sockfd, const struct sockaddr *addr,
         socklen_t addrlen);
```

В отличие от адреса сокета (sockaddr) для стандартного семейства адресов протокола IP (`AF_INET`), который состоит из IP-адреса хост-компьютера и номера порта на этом узле, адрес сокета для `AF_HYPERV` использует для подключения идентификатор виртуальной машины и описанный выше идентификатор приложения. При привязке через гостевую ОС Linux `AF_VSOCK` использует `svm_cid` и `svm_port`.

Так как сокеты Hyper-V не зависят от сетевого стека, TCP/IP, DNS ит.д., для конечной точки сокета требуется формат, не использующий протокол IP и имя узла, но однозначно описывающий подключение.

Вот определение адреса сокета Hyper-V:

``` C
// Windows
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};

// Linux guest
// See include/uapi/linux/vm_sockets.h for more information.
struct sockaddr_vm {
    __kernel_sa_family_t svm_family;
    unsigned short svm_reserved1;
    unsigned int svm_port;
    unsigned int svm_cid;
    unsigned char svm_zero[sizeof(struct sockaddr) -
                   sizeof(sa_family_t) -
                   sizeof(unsigned short) -
                   sizeof(unsigned int) - sizeof(unsigned int)];
};
```

Вместо протокола IP или имени узла конечные точки AF_HYPERV используют два кода GUID:
* ИД ВМ— это уникальный идентификатор виртуальной машины.  Его можно узнать с помощью следующего фрагмента кода PowerShell.
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* Идентификатор службы— код GUID, [описанный выше](#RegisterANewApplication), под которым приложение зарегистрировано в реестре узла Hyper-V.

Кроме того, для идентификатора виртуальной машины доступен ряд подстановочных знаков.

### <a name="vmid-wildcards"></a>Подстановочные знаки для идентификатора виртуальной машины

| Название | Код GUID | Описание |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | Прослушиватели необходимо привязать к этому идентификатору виртуальной машины, чтобы принимать подключения от всех разделов. |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | Прослушиватели необходимо привязать к этому идентификатору виртуальной машины, чтобы принимать подключения от всех разделов. |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | Адрес из подстановочных знаков для дочерних элементов. Прослушиватели необходимо привязать к этому идентификатору виртуальной машины, чтобы принимать подключения от дочерних элементов. |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | Петлевой адрес. При использовании этого адреса выполняется подключение к разделу соединителя. |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | Адрес родительского элемента. При использовании этого адреса выполняется подключение к родительскому разделу соединителя*. |


\* `HV_GUID_PARENT` Родительский элемент виртуальной машины— это ее узел.  Родительский элемент контейнера— это узел контейнера.
При подключении из контейнера, запущенного в виртуальной машине, будет выполнено подключение к виртуальной машине, в которой размещен контейнер.
Прослушивание для этого VmId поддерживает подключение от (в контейнерах): узла контейнера.
(В виртуальной машине: узел контейнера или без контейнера): узла виртуальной машины.
(Не в виртуальной машине: узел контейнера или без контейнера): не поддерживается.

## <a name="supported-socket-commands"></a>Поддерживаемые команды сокета

Socket() Bind() Connect() Send() Listen() Accept()

## <a name="useful-links"></a>Полезные ссылки
[Полный API WinSock](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

[Справочник по службам интеграции Hyper-V](../reference/integration-services.md)