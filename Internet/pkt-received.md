# Réception d'un paquet internet par la machine 

## 1. Arrivée sur la carte réseau (NIC) – hardware seul
Le signal électrique/optique arrive sur la carte réseau.

- **Rôle de la carte** : décode le signal, vérifie l'intégrité physique (CRC Ethernet en hardware), extrait la trame Ethernet.
- **Pas encore de CPU ni de RAM système** : la carte a sa propre petite mémoire tampon (FIFO).

## 2. DMA (Direct Memory Access) – écriture directe en RAM
C'est l'étape la plus importante pour la RAM.

- La carte réseau utilise **DMA** pour écrire le paquet (trame Ethernet complète) directement dans un **buffer de la RAM système** préalloué par le driver.
- **Le CPU ne fait rien pendant ce transfert** – il peut travailler sur d'autres tâches.
- La RAM reçoit donc les données brutes : en-tête Ethernet + en-tête IP + en-tête TCP + payload.

## 3. Interruption matérielle (IRQ) – réveil du CPU
La carte réseau lève une **interruption** (ou utilise MSI-X) pour signaler au CPU qu'un ou plusieurs paquets sont arrivés en RAM.

- **Rôle du CPU** : suspend son travail en cours, sauvegarde son contexte, exécute le **gestionnaire d'interruption** (ISR) du driver.
- Objectif : l'ISR est très courte (ne fait que marquer que du travail est prêt). Le vrai traitement est délégué à une **tâche de fond** (softirq sous Linux, DPC sous Windows).

## 4. Softirq (bas de la pile réseau) – CPU, RAM, cache
Le CPU sort rapidement de l'ISR et un mécanisme (softirq `NET_RX`) est planifié. C'est **le cœur du traitement**.

Pour **chaque paquet** dans la RAM :

### a) Extraction de la trame Ethernet
- Le CPU lit depuis la RAM l'en-tête Ethernet.
- Vérifie que la MAC destination correspond à l'interface (ou broadcast/multicast).
- Si OK, retire l'en-tête Ethernet (simple décalage de pointeur dans le buffer, pas de copie).

### b) Traitement IP (couche réseau)
- Le CPU lit l'en-tête IP dans la RAM : vérifie la somme de contrôle IP (recalculée par le CPU, sauf si *checksum offloading* activé).
- Vérifie la fragmentation IP : si plusieurs fragments arrivent, le CPU les **réassemble** dans un nouveau buffer RAM (copie nécessaire !). Cela consomme beaucoup de CPU et de RAM.
- Vérifie la destination IP (est-ce pour cette machine ?).

### c) Traitement TCP (couche transport)
- Le CPU lit l'en-tête TCP dans la RAM.
- **Calcule la somme de contrôle TCP** (sur l'en-tête + données). C'est une opération lourde pour le CPU – heureusement souvent déléguée à la carte réseau (*checksum offloading*).
- Extrait le numéro de port pour savoir quelle application destinataire.

## 5. Réassemblage TCP et buffer de réception (RAM)
TCP fournit un flux continu, pas des paquets isolés.

- Le CPU place les données utiles (payload) dans un **buffer de réception circulaire** en RAM, associé à la socket.
- Si des paquets arrivent dans le désordre (cause courante), le CPU les garde en RAM dans une file d'attente de réordonnancement jusqu'à combler le trou.
- Quand une séquence complète est disponible, le noyau signale (via `select`, `epoll`, ou un signal) que l'application peut lire.

## 6. Copie vers l'espace utilisateur – dernière étape RAM intensive
L'application appelle `recv()`, `read()` sur la socket.

- **Le CPU copie les données** depuis le buffer du noyau (RAM) vers la mémoire de l'application (RAM aussi).
- *Pourquoi une copie ?* Pour des raisons de sécurité : l'utilisateur ne peut pas corrompre les structures internes du noyau.
- Après cette copie, le buffer noyau est libéré pour recevoir de nouveaux paquets.

## Récapitulatif : rôle précis par composant (réception)

| Composant | Rôle principal |
|-----------|----------------|
| **RAM** | – Stocker le DMA buffer (paquet brut)<br>- Stocker les fragments IP en cours de réassemblage<br>- Stocker les segments TCP en attente de réordonnancement<br>- Stocker les buffers de réception des sockets<br>- Stocker les données finales copiées vers l'application |
| **CPU** | – Vérifier les sommes de contrôle (IP, TCP)<br>- Gérer l'interruption et la softirq<br>- Réassembler les fragments IP (copies en RAM)<br>- Réordonner les segments TCP<br>- Copier données noyau → utilisateur (dernière copie)<br>- Réveiller l'application (changement de contexte) |

## Problème fondamental : les copies multiples

Chaque copie entre buffers RAM consomme :
- **Du temps CPU** (pour déplacer les octets)
- **De la bande passante mémoire** (le bus RAM)
- **Du cache CPU** (pollution)

La réception d'un paquet typique implique **au moins 2 copies** :
1. NIC → buffer noyau (via DMA – sans CPU)
2. buffer noyau → buffer utilisateur (par le CPU)

## Optimisations modernes pour la réception

## GRO (Generic Receive Offloading)
Le CPU agrège plusieurs petits paquets similaires en un seul "super-paquet" avant de remonter la pile. Moins d'interruptions.

## RPS (Receive Packet Steering) / RSS (Receive Side Scaling)
- **RSS** (matériel) : la carte réseau écrit les paquets en RAM dans différentes files selon un hash des en-têtes. Plusieurs CPU peuvent traiter des flux en parallèle.
- **RPS** (logiciel) : si la carte ne supporte pas RSS, le CPU peut répartir le traitement.

## Zero-copy reception (sockets `PACKET_MMAP`, DPDK, AF_XDP)
- L'application **pré-alloue** des buffers en RAM utilisateur.
- La carte réseau écrit **directement** via DMA dans ces buffers.
- Le noyau ne fait presque rien – l'accès est ultra-rapide mais moins sécurisé.

## Checksum & TSO offloading (déjà vus)
La carte réseau calcule elle-même les sommes de contrôle TCP/IP pendant la réception. Le CPU est libéré.

## Schéma simplifié du flux mémoire

```
Câble → NIC → [DMA] → RAM (buffer noyau brut) → CPU (softirq : vérif, réassemble) 
→ RAM (buffer socket) → CPU (copie) → RAM (buffer application) → App
```

## Question complémentaire fréquente

**Pourquoi ne pas éviter la copie finale noyau → utilisateur ?**

Parce que le noyau ne peut pas donner à l'utilisateur un pointeur direct vers ses propres buffers internes :
- L'utilisateur pourrait lire les données d'autres processus (sécurité)
- Les buffers noyau sont réutilisés immédiatement pour de nouveaux paquets
- L'application peut être mise en sommeil – le noyau a besoin de garder le contrôle

C'est pour cela que les solutions *zero-copy* (comme DPDK) contournent complètement le noyau : l'application devient responsable de la gestion de sa propre mémoire et de la sécurité.

---

**Résumé ultra-court** :  
> À la réception, la carte réseau écrit le paquet directement en RAM via DMA. Le CPU est interrompu, lit et vérifie les en-têtes (sommes de contrôle), réassemble fragments et segments dans la RAM, puis copie le résultat final vers la mémoire de l'application. Chaque étape lit ou écrit dans la RAM, avec le CPU comme maître d'œuvre.