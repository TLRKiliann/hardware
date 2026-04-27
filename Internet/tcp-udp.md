# TCP - UDP

Le segment TCP est une unité de données de la `couche transport`, encapsulée à l'intérieur d'un `paquet IP`.
Un segment TCP voyage à l'intérieur d'un paquet IP, lui-même à l'intérieur d'une trame Ethernet. C'est l'emboîtement des couches réseau.

## Différence entre paquet IP et segment TCP

```
Paquet IP                   Segment TCP
-------------------------------------------------------------
Couche réseau               Couche transport
Gère les adresses IP        Gère les ports et la fiabilité
Sans connexion              Avec connexion fiable
Ne garantit pas l'ordre     Garantit l'ordre et la livraison
Contient le segment TCP     Contient les données applicatives
```

TCP nécessite une communication bidirectionnelle entre deux points.

**3-way handshake**

Entre client-server / client-client / server-server / client-imprimante (voir la liste plus bas)

1. Client → Serveur : SYN (Seq=100, ACK=0, Flags=SYN)
2. Serveur → Client : SYN-ACK (Seq=500, ACK=101, Flags=SYN+ACK)
3. Client → Serveur : ACK (Seq=101, ACK=501, Flags=ACK)

## Structure d'un segment TCP

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                         DATA (payload)                        |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Les principaux champs

```
Champ                   Taille              Rôle
-------------------------------------------------------------------------------------------------
Source Port             16 bits             Port de l'application émettrice (ex: 54321)
Destination Port        16 bits             Port de l'application destinataire (ex: 80 pour HTTP, 443 pour HTTPS)
Sequence Number         32 bits             Numéro du premier octet de données (ordonnancement, réassemblage)
Acknowledgment Number   32 bits             Prochain octet attendu (accusé de réception)
Data Offset             4 bits              Taille de l'en-tête (en mots de 32 bits)
Flags (9 bits)          9 bits              Contrôle de la connexion : SYN, ACK, FIN, RST, PSH, URG
Window                  16 bits             Nombre d'octets que l'émetteur accepte de recevoir (contrôle de flux)
Checksum                16 bits             Vérification d'intégrité (en-tête + données)
Urgent Pointer          16 bits             Offset vers les données urgentes (si flag URG actif)
Options                 variable            Timestamps, échelle de fenêtre, SACK, etc.
Data (payload)          variable            Le message réel (ex: requête HTTP, fichier)
```

**6 Flags**

```
Flag                Nom                 Rôle
---------------------------------------------------------------------------------------------------
SYN                 Synchronize         Établissement de connexion (démarrage)
ACK                 Acknowledgment      Accusé de réception (toujours présent après le 1er échange)
FIN                 Finish              Fin normale de la connexion
RST                 Reset               Annulation brutale de la connexion
PSH                 Push                Envoyer immédiatement les données (sans attendre)
URG                 Urgent              Données prioritaires à traiter vite
```

## Liste des connexions TCP

1. Machine ↔ Serveur (cas le plus courant)
- Ton navigateur (port aléatoire) ↔ Serveur web (port 80 ou 443)
- Ton client email ↔ Serveur mail (port 25, 587, 993)

2. Machine ↔ Machine (pair à pair)
- Téléchargement BitTorrent : deux ordinateurs ordinaires connectés directement
- Jeux vidéo en ligne : ta console ↔ celle d'un autre joueur (pas de serveur central)
- Appels VoIP (certains protocoles comme SIP avec RTP)

3. Serveur ↔ Serveur
- Deux serveurs web qui échangent des données (ex: réplication de base de données)
- Serveur ↔ Serveur de sauvegarde
- Load balancer ↔ Serveur applicatif

4. Machine ↔ Périphérique réseau intelligent
- Imprimante réseau (tu te connectes à son port 9100 pour imprimer)
- Routeur administré via SSH (port 22)
- Caméra IP (streaming vidéo)
- Capteur industriel (modbus TCP)

5. Sur la même machine (localhost)
- Connexion TCP entre deux processus sur le même ordinateur (adresse 127.0.0.1)
- Exemple : un serveur web local (Apache) avec ta base de données (MySQL)

6. Via des tunnels ou proxies
- VPN : segment TCP encapsulé dans un autre segment TCP
- Proxy HTTP : connexions TCP enchaînées

---

## Le voyage d'un segment TCP (de votre machine à l'autre)

```
VOTRE ORDINATEUR
┌─────────────────────────────────────────┐
│  Application (Navigateur, curl, etc.)   │
└─────────────────┬───────────────────────┘
                  │ Données "Bonjour"
                  ▼
┌─────────────────────────────────────────┐
│  SEGMENT TCP (couche transport)         │
│  Port source: 54321                     │
│  Port dest: 80                          │
│  Seq: 100, ACK: 500, Flags: PSH+ACK     │
│  DATA: "Bonjour"                        │
└─────────────────┬───────────────────────┘
                  │ Encapsulation
                  ▼
┌─────────────────────────────────────────┐
│  PAQUET IP (couche réseau)              │
│  IP source: 192.168.1.10                │
│  IP dest: 8.8.8.8                       │
│  Protocol: 6 (TCP)                      │
│  Contient: [Segment TCP]                │
└─────────────────┬───────────────────────┘
                  │ Encapsulation
                  ▼
┌─────────────────────────────────────────┐
│  TRAME ETHERNET (couche liaison)        │
│  MAC source: AA:BB:CC:DD:EE:FF          │
│  MAC dest: routeur local XX:XX:XX:XX    │
│  Type: 0x0800 (IPv4)                    │
│  Contient: [Paquet IP]                  │
└─────────────────┬───────────────────────┘
                  │ Sur le câble / WiFi
                  ▼
            [SIGNAL PHYSIQUE]
         (impulsions électriques / ondes radio)
```

**Voir les buffers TCP en attente**

1. Dans la mémoire RAM (avant départ)

`sudo ss -m -t -a`

2. Dans la file de transmission de la carte réseau

`sudo ethtool -S eth0 | grep tx`

C'est là que le CPU a déposé les segments pour que la carte réseau les envoie.

## Voir les segments dans les buffers noyau (avant envoi)**

**Affiche les sockets TCP et leur état des buffers**

`sudo ss -t -a -e -i -p`

**Pour voir les queues d'envoi**

`sudo ss -t -a -m | grep -E "Send|Recv"`

**Surveillance en direct des buffers**

`watch -n 1 'ss -t -a -m | head -20'`

## Voir les segments qui sortent (via tcpdump)**

**Capture en direct - vous les verrez passer !**

`sudo tcpdump -i eth0 tcp -n -v`

```
# Sortie typique :
# 14:32:10.123456 IP 192.168.1.10.54321 > 8.8.8.8.80: Flags [P.], seq 100:107, ack 500, win 65535, length 7
#                                                                    ↑
// Votre segment TCP avec "Bonjour" (7 octets)
```

## Voir les segments dans la pile noyau

**Statistiques TCP en temps réel**

`watch -n 1 'cat /proc/net/netstat | grep -A1 TcpExt'`

**Voir les segments envoyés/reçus**

`cat /proc/net/snmp | grep Tcp`

## Le chemin complet illustré (avec vos commandes)**

**Terminal 1 - Capturez le trajet**

`sudo tcpdump -i eth0 tcp -n -v -Xvv`

**Terminal 2 - Envoyez un segment**

`echo "Bonjour TCP" | nc 8.8.8.8 80`

```
La circulation routière

    Segment = une voiture spécifique
    CPU = le conducteur
    RAM = le parking avant départ
    Carte réseau = la sortie d'autoroute
    Câble = l'autoroute
```

**Étape 1 - Créez un segment**

Dans terminal 1 : capture

`sudo tcpdump -i lo tcp port 12345 -v -X`

loopback (lo) pour rester sur la même machine


**Étape 2 - Envoyez**

Dans terminal 2

`nc -l 12345 &`  # écoute

`echo "TEST" | nc 127.0.0.1 12345`  # envoi


**Étape 3 - Observez le chemin**

Pendant l'envoi, dans un 3ème terminal

`cat /proc/net/tcp`  # montre les sockets actives

`ss -t -a -p`        # montre avec les processus


1. [RAM] Votre application écrit dans un socket
2. [CPU] Le noyau crée le segment TCP
3. [RAM] Segment dans le buffer d'envoi
4. [CPU] Le driver le prend
5. [CARTE RÉSEAU] File de transmission (TX ring buffer)
6. [PHYSIQUE] Sur le câble/ondes
7. [ROUTEUR] Rebondit de routeur en routeur (toujours sous forme IP)
8. [SERVER] Arrive dans la carte réseau du destinataire
9. [RAM] Buffer de réception
10. [CPU] Le noyau le traite
11. [APPLICATION] Le programme le lit

---

## Avec application TOP

**Dans la ligne %Cpu(s), regardez**

- sy : temps CPU passé dans le noyau (system) - c'est la pile TCP qui tourne
- si : temps CPU passé dans les interruptions soft (soft IRQ) - c'est le traitement des paquets réseau
- us : temps utilisateur (votre application qui lit les données)

```
- Dans top, appuyez sur '1' pour voir chaque cœur CPU individuellement
- Appuyez sur 't' pour voir les résumés des tâches
- Appuyez sur 'H' pour voir les threads (incluant ksoftirqd/N qui gère le réseau)
```

**Meilleure commande pour voir le réseau**

`top -d 1 -n 5 -p 0`

-d 1 : rafraîchit chaque seconde

-n 5 : fait 5 itérations puis s'arrête

-p 0 : montre tous les processus

---

## TCPdump

**Voir tout le traffic**

`sudo tcpdump -i eth0 tcp`

**Voir les flags TCP (SYN, ACK, FIN, etc.)**

`sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-ack|tcp-fin|tcp-rst) != 0'`

**Voir les segments en détail (en-têtes)**

`sudo tcpdump -i eth0 tcp -v`

**Voir le contenu des segments (payload)**

`sudo tcpdump -i eth0 tcp -X`

**Pour un port spécifique (ex: HTTP)**

`sudo tcpdump -i eth0 tcp port 80`

****

**Observer les SYN (nouvelles connexions)**

`sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0' -v`

Interprétation : chaque SYN déclenche la création d'une structure de socket dans le noyau (utilisation CPU)

**Détecter si le CPU est saturé (pertes)** (sur 30 secondes)

`sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0 or (tcp[tcpflags] & tcp-ack != 0 and tcp[13] & 0x80 != 0)' -c 100`

**Observer les ACK (accusés)**

`sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-ack != 0' -v 2>&1 | head -20`

Interprétation : chaque ACK vu est un segment traité par le CPU (vérification, mise à jour de séquence, etc.)

**Surveiller port spécifique**

`sudo tcpdump -i eth0 tcp port 443`