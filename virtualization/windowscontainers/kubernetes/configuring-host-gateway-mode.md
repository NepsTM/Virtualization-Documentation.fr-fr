# <a name="host-gateway-mode"></a>Mode passerelle hôte #
Une des options disponibles pour la mise en réseau Kubernetes est le *mode hôte passerelle*, qui implique la configuration d’itinéraires statiques entre les sous-réseaux des pods sur tous les nœuds.


## <a name="configuring-static-routes--linux"></a>Configuration d’itinéraires statiques | Linux ##
Pour ce faire, nous utilisons `iptables`. Remplacez (ou définissez) la variable `$CLUSTER_PREFIX` avec le sous-réseau que tous les pods utiliseront:

```bash
$CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

Cela configure uniquement un NAT de base pour les pods. À présent, nous devons faire en sorte que l’ensemble du trafic destiné aux pods passent par l’interface principale. Là encore, remplacez la variable `$CLUSTER_PREFIX` si nécessaire, et `eth0` si cela s’applique:

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

Enfin, nous devons ajouter la passerelle de saut suivant sur la base **une par nœud**. Par exemple, si le premier nœud est un nœud Windows sur `192.168.1.0/16`, alors:

```bash
sudo route add -net $CLUSTER.1.0 netmask 255.255.255.0 gw $CLUSTER.1.2 dev eth0
```

Un itinéraire similaire doit être ajouté *pour* chaque nœud du cluster, *sur* chaque nœud du cluster.


<a name="explanation-2-suffix"></a>
> [!Important]  
> Pour les nœuds Windows **uniquement**, la passerelle est le `.2`du sous-réseau. Pour Linux, c’est généralement toujours le `.1`. Cette anomalie est due au fait que l’adresse `.1` est réservée en tant que passerelle pour l’adaptateur réseau afin de relier le réseau hôte et le réseau virtuel de pods.


## <a name="configuring-static-routes--windows"></a>Configuration d’itinéraires statiques | Windows ##
Pour ce faire, nous utilisons `New-NetRoute`. Il existe un script automatisé, `AddRoutes.ps1`, disponible dans [ce référentiel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1). Vous devez connaître l’adresse IP du *master Linux* et la passerelle par défaut du nœud Windows de l’adaptateur *externe* (pas de la passerelle de pod). Alors:


```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
wget $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3 -Gateway 10.1.3.1
```
