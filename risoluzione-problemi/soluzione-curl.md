# Soluzione CURL

## Scenario

Ho un progetto avviato in Windows in particolare da Powershell riesco a fare la CURL per una api tipo **http://localhost:8080/api/employees** 
esempio:

        curl http://localhost:8080/api/employees

e funziona restituendomi http OK 200, mentre se vado sulla shell Ubuntu della mia wsl lancio lo stesso comando, non funziona e ricevo:

        curl: (7) Failed to connect to localhost port 8080 after 0 ms: Couldn't connect to server

### Soluzione

Il motivo principale è che PowerShell (su Windows) e la WSL (Linux) vedono la rete in modo diverso.
Sebbene la WSL sembri integrata, in realtà gira dentro una macchina virtuale leggera con il proprio stack di rete.

Diamo prima questo comando sulla shell della WSL e vediamo quale indirizzo ip ci viene proposto:

        cat /etc/resolv.conf | grep nameserver

Proviamo ad usare l'ip che compare al posto di localhost nella nostra curl:

        curl http://172.172.10.1:8080/api/employees

Se ancora riceviamo un errore di connessione vuol dire che le cause possono essere soltanto 2:

- windows firewall non accetta connessione dalla wsl
- il server non accetta connessioni esterne

**Ipsezione**

Recuperiamo l'ip corretto del firewall:

        ip route show | grep default

Usiamo questo indirizzo IP per effettuare la CURL:

        curl http://172.172.20.1:8080/api/employees

Dovrebbe funzionare, quindi a questo punto dobbiamo fare una modifica alla configurazione della nostra WSL:

**Soluzione finale**

Mettiamo nel nostro file **.wslconfig** questa sintassi:

        [wsl2]
        networkingMode=mirrored

Salviamo il file e riavviamo con wsl --shutdown.
Facendo in questo modo potremmo lanciare la CURL dalla nostra wsl usando l'indirizzo http://localhost:8080/api/employees

**Importante**

Se non vogliamo cambiare configurazione della wsl e attivare il modo mirrored possiamo agire manualmente da powershell con questo 
comando:

        New-NetFirewallRule -DisplayName "Allow WSL" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow

Andando ad abilitare la WSL.

