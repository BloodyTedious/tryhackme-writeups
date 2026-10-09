# Anonymous — TryHackMe Writeup

**Target IP:** `10.130.136.84`

## 1. Ricognizione
Scansione iniziale con Nmap per identificare porte e servizi attivi (FTP con login anonimo abilitato, SSH e SMB). Il banner FTP indicava "NamelessOne's FTP Server".

## 2. Enumerazione
* **FTP:** Accesso anonimo consentito con una directory `scripts/` scrivibile contenente tre file (`clean.sh`, `removed_files.log`, `to_do.txt`). Il timestamp recente del log confermava che `clean.sh` veniva eseguito periodicamente da un cron job.
* **SMB:** Condivisione `pics` accessibile in modo anonimo (contenente immagini, risultate poi dei red herring).

## 3. Sfruttamento (Exploitation)
Poiché la directory `scripts/` era scrivibile via FTP e lo script `clean.sh` veniva eseguito automaticamente dal sistema, è stato possibile sovrascrivere `clean.sh` inserendo una payload di reverse shell bash. Attendendo l'esecuzione del cron job, è stata ottenuta una shell come utente `namelessone`.

## 4. Escalation dei privilegi (Privilege Escalation)
* **Analisi SUID:** La ricerca dei binari con bit SUID ha evidenziato la presenza anomala di `/usr/bin/env` con permessi di root.
* **Bypass:** Sfruttando GTFOBins, l'esecuzione di `/usr/bin/env /bin/sh -p` ha permesso di ottenere direttamente una shell con privilegi di root.

## 5. Cosa ho imparato e dove mi sono bloccato
* **Lezione chiave:** I servizi FTP con login anonimo e directory scrivibili rappresentano un vettore critico quando i file vengono elaborati da processi automatizzati (cron). L'analisi dei log con timestamp recenti aiuta a dedurre l'esistenza di task periodici.
* **Difficoltà incontrate:** Inizialmente `sudo -l` restituiva un errore per mancanza di una TTY allocata, rischiando di far scartare quella via. È fondamentale verificare sempre con `id` o allocare una pty prima di procedere.
