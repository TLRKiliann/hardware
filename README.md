# HARDWARE

1) [SATA](#sata)
2) [AHCI](#ahci)
3) [RAID](#raid)
4) [NVMe](#NVMe)
5) [PCIe](#PCIe)
6) [Bus](#Bus)
7) [Chipset](#Chipset)
8) [UEFI (BIOS)](#uefi-bios)
9) [GRUB](#grub)
  
## SATA

***SATA*** => (Serial ATA)

SATA (Serial ATA) est la norme de câble et de connecteur qui sert à brancher la plupart des disques durs (HDD) et SSD à la carte mère.

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

***AHCI*** => (Advanced Host Controller Interface)

- AHCI: règles de circulation pour le SATA et NCQ (PROTCOL) HDD + SSD.

---

## RAID

***RAID*** (Redundant Array of Independent Disks)

- RAID: méthode pour faire travailler plusieurs disks ensemble. 

---

## NVMe

***NVMe*** => (Non-Volatile Memory Express) 128 - 256 - 512 - 1024 GB

- NVMe: Plus rapide que AHCI et fait pour SSD M.2 (connection du SSD M.2 sur Slot M.2 de la mother board)
- Slot M.2: emplacement direct sur la carte mère (remplace SATA).

un SSD NVMe, lui, n'utilise ni câble SATA, ni interface SATA. Il se branche directement dans un slot M.2 et parle un autre protocole (NVMe) via un autre bus (PCI Express).

- IDE: Ne jamais utiliser pour les SSD (vieille techno !)

[⬆️ up](#Hardware)

---

## PCIe

***PCIe*** => (Peripheral Component Interconnect Express)

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

### Chipset

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

***BIOS*** => (Basic Input/Output System) = UEFI

***UEFI*** => (Unified Extensible Firmware Interface)

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

***GRUB*** => (Grand Unified Bootloader)

- GRUB: juste après le menu de la carte mère (le BIOS)

Après l'UEFI vient le GRUB pour choisir l'OS.

GRUB vs. Windows Boot Manager

GRUB reconnaît tout les OS - Windows Boot Manager = très limité !

[⬆️ up](#Hardware)

---
