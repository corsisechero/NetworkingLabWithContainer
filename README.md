# NetworkingLabWithContainer
Lab di Networking con ContainerLab

# Containerlab: Esercizi di Rete

## Esercizio 1: Topologia con due router FRR

### 1. Eliminazione della topologia precedente
Assicurarsi di eliminare la topologia precedente:
```bash
sudo containerlab destroy --topo frr_router_topology.yaml
docker ps -a
docker rm -f $(docker ps -a -q -f name=clab-)
```

### 2. Nuovo file di configurazione: `frr_router_topology.yaml`
```yaml
name: frr-router-topology

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
      exec:
        - sysctl -w net.ipv4.ip_forward=1
    r2:
      kind: linux
      image: frrouting/frr:latest
      exec:
        - sysctl -w net.ipv4.ip_forward=1

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
```

### 3. Avvio della topologia
```bash
sudo containerlab deploy --topo frr_router_topology.yaml
```

### 4. Configurazione dei router
- Configurazione R1:
```bash
sudo docker exec -it clab-frr-router-topology-r1 /bin/sh
mkdir -p /etc/frr
echo "service integrated-vtysh-config" > /etc/frr/vtysh.conf
ip addr add 192.168.1.1/24 dev eth1
ip link set eth1 up
ip addr show eth1
```
- Configurazione R2:
```bash
sudo docker exec -it clab-frr-router-topology-r2 /bin/sh
mkdir -p /etc/frr
echo "service integrated-vtysh-config" > /etc/frr/vtysh.conf
ip addr add 192.168.1.2/24 dev eth1
ip link set eth1 up
ip addr show eth1
```

### 5. Verifica della connettività
```bash
sudo docker exec -it clab-frr-router-topology-r1 ping 192.168.1.2
sudo docker exec -it clab-frr-router-topology-r2 ping 192.168.1.1
```

---

## Esercizio 2: Topologia con Router e PC

### 1. Eliminazione della topologia precedente
```bash
sudo containerlab destroy --topo frr_pc_topology.yaml
```

### 2. Nuovo file di configurazione: `frr_pc_topology.yaml`
```yaml
name: frr-pc-topology

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:latest

  links:
    - endpoints: ["r1:eth1", "pc1:eth1"]
```

### 3. Avvio della topologia
```bash
sudo containerlab deploy --topo frr_pc_topology.yaml
```

### 4. Configurazione dei dispositivi
- Configurazione R1:
```bash
sudo docker exec -it clab-frr-pc-topology-r1 /bin/sh
ip addr add 192.168.1.1/24 dev eth1
ip link set eth1 up
```
- Configurazione PC1:
```bash
sudo docker exec -it clab-frr-pc-topology-pc1 /bin/sh
ip addr add 192.168.1.2/24 dev eth1
ip link set eth1 up
ip route add default via 192.168.1.1
```

### 5. Verifica della connettività
```bash
sudo docker exec -it clab-frr-pc-topology-r1 ping 192.168.1.2
sudo docker exec -it clab-frr-pc-topology-pc1 ping 192.168.1.1
```

---

## Esercizio 3: Topologia PC1 ↔ R1 ↔ R2 ↔ PC2

### 1. Eliminazione della topologia precedente
```bash
sudo containerlab destroy --topo frr_dual_router_topology.yaml
docker ps -a
docker rm -f $(docker ps -a -q -f name=clab-)
```

### 2. Configurazione: `frr_dual_router_topology.yaml`
```yaml
name: frr-dual-router-topology

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
    r2:
      kind: linux
      image: frrouting/frr:latest
    pc1:
      kind: linux
      image: alpine:latest
    pc2:
      kind: linux
      image: alpine:latest

  links:
    - endpoints: ["r1:eth1", "pc1:eth1"]
    - endpoints: ["r2:eth1", "pc2:eth1"]
    - endpoints: ["r1:eth2", "r2:eth2"]
```

### 3. Avvio della topologia
```bash
sudo containerlab deploy --topo frr_dual_router_topology.yaml
```

### 4. Configurazione Router R1 e R2
- R1:
```bash
sudo docker exec -it clab-frr-dual-router-topology-r1 vtysh
conf t
interface eth1
 ip address 192.168.1.1/24
!
interface eth2
 ip address 10.0.0.1/24
!
ip route 192.168.2.0/24 10.0.0.2
end
write
```
- R2:
```bash
sudo docker exec -it clab-frr-dual-router-topology-r2 vtysh
conf t
interface eth1
 ip address 192.168.2.1/24
!
interface eth2
 ip address 10.0.0.2/24
!
ip route 192.168.1.0/24 10.0.0.1
end
write
```

### 5. Configurazione dei PC
- PC1:
```bash
sudo docker exec -it clab-frr-dual-router-topology-pc1 /bin/sh
ip addr add 192.168.1.2/24 dev eth1
ip route add default via 192.168.1.1
```
- PC2:
```bash
sudo docker exec -it clab-frr-dual-router-topology-pc2 /bin/sh
ip addr add 192.168.2.2/24 dev eth1
ip route add default via 192.168.2.1
```

### 6. Verifica della connettività
```bash
sudo docker exec -it clab-frr-dual-router-topology-pc1 ping 192.168.2.2
sudo docker exec -it clab-frr-dual-router-topology-pc2 ping 192.168.1.2
```

### 7. Rimozione della topologia
```bash
sudo containerlab destroy --topo frr_dual_router_topology.yaml
```

---

## Conclusioni
Questi esercizi mostrano come configurare router FRR e container PC utilizzando Containerlab. L'abilitazione dell'IP forwarding e la corretta configurazione delle rotte sono fondamentali per garantire la connettività.
```
