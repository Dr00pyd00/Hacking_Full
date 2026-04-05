

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

### Mettre en partage un dossier via samba

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


> par convention on met dans `/etc/samba/mon_dossier_partage`  
Il faut penser a chmod pour laisser les droits!


--- 

Les essentielles :

**Reconnaissance**
```bash
smbclient -L //IP -N          # lister les partages
enum4linux IP                  # énumération complète
```

**Connexion**
```bash
smbclient //IP/partage -N      # connexion anonyme
smbclient //IP/partage -U user # connexion avec user (demande mdp)
smbclient //IP/partage -U user%password  # connexion avec mdp direct
```

**Dans le shell SMB**
```
ls                  # lister les fichiers
cd dossier          # naviguer
get fichier         # télécharger un fichier
put fichier         # envoyer un fichier
more fichier        # lire un fichier
mkdir dossier       # créer un dossier
del fichier         # supprimer
lcd                 # voir où on est sur notre machine locale
help                # liste toutes les commandes
exit                # quitter
```

