# Lors de l'envoi d'un paquet internet

## 1. Génération des données (User Space)
Une application (navigateur, jeu, client SSH) veut envoyer des données. Elle appelle une fonction système comme `send()` ou `write()` sur une socket.

- **Données brutes** : "Hello, World!" (par exemple).
- À ce stade, les données sont dans la mémoire **RAM** allouée au processus utilisateur.

## 2. Passage en noyau (Kernel Space) – première copie CPU + RAM
Le CPU exécute un **appel système** (mode utilisateur → mode noyau). Le noyau doit sécuriser les données.

- **Rôle du CPU** : Interruptions, changement de contexte (sauvegarde des registres).
- **Rôle de la RAM** : Le noyau **copie les données** depuis la mémoire du processus utilisateur vers un **buffer du noyau** (zone mémoire réservée, souvent une structure `sk_buff` sous Linux).
  - *Pourquoi cette copie ?* Pour que l'utilisateur ne puisse pas modifier les données pendant que le réseau les traite.

## 3. Encapsulation protocolaire (TCP/UDP, IP, Ethernet) – CPU intense
Le CPU construit les en-têtes des couches successives dans la RAM, **sans copier les données** (technique dite *zero-copy partielle*).

- **Couche Transport (TCP)** : Le CPU calcule les numéros de séquence, la somme de contrôle (checksum TCP). Ajoute l’en-tête TCP.
- **Couche Réseau (IP)** : Le CPU ajoute l’en-tête IP (adresses source/destination, TTL). Re-calcul de la somme de contrôle IP.
- **Couche Liaison (Ethernet)** : Le CPU ajoute l’en-tête Ethernet (MAC source/destination).

Pendant cette phase, le CPU lit et écrit fréquemment dans la **RAM cache** (L1/L2/L3) pour des raisons de performances. Si les données ne tiennent pas dans le cache, des **cache misses** ralentissent l’opération.

## 4. Gestion TCP : segmentation et buffer d’envoi (RAM)
Si le message est gros ou si TCP optimise (Nagle, etc.) :

- Les données restent dans un **buffer d’envoi** (RAM du noyau) jusqu’à recevoir un acquittement (ACK).
- Le CPU peut découper les données en **segments TCP** (MSS = Maximum Segment Size).
- Chaque segment pointe vers une partie du buffer sans le dupliquer (scatter-gather I/O).

## 5. File d’attente des paquets (Queueing) – RAM
Les paquets prêts sont placés dans une **file d’attente d’interface** (RAM liée au périphérique réseau). Si la file est pleine, le paquet est jeté (congestion locale).

## 6. Transmission DMA (Direct Memory Access) – RAM seulement, CPU libéré
**C’est l’étape la plus importante pour la RAM.**

- Le CPU prépare un descripteur (pointeur mémoire) dans la RAM que la carte réseau comprend.
- La carte réseau (NIC) utilise **DMA** pour lire directement le paquet depuis la RAM du noyau **sans intervention du CPU**.
- Le nœud CPU est libre pour d’autres tâches. Le CPU est juste informé par interruption quand la transmission est terminée (ou en mode polling pour les cartes haut débit).

## Récapitulatif : rôle précis de chaque composant

| Composant | Rôle principal avant envoi |
|-----------|----------------------------|
| **CPU** | – Copier données user → kernel<br>- Calculer checksums (TCP, IP)<br>- Construire les en-têtes<br>- Gérer les files d’attente et les appels système<br>- Programmer la carte réseau (via le driver) |
| **RAM** | – Stocker le buffer utilisateur original<br>- Stocker le buffer noyau (`sk_buff`)<br>- Stocker les en-têtes ajoutés<br>- Stocker les descripteurs DMA pour la carte réseau<br>- Faire tampon entre l’application et la carte réseau |

## Optimisations modernes
Pour éviter la multiplication des copies (chère en CPU et en occupation de bus mémoire), les OS récents utilisent :
- **Zero-copy** (`sendfile()`, `MSG_ZEROCOPY`) : la carte réseau lit directement les données depuis le buffer utilisateur (ou depuis le cache disque).
- **Checksum offloading** : la carte réseau calcule elle-même les checksums TCP/IP.
- **TSO (TCP Segmentation Offload)** : le CPU envoie un gros segment à la carte, c’est elle qui le découpe en MTU.

## Réponse simplifiée (synthèse)
> Le CPU copie les données de l’application vers un buffer en RAM du noyau, ajoute les en-têtes TCP/IP/Ethernet (en calculant les sommes de contrôle), puis place le paquet dans une file d’attente. Ensuite, il programme la carte réseau pour qu’elle lise le paquet directement en RAM via DMA, sans que le CPU ne touche plus aux données.

Si tu veux, je peux aussi expliquer le chemin **inverse** (paquet reçu → application).