# IP

L'IP (Internet Protocol) fait partie de la couche 3 du modèle OSI, appelée couche réseau (ou Network Layer).

**L'adresse IP que vous voyez (192.168.1.20) est une représentation décimale. En réalité, le routeur ou votre PC manipule sa forme binaire : 192.168.1.20 → 11000000 10101000 00000001 00010100 (32 bits).**

```
Couche            Nom                           Rôle                              Exemples
-------------------------------------------------------------------------------------------------------
7	            Application	            Données utilisateur                 HTTP, FTP, DNS, SMTP
6	            Présentation            Formatage, chiffrement              SSL/TLS, JPEG, ASCII
5	            Session	                Gestion des dialogues               NetBIOS, RPC
4	            Transport               Fiabilité, ports, segmentation      TCP, UDP
3	            Réseau                  Adressage IP, routage               IP, ICMP, ARP, OSPF
2	            Liaison                 Adresses MAC, trames                Ethernet, Wi-Fi, PPP
1	            Physique                Câbles, signaux, bits               RJ45, fibre optique, radio
```

Un routeur travaille à la couche 3 (il lit les IP).
Un switch travaille à la couche 2 (il lit les MAC).

Jusqu'à la couche 3, il n'y a pas de port utilisé.
C'est à partir de la couche 4. TCP-UDP

ARP = couche 2 et 3.

ICMP = couche 3

---

## IPv4 (ex: 192.168.1.10)

Elle est composée de 4 nombres (entre 0 et 255) séparés par des points.

```
192 . 168 . 1 . 10
│      │     │    └── Dernier octet (hôte)
│      │     └─────── 3ème octet
│      └───────────── 2ème octet
└──────────────────── 1er octet (réseau)
```

Chaque nombre = 1 octet = 8 bits.

Soit au total 32 bits.

La première partie identifie le réseau, la seconde partie identifie l'hôte (votre machine dans ce réseau). La coupure est définie par le masque de sous-réseau (ex: /24 = les 3 premiers nombres pour le réseau).


- En-tête IP => 20 à 60 octets => Contient les instructions de routage (adresses, options, etc.)

- Données => Variable (jusqu'à 65 535 octets) => Contient le message transporté (ex: un segment TCP ou une requête ICMP)

## IPv6 (ex: 2001:db8:85a3::8a2e:370:7334)

Elle est composée de 8 groupes de chiffres hexadécimaux (0-9, a-f), séparés par des deux-points (:). Chaque groupe fait 16 bits, soit 128 bits au total.

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
└─┬─┘ └─┬┘ │    └────┬────┘ └──┬──┘
 préfixe │  sous-réseau   identifiant d'interface
       │  (on peut supprimer les zéros inutiles)
   souvent donné par le FAI
```

Les :: signifient "un ou plusieurs groupes de zéros" (raccourci).

**Règles et protocoles: Défini par la RFC 791 (IPv4) et RFC 8200 (IPv6)**

---

## 🔍 Détail de l'en-tête IP (20 octets minimum)

```
Champ                   Taille          Exemple                     À quoi ça sert
--------------------------------------------------------------------------------------------------------------------
Adresse IP source       32 bits         192.168.1.20                D'où vient le paquet
Adresse IP destination  32 bits         8.8.8.8                     Où va le paquet
Version	4 bits          4 (IPv4) ou 6 (IPv6)                        Quel protocole IP utiliser
Longueur de l'en-tête   4 bits          5 (5×4=20 octets)	        Taille de l'en-tête (options comprises)
Longueur totale         16 bits         1500                        Paquet entier (en-tête + données)
TTL (Time To Live)      8 bits          64                          Nombre max de sauts (évite les boucles)
Protocole encapsulé     8 bits          6=TCP / 17=UDP / 1=ICMP     Type de données transportées
Checksum                16 bits         0x8a3f                      Vérifier que l'en-tête n'est pas corrompu
```

---

## Ce qui rend l'IP fonctionnelle

- Comment formater l'en-tête (vu ci-dessus)

- Comment router (trouver le chemin)

- Comment fragmenter (couper en petits paquets si besoin)

- Comment gérer les erreurs (avec ICMP)

---

## Voir les interfaces réseau avec les CMD

`ip a`

ou

`hostname -I`

Voir son ip en détail

`ip a show eth0`

Voir l'ip publique

`wget -qO- icanhazip.com`

ou

`curl -s icanhazip.com`

---