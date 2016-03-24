# Спецификации гипервизора

## Верхнеуровневая функциональная спецификация гипервизора

Верхнеуровневая функциональная спецификация (TLFS) гипервизора Hyper-V описывает видимое извне поведение гипервизора для других компонентов операционной системы. Эта спецификация предназначена для разработчиков операционных систем на виртуальных машинах.

> На спецификацию распространяется действие Обещания в отношении открытых спецификаций корпорации Майкрософт. Дополнительные сведения см. в статье [Обещание в отношении открытых спецификаций корпорации Майкрософт](https://msdn.microsoft.com/en-us/openspecifications).

#### Скачать

 Выпуск| Документ
--- | ---
 Windows Server 2012 R2 (редакция B)| [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
 Windows Server 2012 R2| [Hypervisor Top Level Functional Specification v4.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
 Windows Server 2012| [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
 Windows Server 2008 R2| [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## Требования для реализации интерфейса гипервизора Майкрософт

Для операционных систем Windows требуется ограниченный набор интерфейсов гипервизора для запуска на гостевой виртуальной машине (также известной как интерфейс HV#1). Кроме того, гипервизор, совместимый с Майкрософт, может реализовать несколько дополнительных функций. Эти параметры изменят поведение Windows в виртуальной машине. В разделе "Требования для реализации интерфейса гипервизора Майкрософт" описаны обязательные и дополнительные функции, реализованные совместимым с Майкрософт гипервизором.

#### Скачать

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)



<!--HONumber=Mar16_HO2-->
