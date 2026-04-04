
# Netcat

Outils reseau CLI = `nc`  sert a:
- se connecter a un port
- ecouter un port 
- envoyer/recevoir des données 
- ouvrir des shells (reverse/bind)
- tester des services reseaux

```bash 
nc IP port

# exemple:
nc google.com 80

```

#### Example avec http google

```bash
nc www.google.com 80

# on appel l'ip de google sur SON port 80
```
Et dans la console on utilise le **Protocole HTTP**:
```bash
GET / HTTP/1.1
Host: www.google.com
<saut de ligne>

# Ici l'html de google !

```
---
Ecriture avec charactere specials:
```bash
"GET / HTTP/1.1\r\nHost:www.google.com\r\n\r\n"
```
Donc on Pipe pour mettre ca dans nc:
```bash
echo -e "GET / HTTP/1.1\r\nHost: www.google.com\r\n\r\n" | nc www.google.com 80
```
- `echo -e`: prendre en compte les char speciaux!


## Faire communiquer deux machine

Machine server:
```bash
nc -l -p 4444
```
- `-l`: listen
- `-p`: port sur lequel ecouter

---
Machine client:
```bash
nc <ip_machine_server> 4444
```

### envoyer un fichier entre 2 machines

On commence par le server qui doit accepter et mettre les datas quelque part:
```bash
nc -l -p 4444 > texte_recu.txt
```
ou avec echo:
```bash
echo "voila un message que j'envoie!" | nc -l -p 4444
```

---
Ensuite le client doit recevoir avec la commande:
```bash
nc <ip_du_server> 4444 < mon_doc.txt
```
=> transferer!



# SMB / Samba

- `SMB` **__Server_Message_Block__** = protocole de partage de fichiers en reseau
- `Samba` = application qui tourne pour recevoir les demande (avec le protocole)

### smbclient 

Outils pour faire les appels.

```bash
smbclient -L 192.168.78.129 -N

```
- `-L`: liste des partage disponible par la machine. Ce que le serveur samba
  expose.
-`-N`: filtre les elements qui ne demande pas de mdp.

### Config samba

```bash
/etc/samba

# smb.conf => configuration de ce que l'on veut exposer
```

### Mettre en partgae un dossier via samba

Il faut aller dans **/etc/samba/smb.conf** et ajouter un block.


```
[nom_du_partage]
   path = /chemin/vers/dossier
   browseable = yes
   guest ok = yes
   read only = no
```

- `[nom_du_partage]` — le nom qui apparaît dans `smbclient -L`
- `path` — le vrai chemin sur le disque
- `browseable` — visible dans la liste ou non
- `guest ok` — accessible sans mot de passe
- `read only` — lecture seule ou écriture autorisée

