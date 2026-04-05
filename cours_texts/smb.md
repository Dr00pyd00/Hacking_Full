

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


# smb.conf

Il y a deux blocks: 
- Global 
- Groupe de blocks pour chaque dossier/ objet partager




## Structure générale de `smb.conf`

Le fichier est divisé en **deux grandes parties** :

```
[global]        ← comportement général du serveur Samba
[nom_du_share]  ← définition de chaque dossier partagé
```

C'est exactement comme une config FastAPI : les settings globaux d'un côté, les routes/ressources de l'autre.

---

## Le bloc `[global]` — ce qui compte vraiment pour toi

Je vais ignorer tout ce qui est commenté (`;` ou `#`), parce que c'est juste de la doc inactive. Ce qui est **actif** dans ton `[global]` :

**Identification**
```ini
workgroup = WORKGROUP
server string = %h server (Samba, Ubuntu)
```
`workgroup` = le nom du groupe réseau Windows. `WORKGROUP` est la valeur par défaut, tu n'as pas besoin d'y toucher. `%h` est une variable Samba qui sera remplacée automatiquement par le hostname de ta machine.

**Logs**
```ini
log file = /var/log/samba/log.%m
max log size = 1000
logging = file
```
Un fichier de log par machine qui se connecte. `%m` = nom de la machine cliente. `1000` = taille max en kilo-octets (KiB).

**Authentification — la partie importante**
```ini
server role = standalone server
map to guest = bad user
usershare allow guests = yes
```

- `standalone server` : ton Samba fonctionne seul, pas dans un domaine Active Directory. C'est ce que tu veux pour un lab.
- `map to guest = bad user` : **si quelqu'un se connecte avec un utilisateur qui n'existe pas**, Samba le traite comme un invité anonyme. C'est ce qui va permettre à Metasploitable de se connecter sans mot de passe.
- `usershare allow guests = yes` : les shares créés avec `net usershare` peuvent être publics.

Les lignes `unix password sync`, `passwd program`, `pam password change` concernent la synchronisation des mots de passe Unix/Samba — tu peux les ignorer pour l'instant, elles ne bloquent rien.

---

## Ce que tu dois ajouter

Pour ton share `truc_a_partager`, tu ajoutes un bloc **en dessous** du `[global]`, après les blocs `[printers]` et `[print$]` :

```ini
[truc_a_partager]
   comment = Mon partage de test
   path = /home/kottak/truc_a_partager
   browseable = yes
   read only = no
   guest ok = yes
   create mask = 0755
   directory mask = 0755
```

**Explication ligne par ligne :**

| Ligne | Ce que ça fait |
|---|---|
| `[truc_a_partager]` | Le nom du share — c'est ce qui apparaît sur le réseau |
| `comment` | Description lisible par les clients |
| `path` | Le chemin **absolu** sur ton système vers le dossier à partager |
| `browseable = yes` | Le share est visible dans la liste des partages |
| `read only = no` | On peut écrire dedans (pas juste lire) |
| `guest ok = yes` | Pas besoin de mot de passe pour se connecter |
| `create mask = 0755` | Permissions des **fichiers** créés via Samba |
| `directory mask = 0755` | Permissions des **dossiers** créés via Samba |

---

## Avant de redémarrer Samba

Deux étapes importantes :

**1. Vérifier la syntaxe** — la commande magique que le fichier lui-même te conseille :
```bash
testparm
```
`testparm` = **test parameters** — il lit ton `smb.conf` et te dit s'il y a des erreurs. Aucune sortie d'erreur = tu es bon.

**2. S'assurer que le dossier existe vraiment** sur le système avec les bonnes permissions Unix :
```bash
ls -la /home/kottak/truc_a_partager
```

Si le dossier n'existe pas ou appartient à `root`, Samba ne pourra pas y accéder même avec une config parfaite — c'était probablement le problème la dernière fois.


