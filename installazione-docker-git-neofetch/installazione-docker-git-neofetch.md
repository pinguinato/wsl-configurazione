# Installazione di Docker (nativo in WSL) e strumenti utili (neofetch, htop, git)

Questo documento descrive come installare Docker Engine all'interno di `Ubuntu-24.04` su WSL2 e alcuni strumenti utili per gestire la distro (neofetch, htop, git).

Nota: questa guida presuppone che abbiate rimosso Docker Desktop/Rancher Desktop e che vogliate un'installazione "nativa" dentro la distro WSL. Se preferite usare ancora Docker Desktop, potete saltare la parte di installazione di Docker Engine.

---

## Prerequisiti

- Eseguire i comandi dalla distro WSL (es. `wsl -d Ubuntu-24.04`) o aprendo il terminale della distro dal menu Start.
- Avere privilegi di sudo.
- (Opzionale ma consigliato) aver abilitato systemd dentro WSL se volete avviare e abilitare il servizio Docker con `systemctl`.

---

## 1) Preparare il sistema e le chiavi del repository Docker

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg |
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Questi comandi creano la keyring directory e importano la chiave GPG ufficiale di Docker.

---

## 2) Aggiungere il repository ufficiale Docker

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Poi aggiornare l'indice dei pacchetti:

```bash
sudo apt update
```

---

## 3) Installare Docker Engine

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Note:
- I pacchetti `docker-buildx-plugin` e `docker-compose-plugin` forniscono rispettivamente buildx e la nuova esperienza `docker compose` (integrata come plugin).

---

## 4) Aggiungere il tuo utente al gruppo `docker` (opzionale, evita sudo per i comandi docker)

```bash
sudo usermod -aG docker $USER
```

Dopo questo comando è necessario aprire una nuova sessione (logout/login) oppure eseguire immediatamente:

```bash
newgrp docker
# oppure
exec su -l $USER
```

oppure semplicemente chiudere e riaprire il terminale WSL.

---

## 5) Avviare/abilitare il servizio Docker

Se avete abilitato `systemd` in WSL (vedi `wsl.conf`), potete usare:

```bash
sudo systemctl enable --now docker
sudo systemctl status docker
```

Se non avete systemd abilitato, potete avviare il demone manualmente (meno consigliato):

```bash
sudo dockerd &
```

(Nota: eseguire `dockerd` in background non è la modalità raccomandata per un sistema stabile; preferite systemd quando possibile.)

---

## 6) Verifiche dell'installazione

Eseguite i seguenti comandi per assicurarvi che tutto funzioni:

```bash
# informazioni di base
docker --version
docker info

# controlla i container in esecuzione
docker ps

# test rapido: esegue un container di prova
docker run --rm hello-world
```

Se vedete il messaggio "Hello from Docker!", significa che l'installazione è andata a buon fine.

---

## Installare `neofetch`, `htop` e `git`

È comodo installare alcuni strumenti leggeri per ispezionare rapidamente il sistema:

```bash
sudo apt update
sudo apt install -y neofetch htop git
```

- `neofetch` mostra informazioni di sistema all'avvio.
- `htop` è un monitor di processi interattivo e usabile da terminale.
- `git` è il sistema di controllo versione; probabilmente è già presente ma questa riga lo installa se manca.

Per eseguirli:

```bash
neofetch
htop
git --version
```

---

## Consigli e troubleshooting

- Se ricevete errori di permessi quando eseguite `docker`, probabilmente dovete riaprire la sessione dopo `usermod -aG docker` (vedi sopra).
- Se `systemctl` non è disponibile, verificate di aver abilitato `systemd` seguendo la sezione "Primo avvio e aggiornamento" di questo progetto.
- Se `docker run hello-world` fallisce con errori legati a runtime mancante o containerd, verificate `sudo systemctl status docker` e i log con `journalctl -u docker --no-pager -n 200` (se systemd è disponibile).
- In alternativa a Docker Engine potete considerare `Podman` (rootless) se preferite evitare demoni di sistema.

---

## Ripristino e note finali

- Se avevate immagini o volumi locali creati con Docker Desktop e li volete conservare, assicuratevi di esportarli prima di disinstallare Docker Desktop (es. `docker save`, `docker export`, `docker volume inspect` / `docker volume ls`).
- Controllate sempre la documentazione ufficiale Docker per aggiornamenti o cambiamenti di pacchetti: https://docs.docker.com/engine/install/ubuntu/
