# Configurazione iniziale

Se non sapete proprio nulla, iniziate da qui: https://learn.microsoft.com/it-it/windows/wsl/install

## Sommario
- [Premessa](#premessa)
- [Licenza Docker Desktop](#licenza-docker-desktop)
- [Alternative a Docker Desktop](#alternative-a-docker-desktop)
- [Passaggio a WSL "puro" (tutto dentro una distro WSL)](#passaggio-a-wsl-puro-tutto-dentro-una-distro-wsl)
    - [Il file `.wslconfig`](#il-file-wslconfig)
    - [Comandi WSL utili (da PowerShell/CMD)](#comandi-wsl-utili-da-powershellcmd)

---

## Premessa

Per lo sviluppo locale utilizzo strumenti come Java, Maven, Python, IntelliJ IDEA e Visual Studio Code. Per la gestione di immagini e container su Windows 10/11 prediligo Docker Desktop: è semplice da usare e adatto a chi non vuole perdere tempo in complesse configurazioni.

## Licenza Docker Desktop

Docker Desktop è gratuito solo per uso personale; per utilizzo aziendale è richiesta una licenza a pagamento. Se i progetti sono personali, è possibile scaricarlo, installarlo e usarlo liberamente. Verificate sempre i termini di licenza aggiornati sul sito ufficiale prima dell'uso in un contesto commerciale.

## Alternative a Docker Desktop

- **Rancher Desktop**: valida alternativa, veloce e a bassa configurazione. Svolge le stesse funzioni principali (container, immagini, runtime), ma alcuni plugin e l'interfaccia possono essere meno rifiniti; rimane comunque utile per sviluppo su Windows senza troppe customizzazioni.
- **Podman / CRI-O / microk8s / kind**: alternative più "nerd" o orientate a scenari specifici (rootless, Kubernetes locale, ecc.). Richiedono più configurazione ma sono opzioni valide se si vuole evitare Docker Desktop.

---

## Passaggio a WSL "puro" (tutto dentro una distro WSL)

Se venite da Docker Desktop o Rancher Desktop, probabilmente avete già WSL installato: questi applicativi usano il modulo Linux per Windows (WSL2) o una VM Linux per funzionare. Per passare a un ambiente basato completamente su una singola distro WSL:

1. Verificate di avere WSL2 installato e aggiornato.
2. Scegliete una distro (ad es. `Ubuntu-24.04`) e installatela come distro principale.
3. Configurate risorse e impostazioni con il file `.wslconfig` nella cartella utente di Windows se necessario.
4. Installate Docker Engine / containerd / Podman direttamente nella distro WSL, oppure usate tool che integrano con WSL.

### Il file `.wslconfig`

Il file `C:\Users\<utente>\.wslconfig` permette di definire risorse globali per WSL2 (memoria, CPU, swap). Esempio minimo:

    [wsl2]
    processors=4
    memory=8GB
    swap=2GB
    localhostForwarding=true

Note:
- `processors` = numero di CPU assegnate a WSL.
- `memory` e `swap` = limiti di memoria e swap per le VM WSL2.
- Dopo modifiche a `.wslconfig` eseguire `wsl --shutdown` per applicare.

### Comandi WSL utili (da PowerShell/CMD)

In alternativa consultare: https://learn.microsoft.com/it-it/windows/wsl/basic-commands

    wsl --version              # verifica la versione di WSL installata
    wsl --list --verbose       # lista le distro installate e il loro stato
    wsl --shutdown             # spegne tutte le istanze WSL (applica .wslconfig)
    wsl --unregister <Nome>    # disinstalla una distro WSL (es. Ubuntu)
    wsl --install -d Ubuntu-24.04  # installa una nuova distro (esempio)
    wsl --update               # aggiorna il componente WSL
    wsl --set-default-version 2    # imposta WSL versione 2 come predefinita
    wsl --set-default <nome>   # imposta la distribuzione di default

Suggerimento: eseguite i comandi da PowerShell con privilegi normali; per operazioni di rete o installazione di pacchetti dentro la distro utilizzate il terminale della distro (es. `wsl -d Ubuntu-24.04` o aprite la distro da Start).

---

## Consigli rapidi

- Tenete backup del file `.wslconfig` prima di modificarlo.
- Per sviluppo Java/Python/Node, preferite installare gli strumenti dentro la distro WSL per evitare problemi di compatibilità di path e permessi.
- Se usate IDE come IntelliJ o VS Code: preferite le estensioni Remote - WSL (VS Code) o aprite i progetti direttamente dalla distro per migliore esperienza.
- Controllate le policy aziendali prima di usare Docker Desktop in progetti non personali.

---

## Disinstallare Docker Desktop

Se volete usare WSL in modo "puro" (ossia gestire container e strumenti direttamente dentro la distro), conviene rimuovere Docker Desktop e pulire i residui. Di seguito una procedura ordinata e sicura.

1. Aggiornare WSL:
   ```bash
   wsl --update
   ```
2. Arrestare tutte le istanze WSL (applicare eventuali modifiche a `.wslconfig`):
   ```bash
   wsl --shutdown
   ```
3. Disinstallare Docker Desktop tramite **Impostazioni > App > App installate** (o Pannello di controllo). Assicuratevi che l'app sia completamente chiusa prima di procedere.
4. Verificare la rimozione delle VM/distro correlate. Se sono ancora presenti, rimuoverle manualmente:
   ```bash
   wsl --list --verbose
   wsl --unregister docker-desktop
   wsl --unregister docker-desktop-data
   ```
   Nota: i comandi `wsl --unregister` cancellano la distro/VM indicata.
5. Pulire le cartelle residue su Windows (eseguire PowerShell come amministratore). Usate i comandi seguenti con cautela: rimuovono file e directory che Docker Desktop lascia sul sistema.

```powershell
# Eseguire da PowerShell come amministratore
Remove-Item -Recurse -Force "$env:ProgramData\Docker" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:ProgramFiles\Docker" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\Docker" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:APPDATA\Docker" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:USERPROFILE\.docker" -ErrorAction SilentlyContinue
```

6. Verifiche finali:
   ```bash
   # Se avete ancora il client Docker installato
   docker context ls || echo "docker CLI non disponibile"
   wsl --list --verbose
   ```

Note e precauzioni:
- Eseguite i comandi PowerShell come amministratore. Fate un backup delle directory importanti prima di cancellare.
- Alcuni file di configurazione potrebbero trovarsi in percorsi diversi a seconda della versione o della personalizzazione dell'installazione: verificate prima di cancellare.
- Per `Rancher Desktop` vale lo stesso approccio: disinstallare l'app e rimuovere eventuali VM/directory residue per evitare conflitti.
- Dopo la pulizia è consigliato riavviare Windows per assicurarsi che non ci siano servizi in memoria.

---

## Installazione della distro `Ubuntu-24.04`

Per installare Ubuntu 24.04 in WSL ed usare un ambiente Linux completo:

1. Eseguire (da PowerShell o da una shell con privilegi normali):

```powershell
wsl --install -d Ubuntu-24.04
```

2. Al primo avvio configurare nome utente e password quando richiesto.

3. Operazioni consigliate dopo l'installazione:
   - Impostare WSL 2 come versione predefinita (se non già fatto):
     ```powershell
     wsl --set-default-version 2
     ```
   - Impostare la distro installata come predefinita (opzionale):
     ```powershell
     wsl --set-default Ubuntu-24.04
     ```
   - Verificare lo stato delle distro:
     ```bash
     wsl --list --verbose
     ```

4. Se avete modificato `C:\Users\<utente>\.wslconfig`, ricordate di eseguire `wsl --shutdown` per applicare le modifiche.

Consigli pratici:
- Aprite la distro direttamente (dal menu Start o con `wsl -d Ubuntu-24.04`) prima di installare tool complessi.
- Installate Java, Maven, Python, Node, ecc. all'interno della distro per evitare problemi di permessi e path tra Windows e Linux.
- Se prevedete di eseguire container direttamente nella distro, installate Docker Engine, Podman o containerd all'interno di `Ubuntu-24.04` seguendo la documentazione ufficiale.
