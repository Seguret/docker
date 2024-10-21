```markdown
# Installazione e Configurazione di GitLab con Docker, SSL e Volumi Persistenti

Questa documentazione descrive i passaggi per configurare GitLab CE (Community Edition) con Docker, utilizzando SSL tramite OpenSSL e volumi persistenti per i dati.

## Prerequisiti

- Docker e Docker Compose installati.
- Un dominio configurato (`<YOUR_URL>` in questo esempio).
- Certificati SSL generati tramite OpenSSL.
  
## Passaggi per la Configurazione

### 1. Creazione del file `docker-compose.yml`

Crea una directory per la configurazione di GitLab e crea il file `docker-compose.yml` con il seguente contenuto:

```yaml
version: '3'
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    restart: always
    hostname: '<YOUR_URL>'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://<YOUR_URL>'
        nginx['redirect_http_to_https'] = true
        nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.crt"
        nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.key"
        nginx['listen_https'] = true
        nginx['listen_port'] = 443
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
      - '/etc/ssl/certs/gitlab:/etc/gitlab/ssl'
    networks:
      - gitlab-network

networks:
  gitlab-network:
    driver: bridge
```

### 2. Creazione dei Certificati SSL con OpenSSL

Per abilitare HTTPS, è necessario generare certificati SSL. Esegui i seguenti comandi per creare una chiave privata e un certificato autofirmato:

```bash
mkdir -p /etc/ssl/certs/gitlab
cd /etc/ssl/certs/gitlab

# Genera una chiave privata
openssl genrsa -out gitlab.key 2048

# Crea una richiesta di firma del certificato (CSR)
openssl req -new -key gitlab.key -out gitlab.csr

# Crea il certificato autofirmato
openssl x509 -req -days 365 -in gitlab.csr -signkey gitlab.key -out gitlab.crt
```

Assicurati che i certificati siano salvati nella directory `/etc/ssl/certs/gitlab`.

### 3. Avvio dei Servizi con Docker Compose

Esegui i seguenti comandi per avviare GitLab:

```bash
docker-compose up -d
```

Questo avvierà il container GitLab con i volumi persistenti e i certificati SSL configurati.

### 4. Verifica della Configurazione

Verifica che GitLab sia accessibile all'URL HTTPS configurato (`https://<YOUR_URL>`). Puoi testare il certificato SSL e il redirect da HTTP a HTTPS con:

```bash
curl -I http://<YOUR_URL>
```

Dovresti vedere un redirect verso HTTPS (`301 Moved Permanently`).

### 5. Accesso all'Account Amministratore

Per accedere a GitLab, usa le credenziali predefinite generate al momento dell'installazione. Per trovare la password dell'utente root, esegui il seguente comando all'interno del container GitLab:

```bash
docker exec -it <nome-container-gitlab> bash
cat /etc/gitlab/initial_root_password
```

Usa queste credenziali per accedere a GitLab all'indirizzo:

```
https://<YOUR_URL>/users/sign_in
```

- **Username**: `root`
- **Password**: (la password trovata nel file `initial_root_password`)

Dopo il primo accesso, ti verrà richiesto di cambiare la password.

### 6. Riconfigurazione e Riavvio di GitLab

Se modifichi il file `docker-compose.yml` o i certificati SSL, puoi riconfigurare e riavviare GitLab con i seguenti comandi:

```bash
docker-compose down
docker-compose up -d
```

Oppure, all'interno del container:

```bash
gitlab-ctl reconfigure
gitlab-ctl restart
```

### 7. Verifica delle Porte e dei Certificati

Assicurati che le porte siano correttamente configurate per gestire il traffico HTTP (porta `80`) e HTTPS (porta `443`). Usa `nmap` o `telnet` per verificare che la porta `443` sia in ascolto:

```bash
nmap -p 443 <YOUR_URL>
```

Puoi anche verificare i certificati con il seguente comando OpenSSL:

```bash
openssl x509 -in /etc/ssl/certs/gitlab/gitlab.crt -text -noout
```

### 8. Log di GitLab e Nginx

Se riscontri problemi, puoi visualizzare i log di Nginx per diagnosticare eventuali errori:

```bash
docker exec -it <nome-container-gitlab> bash
tail -f /var/log/gitlab/nginx/gitlab_error.log
```

## Riferimenti

- [Documentazione ufficiale di GitLab](https://docs.gitlab.com/omnibus/docker/)
- [Documentazione ufficiale di Docker](https://docs.docker.com/)
