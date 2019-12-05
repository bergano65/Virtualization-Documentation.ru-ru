# <a name="microsoft-software-supplemental-license-for-windows-container-base-image"></a>ОСНОВНОЙ ОБРАЗ ВСПОМОГАТЕЛЬНОЙ ЛИЦЕНЗИИ НА ПРОГРАММНОЕ ОБЕСПЕЧЕНИЕ МАЙКРОСОФТ ДЛЯ КОНТЕЙНЕРА WINDOWS

Эта дополнительная лицензия предназначена для базового образа контейнера Windows ("изображение контейнера"). Если вы отвечаете на условия этой дополнительной лицензии, вы можете использовать изображение контейнера, как описано ниже.

## <a name="container-os-image"></a>ОБРАЗ ОС КОНТЕЙНЕРА
Изображение контейнера может использоваться только с действительной лицензированной копией:

Windows Server Standard или Windows Server Datacenter по (общее название программного обеспечения сервера) или программное обеспечение Microsoft Windows (версии 10) ("программное обеспечение клиента") или Windows 10 IoT корпоративный и Windows 10 IoT ядро (собрано в центре Интернета вещей Программное обеспечение узла).
Программное обеспечение для хост-сервера, программное обеспечение клиента и программное обеспечение для узла IoT называются "программным обеспечением", а лицензия на программное обеспечение Host — "лицензия на узел".

Вы не можете использовать изображение контейнера, если у вас нет соответствующей версии и выпуска лицензии на узел. Могут применяться определенные ограничения и дополнительные условия, описанные в настоящем документе. Если условия лицензионного соглашения конфликтуют с лицензией Host, эта дополнительная лицензия должна управляться по отношению к образу контейнера. ПРИНИМАЯ ЭТУ ДОПОЛНИТЕЛЬНУЮ ЛИЦЕНЗИЮ ИЛИ ИСПОЛЬЗУЯ ИЗОБРАЖЕНИЕ КОНТЕЙНЕРА, ВЫ СОГЛАШАЕТЕСЬ С УСЛОВИЯМИ. ЕСЛИ ВЫ НЕ ПРИНЯЛИ И НЕ СОГЛАСНЫ С ЭТИМИ УСЛОВИЯМИ, ВЫ НЕ МОЖЕТЕ ИСПОЛЬЗОВАТЬ ИЗОБРАЖЕНИЕ КОНТЕЙНЕРА.

## <a name="definitions"></a>КРАТКИ
**Контейнер Windows Server** (без изоляции Hyper-V) — это компонент программного обеспечения Microsoft Windows Server.

**Контейнер Windows Server с изоляцией Hyper-V.** Раздел 2 (k) условий лицензии на Microsoft Windows Server настоящим удаляется полностью и заменен на пересмотренные условия, как показано в разделе "Обновлено".

Обновлено: контейнер Windows Server с изоляцией Hyper-V (прежнее название — контейнер Hyper-V) — это технология контейнеров в Windows Server, использующая виртуальную среду операционной системы для размещения одного или нескольких контейнеров Windows Server (ов). Каждый экземпляр изоляции Hyper-V, используемый для размещения одного или нескольких контейнеров Windows Server, считается одной из виртуальных сред операционной системы.

## <a name="license-terms"></a>УСЛОВИЯ ЛИЦЕНЗИОННОГО СОГЛАШЕНИЯ
**Лицензия на узел.** Условия лицензионного соглашения применяются к использованию изображения контейнера и любых контейнеров Windows (s), созданных с помощью изображения контейнера, которое отличается от виртуальной машины и отделена от нее.

**Использование прав.** Изображение контейнера можно использовать для создания изолированной виртуальной среды операционной системы Windows, включающей по крайней мере одно приложение, которое добавляет основную и важную функциональность. Вы можете использовать изображение контейнера только для создания, построения и запуска контейнеров Windows (ов) в программном обеспечении Host. Обновления программного обеспечения узла могут не обновить изображение контейнера, чтобы можно было повторно создать контейнеры Windows, основанные на обновленном графическом контейнере.

**Контроль.** Вы не можете удалить этот файл дополнительного лицензионного документа из изображения контейнера. Вы не можете разрешить удаленный доступ к приложениям, которые вы пропустили в вашем контейнере, чтобы избежать появления соответствующих лицензий. Вы не можете реконструировать, декомпилировать или разбирать изображение контейнера или попытаться сделать это, за исключением случаев, предусмотренных условиями лицензирования третьих лиц, которые управляют использованием определенных компонентов с открытым кодом, которые могут быть включены в программное обеспечение. Может взиматься дополнительная лицензия на использование лицензии на узел.

## <a name="additional-terms"></a>ДОПОЛНИТЕЛЬНЫЕ УСЛОВИЯ
**Программное обеспечение на клиентском компьютере.** При запуске изображения контейнера на клиентском хост-сервере вы можете запускать любое количество графических элементов, созданных как контейнеры Windows, только для целей тестирования и разработки. Вы не можете использовать эти контейнеры Windows в производственной среде на клиентском компьютере.

**Программное обеспечение для размещения вещей.** При работе с изображением контейнера по программному обеспечению для размещения вещей вы можете запускать любое количество графических элементов, созданных как контейнеры Windows, только для целей тестирования и разработки. Вы можете использовать изображение контейнера в производственной среде только в том случае, если вы согласились с коммерческими условиями использования Windows 10 Core Images среды выполнения или лицензией на корпоративное приложение Windows 10 IoT ("коммерческое соглашение Windows IoT"). Дополнительные условия и ограничения для коммерческих соглашений о Windows IoT применяются к использованию изображения контейнера в рабочей среде.

**Стороннее программное обеспечение.** Изображение контейнера может включать сторонние приложения, лицензированные вам в рамках этой дополнительной лицензии или собственные условия. Условия лицензионного соглашения, уведомления и подтверждения, если таковые имеются, для сторонних приложений могут быть доступны в http://aka.ms/thirdpartynotices сети или в сопроводительном файле уведомлений. Даже если подобные приложения управляются другими соглашениями, отказ от ответственности, ограничения и исключения ущерба в лицензии на узел также распространяются на разрешенные применимым законодательством.

**Открыть компоненты исходного кода.** На изображении контейнера могут содержаться программные продукты третьих лиц, лицензированные в рамках лицензий Open Source с обязательствами по обеспечению доступности исходного кода. Копии этих лицензий содержатся в файле Сирдпартинотицес или другом сопровождающем файле уведомлений. Вы можете получить полный соответствующий исходный код от корпорации Майкрософт, если в соответствии с соответствующей лицензией на открытых источниках, отправив заказ на деньги или поиск в $5,00: группа соответствия исходному коду, Microsoft Corporation, 1 Microsoft Way, Redmond, WA 98052, США. Укажите имя "Вспомогательная лицензия на программное обеспечение Microsoft для контейнера Windows", а также имя и номер версии исходного компонента в строке "записка" для платежа. Кроме того, вы можете найти копию исходного файла http://aka.ms/getsource.