# RootMe — TryHackMe Writeup

**Target IP:** `10.128.177.192`  
**Difficoltà:** Easy  
**Full Report:** [RootMe-TryHackMe-Writeup.pdf](RootMe-TryHackMe-Writeup.pdf)

## Panoramica della Room
Writeup della stanza classica **RootMe**, incentrata sul bypass di un filtro di upload file basato su blacklist tramite l'utilizzo di estensioni alternative (`.phtml`) eseguite da Apache. L'escalation a root è stata ottenuta individuando e sfruttando un interprete Python configurato con permessi SUID.

## Competenze Chiave
* Web Enumeration (`gobuster`) & Upload Filter Bypass (`.phtml`)
* Reverse Shell (`netcat` / pentestmonkey)
* Privilege Escalation tramite binario SUID (`python2.7`)
