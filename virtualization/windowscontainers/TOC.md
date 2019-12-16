# [Документация по контейнерам в Windows](index.md) 

# Обзор
## [О контейнерах Windows](about/index.md)
## [Контейнеры и виртуальные машины](about/containers-vs-vm.md)
## [Требования к системе](deploy-containers/system-requirements.md)
## [Вопросы и ответы](about/faq.md)

# Начало работы
## [Настройка среды](quick-start/set-up-environment.md)
## [Запуск первого контейнера](quick-start/run-your-first-container.md)
## [Контейнеризация примера приложения](quick-start/building-sample-app.md)

# Учебники
## Создание контейнера Windows
### [Написание Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [Оптимизация Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## Запуск в Службе Azure Kubernetes
### [Создание кластера контейнеров Windows в AKS](/azure/aks/windows-container-cli)
### [Текущие ограничения](/azure/aks/windows-node-limitations)
## Запуск в Service Fabric
### [Развертывание первого контейнера](/azure/service-fabric/service-fabric-quickstart-containers)
### [Развертывание приложения .NET в контейнере Windows](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Запуск в службе приложений Azure
### [Краткое руководство по службе приложений Azure](/azure/app-service/app-service-web-get-started-windows-container)
### [Перенос приложения ASP.NET с помощью контейнеров Windows и службы приложений Azure](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Контейнеры Linux в Windows
### [Обзор](deploy-containers/linux-containers.md)
### [Запуск первого контейнера LCOW](quick-start/quick-start-windows-10-linux.md)
## Использование контейнеров с программой предварительной оценки Windows
### [Обзор](deploy-containers/insider-overview.md)

# Понятия
## Основные сведения о контейнерах в Windows
### [Базовые образы контейнеров](manage-containers/container-base-images.md)
### [Режимы изоляции](manage-containers/hyperv-container.md)
### [Совместимость версий](deploy-containers/version-compatibility.md)
### [Элементы управления ресурсами](manage-containers/resource-controls.md)
## Docker
### [Подсистема Docker в Windows](manage-docker/configure-docker-daemon.md)
### [Удаленное управление узлом Docker в Windows](management/manage_remotehost.md)
## Оркестрация контейнеров
### [Обзор](about/overview-container-orchestrators.md)
### Kubernetes в Windows
#### [Kubernetes в Windows](kubernetes/getting-started-kubernetes-windows.md)
#### [Создание главного узла Kubernetes](kubernetes/creating-a-linux-master.md)
#### [Выбор сетевого решения](kubernetes/network-topologies.md)
#### [Присоединение рабочих ролей Windows](kubernetes/joining-windows-workers.md)
#### [Присоединение рабочих ролей Linux](kubernetes/joining-linux-workers.md)
#### [Развертывание ресурсов Kubernetes](kubernetes/deploying-resources.md)
#### [Устранение неполадок](kubernetes/common-problems.md)
#### [Kubernetes как служба Windows](kubernetes/kube-windows-services.md)
#### [Компиляция двоичных файлов Kubernetes](kubernetes/compiling-kubernetes-binaries.md)
### Service Fabric
#### [Service Fabric и контейнеры](/azure/service-fabric/service-fabric-containers-overview)
#### [Система управления ресурсами](/azure/service-fabric/service-fabric-resource-governance)
### Docker Swarm
#### [Режим Swarm](manage-containers/swarm-mode.md)
## Рабочие нагрузки
### Групповые управляемые учетные записи служб
#### [Создание групповой управляемой учетной записи службы](manage-containers/manage-serviceaccounts.md)
#### [Настройка приложения для использования групповой управляемой учетной записи службы](manage-containers/gmsa-configure-app.md)
#### [Запуск контейнера с помощью групповой управляемой учетной записи службы](manage-containers/gmsa-run-container.md)
#### [Управление контейнерами с помощью групповой управляемой учетной записи службы](manage-containers/gmsa-orchestrate-containers.md)
#### [Устранение неполадок групповой управляемой учетной записи службы](manage-containers/gmsa-troubleshooting.md)
### [Службы принтера](deploy-containers/print-spooler.md)
## Сеть
### [Обзор](container-networking/architecture.md)
### [Сетевые топологии и драйверы](container-networking/network-drivers-topologies.md)
### [Сетевая изоляция и безопасность](container-networking/network-isolation-security.md)
### [Настройка расширенных сетевых параметров](container-networking/advanced.md)
## Хранилище
### [Обзор](manage-containers/container-storage.md)
### [Постоянное хранилище](manage-containers/persistent-storage.md)
## Устройства
### [Аппаратные устройства](deploy-containers/hardware-devices-in-containers.md)
### [Ускорение с помощью GPU](deploy-containers/gpu-acceleration.md)

# Справочные материалы
## [Жизненные циклы обслуживания базовых образов](deploy-containers/base-image-lifecycle.md)
## [Оптимизация антивирусного программного обеспечения](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [Инструменты платформы контейнеров](deploy-containers/containerd.md)
## [Лицензионное соглашение для образа ОС контейнера](Images_EULA.md)

# Ресурсы
## [Примеры контейнеров](samples.md)
## [Устранение неполадок](troubleshooting.md)
## [Форум по контейнерам](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [Видеоматериалы и блоги сообщества](communitylinks.md)
