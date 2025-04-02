# NetworkingLabWithContainer
# Installazione di Containerlab

Containerlab è uno strumento che consente di creare topologie di rete containerizzate con semplicità. Utilizza container Docker per simulare dispositivi di rete, come router e switch.

## Requisiti

- Docker (installato e configurato correttamente)
- Sistema operativo Linux

## Passaggi di Installazione

### 1. Installazione di Containerlab

Esegui il comando seguente per installare Containerlab utilizzando uno script Bash ufficiale:
```bash
sudo bash -c "$(curl -L https://get.containerlab.dev)"
```

Oppure puoi installarlo manualmente seguendo i passaggi indicati nella [documentazione ufficiale](https://containerlab.dev).

### 2. Verifica dell'Installazione

Controlla che Containerlab sia stato installato correttamente:
```bash
containerlab version
```

### 3. Installazione di Docker

Se non hai già Docker installato, esegui i seguenti comandi per installarlo su una distribuzione basata su Debian/Ubuntu:
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

### 4. Aggiunta dell'utente al gruppo Docker (opzionale)

Per evitare di usare `sudo` con Docker:
```bash
sudo usermod -aG docker $USER
```
Dopo aver eseguito il comando, disconnettiti e riconnettiti per applicare le modifiche.

### 5. Test di Containerlab

Esegui un semplice test per verificare il funzionamento:
```bash
containerlab deploy --help
```

## Link al progetto

Puoi trovare ulteriori informazioni e la documentazione completa nel repository ufficiale su GitHub:
[Containerlab su GitHub](https://github.com/srl-labs/containerlab)

## Risoluzione dei Problemi

Se riscontri problemi durante l'installazione o l'uso, consulta la [documentazione ufficiale](https://containerlab.dev) o apri una issue nel [repository GitHub](https://github.com/srl-labs/containerlab/issues).

---
```
