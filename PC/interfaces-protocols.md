# Interfaces et Protocoles

- Interfaces = SATA, PCIe, Bus (le connecteur / le bus physique)
- Protocoles = IDE, AHCI, NVMe (le langage de communication)

**Firmware et Bootloader**

- Firmware = UEFI (BIOS)
- Bootloader = GRUB

- [HARDWARE PC](#hardware-pc)
  - [SATA](#sata)
    - [Logique SATA](#logique-sata)
  - [AHCI](#ahci)
  - [RAID](#raid)
  - [NVMe](#nvme)
  - [PCIe](#pcie)
  - [BUS](#bus)
  - [Chipset](#chipset)
  - [UEFI (BIOS)](#uefi-bios)
  - [GRUB](#grub)
  

## SATA

**SATA** => (Serial ATA)

- SATA (Serial ATA) est la norme de câble et de connecteur qui sert à brancher la plupart des disques durs (HDD) et SSD à la carte mère.

1. C'est lent (aujourd'hui)	Un SATA classique (version 3.0) tourne à 6 Gbit/s (environ 550 Mo/s réels). C'est très bien pour un disque dur mécanique, mais c'est le goulot d'étranglement pour un SSD récent.

2. Il est remplacé par le NVMe	Les nouveaux SSD rapides n'utilisent plus le SATA, mais le bus PCIe directement, via le format M.2. C'est 5 à 10 fois plus rapide.

SATA: câble qui relie SSD à la carte mère (NVMe relie direct au CPU).
Il existe aussi le SATA POWER.

```
Connecteur	                            Rôle                              Venant de
SATA Data (petit, en L)	                Les données (votre question)      La carte mère
SATA Power (plus large, en L aussi)	    L'électricité (5V/12V)            L'alimentation de l'ordinateur
```

Voir la liste des disques et leur type de connexion

`lsblk -d -o name,rota,size,type,tran`

Résultats

```
NAME    ROTA   SIZE   TYPE   TRAN
sda      1     256G   disk   sata   ← disque dur mécanique (ROTA=1)
sdb      0     512G   disk   sata   ← SSD SATA (ROTA=0)
nvme0n1  0     1T     disk   nvme   ← SSD rapide (pas du SATA)
```

### Logique SATA

Un disque dur utilise l'interface SATA, donc il se branche avec un câble SATA sur un port SATA de la carte mère.

[⬆️ up](#Hardware)

---

## AHCI

**AHCI** => (Advanced Host Controller Interface)

- AHCI: règles de circulation pour le SATA et NCQ (PROTCOL) HDD + SSD.

(IDE = l'ancêtre)

L'AHCI (Advanced Host Controller Interface) est un mode de fonctionnement pour les cartes mères qui permet au système d'exploitation (Windows, Linux, etc.) de communiquer efficacement et de profiter pleinement des fonctionnalités avancées des disques durs et SSD modernes.

Avantage

- NCQ (Native Command Queuing) rend le PC plus réactif lorsqu'il y a plusieurs tâches en même temps et réorganise les demandes de lecture/écriture. 

- Le "Hot Plug" : C'est la possibilité de brancher ou débrancher un disque dur SATA pendant que l'ordinateur est allumé

IDE est obsolète

AHCI et plus rapide que IDE

NVMe est plus rapide que AHCI

[⬆️ up](#Hardware)

---

## RAID

**RAID** (Redundant Array of Independent Disks)

- RAID: méthode pour faire travailler plusieurs disks ensemble. 

Sans RAID (le mode normal)
Vous disposez d'une seule camionnette qui effectue tous les trajets. C'est le fonctionnement standard.

Avec RAID 0 (la vitesse)
Vous utilisez deux camionnettes. Vous chargez 500 colis dans chacune. Les deux partent en même temps. Résultat : vous livrez deux fois plus vite. Mais c'est risqué : si une camionnette a un accident, vous perdez la moitié des colis (ceux qu'elle transportait).

Avec RAID 1 (la sécurité)
Vous utilisez deux camionnettes, mais vous chargez les 1000 colis complets dans chacune d'elles. Les deux partent en même temps. Si une camionnette tombe en panne, l'autre continue et vous ne perdez aucun colis. C'est sécurisé, mais c'est cher : vous utilisez deux camionnettes pour livrer seulement 1000 colis (au lieu de 2000).

Les RAID les plus courants

```
RAID 0
RAID 1
RAID 5
RAID 10
```

```
SANS RAID (2 disques séparés) :
┌────────┐     ┌────────┐
│ Disque A│     │ Disque B│
│ Fichier │     │ Fichier │
│ complet │     │ complet │
└────────┘     └────────┘
→ Le système voit deux lettres (D: et E:)

RAID 0 (découpage) :
┌────────┐     ┌────────┐
│ Disque A│     │ Disque B│
│ Moitié  │     │ Moitié  │
│ du      │     │ du      │
│ fichier │     │ fichier │
└────────┘     └────────┘
→ Le système voit une seule lettre (D:)

RAID 1 (miroir) :
┌────────┐     ┌────────┐
│ Disque A│     │ Disque B│
│ Copie   │ =  │ Copie   │
│ exacte  │     │ exacte  │
└────────┘     └────────┘
→ Le système voit une seule lettre (D:)
```

Les deux types de RAID

```
Type	                Où ça se passe ?	                                                  Avantage	                                                          Inconvénient
RAID matériel	        Sur une carte contrôleur dédiée (souvent dans les serveurs)	        Transparent pour le système, plus rapide	                          Cher, carte spécifique
RAID logiciel	        Géré par le système d'exploitation (Linux mdadm, Windows, macOS)	  Gratuit, flexible, fonctionne avec n'importe quels disques	        Utilise un peu de CPU
```

Voir les disques et leur organisation

`lsblk`

Si un RAID logiciel existe (mdadm)

`cat /proc/mdstat`

[⬆️ up](#Hardware)

---

## NVMe

**NVMe** => (Non-Volatile Memory Express) 128 - 256 - 512 - 1024 GB

NVMe (Non-Volatile Memory Express) est un protocole (un langage de communication) conçu spécialement pour que le CPU parle très vite aux SSD modernes branchés sur le bus PCIe.

- NVMe: Plus rapide que AHCI et fait pour SSD M.2 (connection du SSD M.2 sur Slot M.2 de la mother board)
- Slot M.2: emplacement direct sur la carte mère (remplace SATA).

un SSD NVMe, lui, n'utilise ni câble SATA, ni interface SATA. Il se branche directement dans un slot M.2 et parle un autre protocole (NVMe) via un autre bus (PCI Express).

- IDE: Ne jamais utiliser pour les SSD (vieille techno !)

```
Caractéristique	      AHCI (SATA)	      NVMe (PCIe)	              Gain
-------------------------------------------------------------------------------
Débit max             ~550 Mo/s         ~7 000 Mo/s (PCIe 4.0)    x12
Nombre de files       1 file (queue)    65 535 files              x65 535
Commandes par file    32 max            65 535 max                x2 048
Latence               ~6 µs             ~3 µs                     plus réactif
```

- Transfert d'un fichier de 50Go => 1m30 (NVMe) vs 10 minutes (SATA) → très visible

**Le format physique : M.2**

Un SSD NVMe ressemble à une barrette de chewing-gum qui se branche directement sur la carte mère (slot M.2). Pas de câbles.

```
Type de SSD	            Connectique	                Vitesse max
-----------------------------------------------------------------
SSD SATA	              Câble SATA + alimentation	  ~550 Mo/s
SSD NVMe (PCIe 3.0)	    Slot M.2	                  ~3 500 Mo/s
SSD NVMe (PCIe 4.0)	    Slot M.2	                  ~7 000 Mo/s
SSD NVMe (PCIe 5.0)	    Slot M.2	                  ~12 000 Mo/s
```

**Voir si un disque est NVMe**

`lspci | grep -i "non-volatile"`

**Liste des disques NVMe**

`lsblk | grep nvme`

**Infos détaillées (si disque NVMe)**

`sudo nvme list`

[⬆️ up](#Hardware)

---

## PCIe

**PCIe** => (Peripheral Component Interconnect Express)

- PCIe (Peripheral Component Interconnect Express) est un bus (un réseau de communication) qui relie tous les composants critiques directement au processeur.

* La carte graphique (le plus gros consommateur)
* Les SSD NVMe (les disques ultra-rapides)
* Les cartes d'extension (son, réseau, capture vidéo)
* Les ports USB, Ethernet, Wi-Fi (via des ponts)

```
Génération	    Débit par voie      Débit x16         Année
PCIe 1.0	      250 Mo/s	          4 Go/s           2004
PCIe 2.0	      500 Mo/s	          8 Go/s           2007
PCIe 3.0	      1 Go/s              16 Go/s          2010
PCIe 4.0	      2 Go/s              32 Go/s          2017
PCIe 5.0	      4 Go/s              64 Go/s          2021
PCIe 6.0	      8 Go/s              128 Go/s         2022
```

Le processeur parle directement à la carte graphique et aux SSD NVMe via PCIe

```
Technologie	    Route	                            Protocole	                Vitesse typique
SSD NVMe     (PCIe 5.0 x4) Autoroute 8 voies	      NVMe	                    10 000+ Mo/s
SSD NVMe     (PCIe 4.0 x4) Autoroute 4 voies	      NVMe	                    5 000-7 000 Mo/s
SSD NVMe     (PCIe 3.0 x4) Autoroute 2 voies	      NVMe	                    3 000-3 500 Mo/s
SSD SATA     Route nationale	                      AHCI	                    550 Mo/s
HDD SATA     Petite route	                          AHCI	                    150-250 Mo/s
IDE          Chemin de terre	                      PATA	                    133 Mo/s
```

[⬆️ up](#Hardware)

---

## BUS

Le bus de données (Data Bus) : C'est l'autoroute qui transporte les informations elles-mêmes (les données). Sa "largeur" (8, 16, 32, 64 bits) détermine combien de bits peuvent passer en même temps. Plus c'est large, plus c'est rapide.

Le bus d'adresses (Address Bus) : C'est le facteur qui transporte l'adresse de l'information. Il dit "Hé, va chercher ce qui se trouve à l'adresse X". Sa largeur détermine la quantité maximale de mémoire que le CPU peut adresser.

Le bus de contrôle (Control Bus) : Ce sont les feux de signalisation et les panneaux de direction. Il transporte les signaux qui indiquent quoi faire (Lire ? Écrire ? Attendre ?).

Sur Linux

La commande `lspci` (List PCI) : Le bus PCI (Peripheral Component Interconnect) est le bus moderne le plus important. C'est l'autoroute principale pour la plupart des composants internes (carte graphique, carte son, réseau SATA, USB...).

`lspci`

```
00:00.0 Host bridge
00:02.0 VGA compatible controller   <-- Votre carte graphique
00:1f.2 SATA controller             <-- Vos disques durs
```

USB:

`lsusb`

```
périphériques externes (clef USB, souris, clavier, imprimante)
```

CPU:

`lscpu`

```
Architecture :            x86_64   # <-- CECI est crucial
```

Détail du bus entre le CPU et la RAM (sa vitesse, sa largeur, son nombre de canaux:

`sudo lshw -class memory | grep -A 5 "description: System Memory"`

Carte graphique:

`lspci -v`

`lspci -v | grep -A 10 "VGA"`

Version plus lisible:

`sudo lshw -class display`

[⬆️ up](#Hardware)

---

## Chipset

```
Processeur (le patron)
    │
    ├── Direct (autoroute) → Carte graphique, RAM, SSD NVMe principal
    │
    └── Via le chipset (petite route) → USB, SATA, Ethernet, Audio, etc.
```

```
                    PROCESSEUR (CPU)
                           │
            ┌──────────────┼──────────────┐
            │              │              │
        PCIe x16        PCIe x4        	PCIe x4
            │              │              │
      Carte graphique  SSD NVMe #1   	SSD NVMe #2
            │              |
            └──────┬───────┘
                   │
            LIAISON RAPIDE (DMI / PCIe)
                   │
            ┌──────┴──────┐
            │   CHIPSET   │
            └──────┬──────┘
                   │
        ┌──────────┼──────────┬──────────┐
        │          │          │          │
    Ports SATA  Ports USB  Ethernet    	Audio
```

[⬆️ up](#Hardware)

---

## UEFI (BIOS)

**BIOS** => (Basic Input/Output System) = UEFI

**UEFI** => (Unified Extensible Firmware Interface)

- BIOS: est un micro-programme (un firmware) intégré dans une puce sur la carte mère, il lance l’OS.

Le BIOS : Une vieille technologie remplacée par l'UEFI

Ce qu'on appelle "BIOS" aujourd'hui est en réalité presque toujours de l'UEFI (Unified Extensible Firmware Interface). Mais tout le monde dit encore "BIOS" par habitude.

Au moment où vous appuyez sur le bouton "Allumer", le BIOS :
1. Se réveille.
2. Vérifie que tous les composants sont présents : processeur, RAM, carte graphique, disques... (c'est le POST).
3. Initialise le matériel (met les disques, les ports USB, la carte graphique en état de marche).
4. Cherche un système d'exploitation sur les disques (dans l'ordre que vous avez défini).
5. Charge le système d'exploitation en mémoire et lui passe la main.

⚠️ Dans le BIOS de Windows il faut que le SATA soit sur AHCI ⚠️

[⬆️ up](#Hardware)

---

## GRUB

**GRUB** => (Grand Unified Bootloader)

- GRUB: juste après le menu de la carte mère (le BIOS)

Après l'UEFI vient le GRUB pour choisir l'OS.

GRUB vs. Windows Boot Manager

GRUB reconnaît tout les OS - Windows Boot Manager = très limité !

[⬆️ up](#Hardware)

---
