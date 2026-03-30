# Scapy python

Scapy sert a creer des paquets **couche par couche**.

#### Syntaxe de base:

```python 
paquet = Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(pdst="192.168.1.1")
````
Le `/`emballe les couches.

--- 

### `Ether()`: couche 2 = Ethernet  

>Ethernet est le reseau LOCAL pour on utilise les MAC.
- `dst`: destination = la MAC a qui envoyer
- `"ff:ff:ff:ff:ff:ff"`: broadcast = envoie a tous

---

### `ARP()`: couche 2/3 = 
- `op` — **operation** — opération : le type de message ARP. Valeurs possibles : `who-has` (question) ou `is-at` (réponse)

- `hwsrc` — **hardware source** — adresse matérielle source : ta propre adresse MAC

- `psrc` — **protocol source** — adresse protocole source : ton propre adresse IP

---

tableau complet des champs ARP :

| Abréviation | Mot complet | Traduction |
|-------------|-------------|------------|
| `op` | operation | opération (question ou réponse) |
| `hwsrc` | hardware source | MAC source (la tienne) |
| `hwdst` | hardware destination | MAC destination (celle qu'on cherche) |
| `psrc` | protocol source | IP source (la tienne) |
| `pdst` | protocol destination | IP destination (celle qu'on cherche) |
| `hwtype` | hardware type | type matériel (Ethernet, WiFi...) |
| `ptype` | protocol type | type protocole (IPv4, IPv6...) |
| `hwlen` | hardware length | longueur de l'adresse MAC (6 octets) |
| `plen` | protocol length | longueur de l'adresse IP (4 octets) |


## Envoyer et recevoir

### Moyen 1:

```python
responses, _ = srp(paquet, timeout=0.2, verbose=False)

```

- `srp` — **send and received paquet** — envoyer ET recevoir paquet
- `timeout` — **timeout** — delai d'attente avant d'abandonner
- `verbose` — **details/ bavards** — False:  pas affichage automatique
- `_` = les paquets **sans** reponses ( on s'en fou)


Dans "reponses" ce sont des **Pairs**:
- `Sended` = le paquet que j'ai envoyer
- `Received` = le paquet recu en reponse

```python

for sended, received in responses:
    print(f"IP: {received.psrc} // MAC: {received.hwsrc}")

```


Bien sûr —

---

**Ce qu'on a fait avec Scapy**

Scapy c'est une bibliothèque Python qui te permet de **construire et envoyer des paquets réseau couche par couche** — là où `socket` te cachait tout ça.

---

**La syntaxe de base**

```python
paquet = Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(pdst="192.168.1.1")
```

Le `/` signifie "emballe dans" — tu empiles les couches comme dans le modèle OSI.

---

**Les deux couches qu'on utilise**

`Ether()` — couche 2, Ethernet
- `dst` — **destination** — destinataire : la MAC à qui envoyer physiquement
- `ff:ff:ff:ff:ff:ff` = broadcast = tout le monde sur le réseau local

`ARP()` — couche 2/3, protocole de résolution d'adresse
- `pdst` — **protocol destination** — IP destination : l'IP qu'on cherche
- Scapy remplit automatiquement `hwsrc` (**hardware source** — ta MAC) et `psrc` (**protocol source** — ton IP)

---

**Envoyer et recevoir**

```python
reponses, _ = srp(paquet, timeout=0.2, verbose=False)
```

- `srp` — **send receive packet** — envoyer/recevoir un paquet : envoie ET attend une réponse
- `timeout` — **timeout** — délai d'attente : secondes à attendre avant d'abandonner
- `verbose` — **verbose** — bavard : `False` = pas d'affichage automatique
- `_` = les paquets sans réponse — on s'en fout

---

**Lire la réponse**

```python
for sended, received in reponses:
    print(f"IP: {received.psrc} // MAC: {received.hwsrc}")
```

- `received.psrc` — **protocol source** — IP source : l'IP de la machine qui a répondu
- `received.hwsrc` — **hardware source** — MAC source : la MAC de la machine qui a répondu

---

