# <a name="host-gateway-mode"></a>Режим узел-шлюз #
Один из доступных вариантов для сетей Kubernetes называется *режимом узел-шлюз*, который реализует конфигурацию статических маршрутов между подсетями модулей pod на всех узлах.


## <a name="configuring-static-routes--linux"></a>Настройка статических маршрутов | Linux ##
Для этого мы используем `iptables`. Замените (или задайте) переменную `$CLUSTER_PREFIX`, используя сокращенную подсеть, которую будут использовать все модули pod:

```bash
CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

При этом настраивается базовый механизм трансляции сетевых адресов для модулей pod. Теперь нам необходимо направить весь трафик, предназначенный для модулей pod, через основной интерфейс. Снова замените переменную `$CLUSTER_PREFIX`, а также `eth0` при необходимости:

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

Наконец, нам следует добавить шлюз следующего прыжка на уровне **узла**. Например, если первый узел является узлом Windows на `192.168.1.0/16`, то:

```bash
sudo route add -net $CLUSTER_PREFIX.1.0 netmask 255.255.255.0 gw $CLUSTER_PREFIX.1.2 dev eth0
```

Аналогичный маршрут следует добавить *для* каждого узла в кластере, *на* каждом узле в кластере.


<a name="explanation-2-suffix"></a>
> [!Important]  
> **Только** для узлов Windows шлюзом будет адрес `.2` в подсети. Для Linux значение всегда будет равно `.1`. Этот аномально, так как адрес `.1` резервируется в качестве шлюза для сетевого адаптера, который соединяет сеть узла и виртуальную сеть модуля pod.


## <a name="configuring-static-routes--windows"></a>Настройка статических маршрутов | Windows ##
Для этого мы используем `New-NetRoute`. В [этом репозитории](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1) доступен автоматизированный сценарий `AddRoutes.ps1`. Вам необходимо знать IP-адрес *главного узла Linux* и шлюз по умолчанию *внешнего* адаптера узла Windows (а не шлюз модуля pod). Затем:

```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
wget $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3 -Gateway 10.1.3.1
```
