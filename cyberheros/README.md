# CyberHeros — TryHackMe Writeup

**Target IP:** `10.128.130.106`  
**Difficoltà:** Easy  
**Full Report:** [CyberHeros-Writeup.pdf](CyberHeros-Writeup.pdf)

## Panoramica della Room
Writeup della stanza **CyberHeros**, incentrata su una vulnerabilità di logica applicativa lato client. Analizzando il codice sorgente JavaScript della pagina di login, è stato possibile individuare credenziali hardcoded e una funzione di offuscamento basata sull'inversione di stringa (`reverse string`), permettendo di bypassare l'autenticazione.

## Competenze Chiave
* Analisi del codice JavaScript lato client
* Deoffuscamento e logica di autenticazione debole
* Web Exploitation / Client-Side Logic Flaw
