### Documentazione per il file `docker-compose.yml` di Vault

Questo file `docker-compose.yml` è configurato per avviare un container Docker che esegue HashiCorp Vault, versione `1.13.3`, con archiviazione persistente su file system. La configurazione include l'abilitazione dell'interfaccia utente web di Vault e la disabilitazione della funzione `mlock` per evitare errori di memoria.

#### Versione

```yaml
version: '3.8'
```
La versione del formato `docker-compose`. In questo caso, utilizziamo la versione 3.8.

#### Servizi

Definisce i servizi che verranno eseguiti. In questo esempio, c'è un solo servizio: `vault`.

##### Servizio `vault`

```yaml
services:
  vault:
    image: vault:1.13.3
```
Utilizza l'immagine Docker ufficiale di Vault versione `1.13.3`.

```yaml
    container_name: vault
```
Specifica un nome per il container Docker, in questo caso `vault`.

```yaml
    environment:
      VAULT_LOCAL_CONFIG: |
        {
          "storage": {
            "file": {
              "path": "/vault/file"
            }
          },
          "listener": {
            "tcp": {
              "address": "0.0.0.0:8200",
              "tls_disable": 1
            }
          },
          "default_lease_ttl": "168h",
          "max_lease_ttl": "720h",
          "ui": true,
          "disable_mlock": true
        }
```
Configura l'ambiente di Vault. La variabile `VAULT_LOCAL_CONFIG` definisce la configurazione di Vault in formato JSON:

- `"storage"`: Configura il backend di archiviazione per Vault. In questo caso, viene utilizzato il file system con il percorso `/vault/file`.
- `"listener"`: Configura Vault per ascoltare le connessioni TCP sull'indirizzo `0.0.0.0` sulla porta `8200`. `tls_disable: 1` disabilita TLS per semplificare la configurazione iniziale.
- `"default_lease_ttl"`: Imposta il TTL (Time To Live) predefinito per i segreti a 168 ore (7 giorni).
- `"max_lease_ttl"`: Imposta il TTL massimo per i segreti a 720 ore (30 giorni).
- `"ui"`: Abilita l'interfaccia utente web di Vault.
- `"disable_mlock"`: Disabilita l'uso di `mlock` per evitare errori di allocazione della memoria.

```yaml
    ports:
      - "8200:8200"
```
Mappa la porta `8200` del container alla porta `8200` dell'host, rendendo Vault accessibile all'indirizzo `http://localhost:8200`.

```yaml
    volumes:
      - ./data:/vault/file
```
Monta la directory `./data` dell'host sulla directory `/vault/file` del container, fornendo archiviazione persistente per Vault.

```yaml
    networks:
      - vault-net
```
Collega il container `vault` alla rete Docker `vault-net`.

```yaml
    command: ["vault", "server", "-config=/vault/config/vault-config.json"]
```
Specifica il comando per avviare Vault con la configurazione definita nel file `/vault/config/vault-config.json`.

```yaml
    entrypoint: /bin/sh -c 'mkdir -p /vault/config && echo "$$VAULT_LOCAL_CONFIG" > /vault/config/vault-config.json && vault server -config=/vault/config/vault-config.json'
```
Utilizza `sh` per creare la directory di configurazione di Vault, scrivere la configurazione di Vault in un file, e avviare il server Vault con tale configurazione.

#### Reti

```yaml
networks:
  vault-net:
    driver: bridge
```
Configura una rete Docker denominata `vault-net` utilizzando il driver `bridge`.

### Esecuzione

Per avviare il servizio Vault definito in questo file `docker-compose.yml`, eseguire i seguenti comandi:

1. **Crea la directory del progetto e il file `docker-compose.yml`**:

   ```bash
   mkdir vault-project
   cd vault-project
   touch docker-compose.yml
   ```

2. **Incolla il contenuto sopra nel file `docker-compose.yml`**.

3. **Crea la directory per l'archiviazione dei dati**:

   ```bash
   mkdir data
   ```

4. **Avvia Vault**:

   ```bash
   docker-compose up -d
   ```

5. **Accedi a Vault**:

   Puoi accedere a Vault tramite l'indirizzo [http://localhost:8200](http://localhost:8200). Utilizza il token root per autenticarti.