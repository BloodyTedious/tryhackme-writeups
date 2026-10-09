# Lian Yu (Arrowverse) — TryHackMe Writeup

**Target IP:** `10.130.169.156`  
**Difficoltà:** Medium / Hard  
**Full Report:** [LianYu-TryHackMe-Writeup.pdf](LianYu-TryHackMe-Writeup.pdf)

## Panoramica della Room
Writeup della stanza **Lian_Yu**, ispirata all'universo di Arrow. La compromissione richiede l'uso di tecniche miste: fuzzing di directory web, OSINT da commenti nascosti e CSS, decodifica di token (Base-58), manipolazione di header PNG corrotti tramite hex editing, estrazione di dati tramite steganografia (`steghide`) e infine privilege escalation abusando dei permessi `sudo` su `pkexec`.

## Competenze Chiave
* Web fuzzing avanzato e OSINT
* Hex Editing e correzione header file corrotti
* Steganografia (`steghide`)
* Privilege Escalation (`pkexec`)
