# <a name="hyper-v-architecture"></a>Архитектура Hyper-V

Hyper-V— это технология виртуализации на базе низкоуровневой оболочки (или по-другому "гипервизора") для отдельных 64-разрядных версий Windows.  Гипервизор ключевым компонентом технологии виртуализации.  Это процессор-зависимая платформа виртуализации, позволяющая нескольким изолированным операционным системам использовать общую аппаратную платформу.

Hyper-V поддерживает изоляцию по разделам. Раздел— это логическая единица изоляции, поддерживаемая гипервизором, в котором работают операционные системы. У гипервизора Майкрософт должен быть по крайней мере один корневой (или по-другому "родительский") раздел под управлением Windows. Стек виртуализации запускается в родительском разделе и обладает прямым доступом к аппаратным устройствам. Затем корневой раздел порождает дочерние разделы, в которых и располагаются гостевые ОС. Корневой раздел создает дочерние с помощью API-интерфейса гипервызова.

У разделов нет доступа к физическому процессору и они не обрабатывают прерывания процессора. Вместо этого у них есть виртуальное представление процессора и они выполняются в виртуальном адресном пространстве, которое является частным для каждого гостевого раздела. Гипервизор управляет прерываниями процессора и перенаправляет их в соответствующий раздел. Кроме того, Hyper-V может аппаратным образом ускорять преобразование адресов между различными гостевыми виртуальными адресными пространствами с помощью модуля управления вводом/выводом памяти (IOMMU, Input Output Memory Management Unit), который работает независимо от аппаратного управления памятью, используемого процессором. Модуль IOMMU используется для изменения сопоставления адресов физической памяти с адресами, которые используют дочерние разделы.

У дочерних разделов также отсутствует прямой доступ к другим аппаратным ресурсам оборудования и есть виртуальное представление ресурсов в виде виртуальных устройств (VDev). Запросы к виртуальным устройствам перенаправляются через шину VMBus или через гипервизор к устройствам, находящимся в родительском разделе, который обрабатывает эти запросы. VMBus— это логический канал, по которому осуществляется взаимодействие между разделами. В родительских разделах находятся поставщики служб виртуализации (VSP, Virtualization Service Provider), которые подключаются к шине VMBus и обрабатывают запросы на доступ к устройствам от дочерних разделов. В дочерних разделах находятся клиенты служб виртуализации (VSC, Virtualization Service Client), которые перенаправляют запросы устройств через шину VMBus к поставщикам VSP родительского раздела. Этот процесс прозрачен для гостевой ОС.

Виртуальные устройства также могут использовать функцию виртуализации Windows Server под названием Enlightened I/O для подсистем хранения, сети, графической подсистемы и подсистемы ввода. Enlightened I/O— это специализированная, ориентированная на виртуализацию реализация протоколов связи высокого уровня (например SCSI), которые используют шину VMBus напрямую, в обход уровня эмуляции устройств. Это обеспечивает более эффективное взаимодействие, но требует наличия гостевой системы с поддержкой Enlightened I/O, которая знает о гипервизоре и VMBus Технология Hyper-V Еnlightened I/O и ядро с поддержкой определения гипервизора предоставляются при установке компонентов интеграции Hyper-V. Компоненты интеграции, к которым относятся драйверы клиента виртуальных серверов (VSC), также доступны для других клиентских операционных систем. Для Hyper-V необходим процессор с поддержкой аппаратной виртуализации, реализованной в таких технологиях, как Intel VT или AMD Virtualization (AMD-V).

На следующей схеме представлен общий обзор архитектуры среды Hyper-V.

![](./media/hv_architecture.png)

## <a name="glossary"></a>Словарь терминов
* **APIC, Advanced Programmable Interrupt Controller**— улучшенный программируемый контроллер прерываний. Устройство, позволяющее назначать уровни приоритетов соответствующим выводам по прерываниям.
* **Дочерний раздел**— раздел, в котором размещается гостевая операционная система. Доступ к физической памяти и устройствам дочерний раздел осуществляет через шину виртуальной машины (VMBus) или гипервизор.
* **Гипервызов**— интерфейс для связи с гипервизором. Интерфейс гипервызова обеспечивает доступ к оптимизации, предоставляемой гипервизором.
* **Гипервизор**— это программный уровень, расположенный между оборудованием и одной или несколькими операционными системами. Его основной задачей является предоставление изолированных сред выполнения, называемых разделами. Гипервизор управляет доступом к базовому оборудованию и определяет его приоритет.
* **IC, Integration component**— компонент интеграции. Компонент, обеспечивающий взаимодействие дочерних разделов с другими разделами и гипервизором.
* **Стек ввода-вывода**— стека ввода-вывода.
* **MSR, Memory Service Routine**— процедура службы памяти.
* **Корневой раздел**— иногда называемый родительским разделом.  Управляет функциями на уровне машины, например драйверами устройств, питанием и горячим подключением/отключением устройств. Корневой (или родительский) раздел является единственным разделом, который имеет прямой доступ к физической памяти и устройствам.
* **VID, Virtualization Infrastructure Driver**— драйвер инфраструктуры виртуализации. Предоставляет разделам службы управления разделами, службы управления виртуальными процессорами и службы управления памятью.
* **VMBus**— канальный механизм взаимодействия, используемый для взаимодействия между разделами и перечисления устройств в системах с несколькими активными виртуализированными разделами. VMBus устанавливается со службами интеграции Hyper-V.
* **VMMS, Virtual Machine Management Service**— служба управления виртуальными машинами. Отвечает за управление состоянием всех виртуальных машин в дочерних разделах.
* **VMWP, Virtual Machine Worker Process**— рабочий процесс виртуальной машины. Пользовательский компонент стека виртуализации. Рабочий процесс предоставляет службы управления виртуальной машиной из экземпляра Windows Server 2008 в родительском разделе гостевым операционным системам в дочерних разделах. Служба управления виртуальными машинами создает отдельный рабочий процесс для каждой запущенной виртуальной машины.
* **VSC, Virtualization Service Client**— клиент службы виртуализации. Экземпляр синтетического устройства, который располагается в дочернем разделе. Клиенты VSC используют аппаратные ресурсы, которые предоставляются поставщиками служб виртуализации (VSP) в родительском разделе. Они взаимодействуют с соответствующими поставщиками VSP в родительском разделе через шину VMBus для удовлетворения запросов ввода-вывода на устройство дочерних разделов.
* **VSP, Virtualization Service Provider**— поставщик служб виртуализации. Размещается в корневом разделе и обеспечивает поддержку синтетических устройств для дочерних разделов через шину виртуальной машины (VMBus).
* **WinHv, Windows Hypervisor Interface Library**— библиотека интерфейса гипервизора Windows. Библиотека WinHv является важным связующим звеном между драйверами операционной системы в разделе и гипервизором, который позволяет драйверам обращаться к гипервизору, используя стандартные правила вызова в Windows.
* **WMI, Windows Management Instrumentation**— служба управления виртуальной машиной предоставляет API-интерфейсы на основе инструментария управления Windows (WMI) для управления виртуальными машинами.