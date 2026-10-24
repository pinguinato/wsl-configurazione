# Configurazione ambiente di sviluppo Java (Ubuntu in WSL)

Questa pagina descrive come installare il JDK Azul Zulu (uno dei JDK che uso) su `Ubuntu-24.04` in WSL e come configurare l'ambiente per lo sviluppo (JAVA_HOME, Maven, suggerimenti per IDE).

Documentazione ufficiale Azul Zulu: https://docs.azul.com/core/install/linux-ca-deb

---

## Prerequisiti

- Eseguire i comandi all'interno della distro WSL (es. `wsl -d Ubuntu-24.04`) o aprendo il terminale della distro.
- Avere privilegi sudo.
- Aggiornare l'indice pacchetti prima di iniziare:

    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```

---

## Installare Azul Zulu (JDK)

1. Installare i pacchetti richiesti e creare la keyring per APT:

    ```bash
    sudo apt install -y gnupg ca-certificates curl
    sudo mkdir -p /etc/apt/keyrings
    ```

2. Importare la chiave GPG ufficiale di Azul e impostarne i permessi:

    ```bash
    curl -fsSL https://repos.azul.com/azul-repo.key | sudo gpg --dearmor -o /etc/apt/keyrings/azul.gpg
    sudo chmod 644 /etc/apt/keyrings/azul.gpg
    ```

3. Aggiungere il repository Azul alle sorgenti APT:

    ```bash
    echo "deb [signed-by=/etc/apt/keyrings/azul.gpg] https://repos.azul.com/zulu/deb stable main" \
      | sudo tee /etc/apt/sources.list.d/zulu.list > /dev/null
    ```

4. Aggiornare l'indice dei pacchetti e installare il JDK desiderato (sostituire `21` con la versione desiderata):

    ```bash
    sudo apt update
    sudo apt install -y zulu21-jdk
    ```

    Nota: il nome del pacchetto segue il pattern `zulu<VERSION>-jdk` (es. `zulu11-jdk`, `zulu17-jdk`, `zulu21-jdk`). Verificate i nomi disponibili con `apt-cache search zulu`.

---

## Verifiche post-installazione

```bash
java -version
javac -version
```

Per elencare i JDK installati e gestire le alternative di sistema:

```bash
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

Il JDK Zulu normalmente viene installato in `/usr/lib/jvm/` (es. `/usr/lib/jvm/zulu21`).

---

## Impostare JAVA_HOME

Per impostare `JAVA_HOME` per l'utente corrente, aggiungete le seguenti righe al file `~/.bashrc` o `~/.profile` (sostituendo il percorso se diverso):

```bash
# aggiungere alla fine di ~/.bashrc
export JAVA_HOME="$(dirname "$(dirname "$(readlink -f "$(which java)")")")"
export PATH="$JAVA_HOME/bin:$PATH"
```

Dopo la modifica, ricaricate il profilo o aprite una nuova shell:

```bash
source ~/.bashrc
# verifica
echo $JAVA_HOME
```

Se preferite impostare `JAVA_HOME` a livello di sistema per tutti gli utenti, aggiungete una riga in `/etc/environment` (richiede sudo, senza `export`), usando il percorso reale del JDK, e riavviate la sessione:

```text
JAVA_HOME=/usr/lib/jvm/zulu21
```

---

## Installazione Maven

Il Maven di Ubuntu 24.04 è una versione datata. Per i progetti usate il wrapper incluso nel progetto (`mvnw`), che scarica la versione giusta:

```bash
./mvnw -v
```

Se vi serve Maven di sistema o più versioni di JDK e Maven, usate SDKMAN!, seguendo la guida ufficiale su https://sdkman.io/install, poi:

```bash
sdk install maven
mvn -v
```

---

## Consigli su dove tenere i progetti (performance)

- Evitate di lavorare su progetti montati in `/mnt/c/...` perché le prestazioni I/O sono peggiori rispetto ai file presenti nella home della distro WSL. Esempio: invece di `C:\Users\...`, create `~/progetti` dentro WSL e clonate lì i sorgenti.

    ```bash
    mkdir -p ~/progetti
    cd ~/progetti
    git clone <your-repo>
    ```

- Lavorare direttamente sulla filesystem di WSL migliora velocità di build (Maven, Gradle) e l'indicizzazione degli IDE.

---

## Aprire i progetti dagli IDE (VS Code e IntelliJ)

### Visual Studio Code

- Metodo consigliato (Remote - WSL): aprite la cartella del progetto dentro la distro e lanciate `code .` dalla shell WSL. Questo apre VS Code con la sessione remota sulla distro e usa tool e JDK installati lì.

    ```bash
    cd ~/progetti/mio-progetto
    code .
    ```

- In alternativa, dal VS Code lato Windows usate l'estensione "Remote - WSL" e connettetevi alla distro.

### IntelliJ IDEA

- Da IntelliJ (Windows) potete aprire progetti in WSL usando il percorso speciale `\\wsl.localhost\Ubuntu-24.04\home\<utente>\progetti\mio-progetto`.
- IntelliJ rileverà che il progetto risiede in WSL e potrà utilizzare il JDK/Maven presenti nella distro (potrebbe chiedere di configurare gli SDK corretti). In caso di lentezza preferite aprire il progetto direttamente dalla shell WSL usando VS Code o usare IntelliJ sul WSL (con supporto WSL integrato nelle versioni recenti).

---

## Configurare IntelliJ (Windows) per usare JDK e Maven dalla WSL (guida passo‑passo)

Questa sezione mostra come configurare IntelliJ IDEA in esecuzione su Windows perché usi il JDK e Maven già installati nella vostra distribuzione WSL (`Ubuntu-24.04`). Le istruzioni sono valide per le versioni recenti di IntelliJ (2021+); alcuni nomi di menu possono variare leggermente a seconda della versione.

Prerequisiti:
- IntelliJ IDEA installato su Windows (Community o Ultimate).
- JDK e Maven installati nella WSL (vedi sezioni sopra).
- Il progetto salvato nel filesystem di WSL (consigliato) oppure accessibile tramite `\\wsl.localhost\`.

Passaggi:

1) Aprire il progetto in IntelliJ

- Metodo consigliato: aprite il progetto direttamente dalla cartella WSL usando il percorso UNC di Windows, ad es.: `\\wsl.localhost\Ubuntu-24.04\home\<utente>\progetti\mio-progetto` (File > Open...).
- In alternativa potete aprire IntelliJ e usare "Open" puntando a quel percorso.

2) Aggiungere il JDK della WSL come SDK di progetto

- Aprite `File > Project Structure...` (o premi Ctrl+Alt+Shift+S).
- Nella sezione `SDKs` cliccate sul `+` per aggiungere un nuovo SDK.
  - Se la vostra versione di IntelliJ supporta l'opzione "Add SDK from WSL" sceglietela e selezionate la distro (es. `Ubuntu-24.04`) e il path del JDK (es. `/usr/lib/jvm/zulu21`).
  - Se non è disponibile l'opzione WSL, selezionate "JDK" e navigate nel percorso Windows corrispondente al path della WSL via UNC: `\\wsl.localhost\Ubuntu-24.04\usr\lib\jvm\zulu21`.
- Dopo l'aggiunta, impostate questo SDK come `Project SDK` nella sezione `Project` (sempre dentro Project Structure).

3) Configurare Maven per usare il Maven della WSL

- Aprite `File > Settings > Build, Execution, Deployment > Build Tools > Maven`.
- Per la voce `Maven home directory` potete scegliere:
  - `Bundled` (se volete usare il Maven integrato in IntelliJ), oppure
  - `...` (Browse) e puntare al Maven installato nella WSL via UNC: `\\wsl.localhost\Ubuntu-24.04\usr\share\maven` (o al path corretto dove `maven` è installato nella distro).
- Nella sezione `Importing` impostate `JDK for importer` sullo stesso SDK appena configurato (così l'import userà il JDK WSL).

4) Impostare il Terminale integrato per usare la shell WSL (consigliato)

- Aprite `File > Settings > Tools > Terminal`.
- Impostate `Shell path` su `wsl.exe` (eventualmente aggiungendo `-d Ubuntu-24.04` per puntare alla distro specifica):

    ```text
    wsl.exe -d Ubuntu-24.04
    ```

- In questo modo il terminale integrato aprirà una shell dentro la WSL e i comandi `mvn`, `java`, `javac` saranno quelli della distribuzione.

5) Sincronizzare/ricostruire il progetto

- Dopo le modifiche aprite `View > Tool Windows > Maven` e cliccate sul pulsante di refresh per ricaricare i progetti Maven.
- Eseguite una rebuild (`Build > Rebuild Project`) per verificare che l'SDK e Maven funzionino correttamente.

6) Verifiche finali

- Aprite il Terminale integrato e verificate:

    ```bash
    java -version
    javac -version
    mvn -v
    echo $JAVA_HOME
    ```

  Dovreste vedere le versioni installate nella WSL e il `JAVA_HOME` che punta al percorso WSL.

Note e suggerimenti:
- Se vedete lentezza o problemi di indexing, assicuratevi che il progetto si trovi sul filesystem di WSL (non su `C:\` montato). Lavorare su `\\wsl.localhost\...` è consigliato.
- Se l'IDE non riconosce il JDK o Maven via UNC, provate ad aggiornare IntelliJ all'ultima versione o ad usare l'opzione "Add SDK from WSL" se presente.
- In alternativa, per una integrazione più completa, potete utilizzare JetBrains Gateway / IntelliJ WSL backend (se disponibile nella vostra versione) che esegue il backend dell'IDE nella WSL e fornisce esperienza nativa.

---

## Ulteriori note e troubleshooting

- Se `java` o `javac` non vengono trovati dopo l'installazione, verificate il PATH e le alternative (`update-alternatives`).
- Se usate IntelliJ e riscontrate problemi di indexing, assicuratevi di aprire il progetto dal filesystem della distro (non da `/mnt/c`), oppure aumentate le risorse dedicate alla VM WSL tramite `~/.wslconfig` e riavviate con `wsl --shutdown`.
- Per rimuovere Zulu:

    ```bash
    sudo apt remove --purge -y zulu*-jdk
    sudo apt autoremove -y
    ```
