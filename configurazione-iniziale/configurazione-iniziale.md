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
