---
title: "Спецификации гипервизора"
description: "Спецификации гипервизора"
keywords: "windows 10, hyper-v"
author: theodthompson
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: a92ba96f26ae2881d2c6013e8b8dcf7bafe69a47

---

# Спецификации гипервизора

## Верхнеуровневая функциональная спецификация гипервизора

Верхнеуровневая функциональная спецификация (TLFS) гипервизора Hyper-V описывает видимое извне поведение гипервизора для других компонентов операционной системы. Эта спецификация предназначена для разработчиков операционных систем на виртуальных машинах.
  
> На спецификацию распространяется действие Обещания в отношении открытых спецификаций корпорации Майкрософт.  Дополнительные сведения см. в статье [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications) (Обещание в отношении открытых спецификаций корпорации Майкрософт).  

#### Скачать
Выпуск | Документ
--- | ---
Windows Server 2012 R2 (редакция B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 R2 | [Hypervisor Top Level Functional Specification v4.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## Требования для реализации интерфейса гипервизора Майкрософт

Для операционных систем Windows требуется ограниченный набор интерфейсов гипервизора для запуска на гостевой виртуальной машине (также известной как интерфейс HV#1). Кроме того, гипервизор, совместимый с Майкрософт, может реализовать несколько дополнительных функций. Эти параметры изменят поведение Windows в виртуальной машине. В разделе "Требования для реализации интерфейса гипервизора Майкрософт" описаны обязательные и дополнительные функции, реализованные совместимым с Майкрософт гипервизором.

#### Скачать

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)


<!--HONumber=Oct16_HO4-->


