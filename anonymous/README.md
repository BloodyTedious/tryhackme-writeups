# Anonymous — TryHackMe Writeup

**Target IP:** `10.130.136.84`  
**Difficoltà:** Medium  
**Formato disponibile:** [Anonymous-TryHackMe-Writeup.pdf](Anonymous-TryHackMe-Writeup.pdf)

## Panoramica della Room
Writeup dettagliato della stanza **Anonymous**, focalizzato sull'abuso di un servizio FTP con login anonimo e directory scrivibili, sfruttato per alterare uno script di pulizia automatica (cron job) ed ottenere una reverse shell. L'escalation dei privilegi a root è stata completata sfruttando un binario SUID non standard (`/usr/bin/env`).

## Competenze Chiave
* Enumerazione FTP anonimo e SMB
* Abuso di script periodici (Cron Job / Cron Abuse)
* Privilege Escalation tramite binario SUID (`/usr/bin/env` via GTFOBins)
