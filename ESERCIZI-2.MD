# Containerlab: Topologia PC1 ↔ Switch ↔ Router ↔ Switch ↔ PC2

## 1. Obiettivo della Topologia
- **2 PC** (PC1 e PC2) collegati a un **Switch**.
- Lo **Switch** collegato a un **Router FRR (R1)**.
- Il **Router R1** funge da gateway per entrambi i PC.
- Verifica della connettività tra i PC e il router tramite ping.

---

## 2. Pulizia della Topologia Precedente
Prima di creare la nuova topologia, rimuovi eventuali configurazioni precedenti:

```bash
sudo containerlab destroy --topo frr_switch_router_topology.yaml
```

Rimuovi eventuali container residui:

```bash
docker ps -a
docker rm -f $(docker ps -a -q -f name=clab-)
```

---

## 3. File di Configurazione: frr_switch_router_topology.yaml

Crea il file con il seguente contenuto:

```yaml
name: frr-switch-router-topology

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    switch1:
      kind: linux
      image: alpine:latest
      exec:
        - apk update
        - apk add iproute2
        - ip link add name br0 type bridge
        - ip link set dev br0 up
    pc1:
      kind: linux
      image: alpine:latest
    pc2:
      kind: linux
      image: alpine:latest

  links:
    - endpoints: ["r1:eth1", "switch1:eth1"]
    - endpoints: ["pc1:eth1", "switch1:eth2"]
    - endpoints: ["pc2:eth1", "switch1:eth3"]
```

---

## 4. Avvio della Topologia

Avvia la topologia con il comando:

```bash
sudo containerlab deploy --topo frr_switch_router_topology.yaml
```

---

## 5. Configurazione dello Switch (Bridge)

Accedi allo **switch1** per verificare il bridge:

```bash
sudo docker exec -it clab-frr-switch-router-topology-switch1 /bin/sh
```

Verifica la configurazione del bridge:

```bash
bridge link
ip link show br0
```

Associa le interfacce al bridge:

```bash
ip link set eth1 master br0
ip link set eth2 master br0
ip link set eth3 master br0
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up
```

---

## 6. Configurazione del Router (R1) tramite VTYSH

Accedi al router R1:

```bash
sudo docker exec -it clab-frr-switch-router-topology-r1 vtysh
```

Esegui la configurazione:

```
conf t
interface eth1
 ip address 192.168.1.1/24
 no shutdown
!
ip forwarding
end
write
exit
```

### Abilitazione del Forwarding IP:

Abilita il forwarding IP sul router:

```bash
sudo docker exec -it clab-frr-switch-router-topology-r1 /bin/sh
sysctl -w net.ipv4.ip_forward=1
exit
```

---

## 7. Configurazione dei PC

### PC1

Accedi al PC1:

```bash
sudo docker exec -it clab-frr-switch-router-topology-pc1 /bin/sh
```

Configurazione dell'interfaccia:

```bash
ip addr add 192.168.1.2/24 dev eth1
ip link set eth1 up
ip route del default
ip route add default via 192.168.1.1
```

---

### PC2

Accedi al PC2:

```bash
sudo docker exec -it clab-frr-switch-router-topology-pc2 /bin/sh
```

Configurazione dell'interfaccia:

```bash
ip addr add 192.168.1.3/24 dev eth1
ip link set eth1 up
ip route del default
ip route add default via 192.168.1.1
```

---

## 8. Verifica della Connettività

### Ping tra i PC:
- Da **PC1** a **PC2**:

```bash
sudo docker exec -it clab-frr-switch-router-topology-pc1 ping 192.168.1.3
```

- Da **PC2** a **PC1**:

```bash
sudo docker exec -it clab-frr-switch-router-topology-pc2 ping 192.168.1.2
```

---

### Ping tra i PC e il Router:
- Da **PC1** a **R1**:

```bash
sudo docker exec -it clab-frr-switch-router-topology-pc1 ping 192.168.1.1
```

- Da **PC2** a **R1**:

```bash
sudo docker exec -it clab-frr-switch-router-topology-pc2 ping 192.168.1.1
```

---

## 9. Rimozione della Topologia

Quando hai terminato i test, rimuovi la topologia:

```bash
sudo containerlab destroy --topo frr_switch_router_topology.yaml
```

---

## Note Finali:
1. **Bridge Linux come Switch:**  
   - Utilizzando il tipo **`linux`** con un **bridge (br0)** creato manualmente, lo switch funziona correttamente.  
   - Le interfacce **eth1**, **eth2**, **eth3** sono collegate al bridge **br0**.

2. **FRR come Router:**  
   - Il router **R1** utilizza **FRRouting (FRR)** per gestire l'inoltro tra i PC.  
   - L'IP forwarding è abilitato sia tramite **vtysh** sia a livello di sistema.  

3. **Configurazione dei PC:**  
   - Ogni PC ha l'indirizzo IP assegnato correttamente e il gateway impostato verso il router.  

---

## Test di Connettività:
- I PC possono comunicare tra loro tramite lo switch.
- I PC possono comunicare con il router.
- Il router instrada correttamente i pacchetti tra i dispositivi.
```
