# Primo avvio e aggiornamento

Avviare WSL da un qualsiasi terminale (PowerShell, CMD o Windows Terminal):

```bash
wsl
```

Una volta dentro la distribuzione (es. `Ubuntu-24.04`) eseguire i seguenti comandi per aggiornare pacchetti e dipendenze:

```bash
sudo apt update && sudo apt upgrade -y
```

## Abilitare systemd

Per far funzionare i servizi di sistema (ad es. Docker, systemd-based services) all'interno di WSL2 è possibile abilitare systemd modificando il file `/etc/wsl.conf` nella distro.

> Nota: la funzionalità richiede una versione recente di WSL e Windows 11; se non è disponibile, verificate gli aggiornamenti di WSL (`wsl --update`) e di Windows.

1. Aprire il file con un editor (qui usiamo `nano`):

```bash
sudo nano /etc/wsl.conf
```

2. Inserire queste righe (o modificarle se già presenti):

```ini
[boot]
systemd=true
```

3. Salvare ed uscire dall'editor (in `nano`: Ctrl+O, Invio, Ctrl+X).

4. Uscire dalla distro ed arrestare WSL per applicare la modifica:

```bash
exit
wsl --shutdown
```

5. Riavviare la distro e verificare lo stato di systemd:

```bash
wsl -d Ubuntu-24.04
systemctl is-system-running
```

Lo stato tipico quando systemd è operativo è `running`. Altri stati possibili: `degraded`, `starting`, `stopped`, ecc. Se ricevete un errore relativo a systemd, verificate la versione di WSL e i log di sistema.

## Note pratiche e suggerimenti

- Eseguite i comandi di aggiornamento e le modifiche a `wsl.conf` con attenzione. Fate un backup del file prima di modificarlo, se necessario.
- Alcuni servizi potrebbero non inizializzarsi immediatamente: attendete qualche secondo e riprovate.
- Se incontrate problemi, controllate i log con `journalctl -xe` (eseguito come root all'interno della distro) o `dmesg`.
- Per operazioni amministrative su Windows (es. aggiornamento WSL o modifiche ai file di sistema), usate PowerShell con privilegi amministrativi.

## Troubleshooting rapido

- `wsl --version` — verifica la versione di WSL installata.
- `wsl --list --verbose` — mostra le distro installate e il loro stato.
- Se dopo `wsl --shutdown` il cambio a `wsl.conf` non sembra applicarsi, riavviate Windows.

