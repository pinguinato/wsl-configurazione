# Soluzione CURL: chiamare da WSL un server avviato su Windows

## Scenario

Il progetto gira su Windows (es. Spring Boot su `http://localhost:8080`). Da PowerShell funziona:

```powershell
curl http://localhost:8080/api/employees
```

Dalla shell Ubuntu di WSL, lo stesso comando fallisce:

```text
curl: (7) Failed to connect to localhost port 8080 after 0 ms: Couldn't connect to server
```

## Perché succede

PowerShell e WSL usano stack di rete diversi. WSL2 gira in una macchina virtuale con una propria interfaccia di rete. Per default (modalità NAT) `localhost` dentro WSL non punta all'host Windows, a meno che il server Windows non sia raggiungibile su quell'interfaccia.

## Soluzione consigliata: modalità `mirrored`

Requisiti: Windows 11 22H2 o successivo e WSL aggiornata (`wsl --update`).

1. Su Windows, creare o modificare `C:\Users\<utente>\.wslconfig` (fare prima una copia di backup):

    ```ini
    [wsl2]
    networkingMode=mirrored
    ```

2. Da PowerShell, riavviare WSL:

    ```powershell
    wsl --shutdown
    ```

3. Riaprire la distro e verificare:

    ```bash
    curl http://localhost:8080/api/employees
    ```

In modalità `mirrored`, WSL condivide la rete dell'host e `localhost` funziona in entrambe le direzioni.

## Diagnosi, se non funziona

**a) Il server deve accettare connessioni esterne.** Su Windows, in PowerShell:

```powershell
netstat -ano | findstr :8080
```

- `0.0.0.0:8080` o `[::]:8080`: il server accetta connessioni da tutte le interfacce.
- `127.0.0.1:8080` o `[::1]:8080`: il server accetta solo connessioni locali. Va configurato per ascoltare su `0.0.0.0`. Per Spring Boot, in `application.properties`:

    ```properties
    server.address=0.0.0.0
    ```

**b) Il firewall di Windows deve accettare il traffico da WSL.** Eseguire PowerShell come amministratore. Il nome dell'adattatore WSL può variare: verificarlo con `Get-NetAdapter`.

```powershell
New-NetFirewallRule -DisplayName "WSL - porta 8080" -Direction Inbound -InterfaceAlias "vEthernet (WSL)" -LocalPort 8080 -Protocol TCP -Action Allow
```

La regola è limitata all'adattatore WSL e non apre la porta alla rete esterna.

**c) Solo in modalità NAT, IP diretto dell'host.** Se non si vuole usare `mirrored`, l'indirizzo dell'host Windows visto da WSL si ricava così:

```bash
ip route show default
```

Esempio di output: `default via 172.x.y.1 dev eth0`. Usare quell'IP al posto di `localhost`:

```bash
curl http://172.x.y.1:8080/api/employees
```

Va usato l'IP restituito dal proprio sistema, non un valore copiato da questa guida.