---
title: Спецификации гипервизора
description: Спецификации гипервизора
keywords: windows10, hyper-v
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: afbbcf120961081191aaf9051866427c9ce1478e
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621282"
---
# <a name="hypervisor-specifications"></a>Спецификации гипервизора

## <a name="hypervisor-top-level-functional-specification"></a>Верхнеуровневая функциональная спецификация гипервизора

Верхнеуровневая функциональная спецификация (TLFS) гипервизора Hyper-V описывает видимое извне поведение гипервизора для других компонентов операционной системы. Эта спецификация предназначена для разработчиков операционных систем на виртуальных машинах.
  
> На спецификацию распространяется действие Обещания в отношении открытых спецификаций корпорации Майкрософт.  Дополнительные сведения см. в статье [Microsoft Open Specification Promise](https://docs.microsoft.com/openspecs/dev_center/ms-devcentlp/51a0d3ff-9f77-464c-b83f-2de08ed28134) (Обещание в отношении открытых спецификаций корпорации Майкрософт).  

#### <a name="download"></a>Скачать
Выпуск | Документ
--- | ---
Windows Server 2016 (редакция С) | [Hypervisor Top Level Functional Specification v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2 (редакция B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server2008R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Требования для реализации интерфейса гипервизора Майкрософт

Спецификация TLFS содержит полное описание всех аспектов архитектуры гипервизора, характерных для продуктов Майкрософт, который объявляется гостевым виртуальным машинам как интерфейс «HV #1».  Однако не все интерфейсы, описанные в спецификации TLFS, необходимы для реализации в гипервизоре сторонних разработчиков, желающих объявить совместимость со спецификацией гипервизора Майкрософт HV#1. В документе «Требования по реализации интерфейса гипервизора Майкрософт» описан минимальный набор интерфейсов гипервизора, которые должны быть реализованы каждым гипервизором, для которого заявляется совместимость с интерфейсом Майкрософт HV#1.

#### <a name="download"></a>Скачивание

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
