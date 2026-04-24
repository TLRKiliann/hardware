# Hardware PC

- [Hardware PC](#hardware-pc)
  - [Carte mère](#carte-mère)
    - [Les composants "carte mère" typiques à chercher](#les-composants-carte-mère-typiques-à-chercher)
  - [CPU](#cpu)
  - [RAM](#ram)
  - [Carte graphique](#carte-graphique)
    - [GPU](#gpu)
  - [Carte réseau (internet)](#carte-réseau-internet)

---

## Carte mère 

La carte mère (ou motherboard en anglais) est le grand circuit imprimé qui relie et fait communiquer tous les composants d'un ordinateur.

C'est le squelette et le système nerveux du PC : tout se branche dessus, tout passe par elle.

**Matériaux**

Faite en fibre de verre, de resine, et de cuivre.

---

**Ce qu'il y a sur ou connecté à la carte mère**

```
Type	                                Exemples
------------------------------------------------------------------------------------------------------
Soudés directement	                    Chipset, BIOS/UEFI (puce), horloge temps réel (CMOS)
Dans des sockets (prises)	            CPU, RAM
Dans des slots (connecteurs longs)	    Carte graphique (PCIe), carte son, carte Wi-Fi
Branchés par câbles	                    Disques durs (SATA), SSD (SATA ou NVMe via M.2), alimentation, 
                                        ventilateurs
En façade (à l'arrière)	                Ports USB, Ethernet, sortie audio, HDMI de la carte mère
```

### Les composants "carte mère" typiques à chercher

**La totale**

`lspci`

**Plus en détail**

`lspci -vvv`

**Host bridge => Le pont principal CPU ↔ chipset/RAM/PCIe**

`lspci -vvv | grep -i "host bridge"`

ou

`sudo dmidecode -t baseboard`

**PCI bridge => Un pont qui étend le bus PCIe (souvent dans le chipset)**

`lspci -vvv | grep -i "pci bridge"`

**ISA bridge ou LPC bridge => Le pont vers les vieux périphériques lents (clavier, souris, CMOS). C'est souvent le chipset lui-même**

`lspci -vvv | grep -i "isa bridge"`

**SATA controller => Le contrôleur pour vos disques SATA (mode AHCI ou IDE)**

`lspci -vvv | grep -i "sata"`

**USB controller => Les ports USB**

`lspci -vvv | grep -i "usb"`

**Audio device => La carte son**

`lspci -vvv | grep -i "audio"`

**Ethernet controller => La carte réseau**

`lspci -vvv | grep -i "ethernet"`

**Tout ce qui touche à la carte mère**

`lspci | grep -E "bridge|sata|usb|audio|ethernet"`

---

**Les formes de carte mère (formats)**

```
Format          Taille	            Utilisation typique
------------------------------------------------------------------------------------------
ATX	            30,5 × 24,4 cm	    PC fixe standard
Micro-ATX	    24,4 × 24,4 cm	    PC fixe compact
Mini-ITX	    17 × 17 cm	        PC tout petit (HTPC, serveur maison)
Propriétaire	Variable	        Laptops, PC Dell/HP/Lenovo préfabriqués (non standard)
```

---

## CPU

Intel ou AMD

Le CPU (Central Processing Unit), ou processeur en français, est le cerveau de l'ordinateur.
En une phrase

Le CPU exécute les instructions des programmes : il lit, calcule, compare, déplace, et décide de tout ce que fait l'ordinateur.

SATA, AHCI, NVMe, chipset – sont tous des composants qui existent pour une seule raison : faire communiquer le CPU avec le reste.

Le CPU est le boss. Le chipset, le bus PCIe, le protocole NVMe... tout est conçu pour que le CPU puisse lire et écrire des données le plus vite possible.

**Un CPU moderne exécute des milliards d'opérations par seconde**

```
Type d'opération	    Exemple
----------------------------------------------------------------------------------
Arithmétique	        2 + 3 = 5
Logique	                Est-ce que A est plus grand que B ?
Comparaison	            Ce mot est-il identique à celui-ci ?
Déplacement	            Prendre cette donnée en RAM et la mettre dans un registre
Saut	                Si condition vraie, aller à l'instruction n°42
Appel système	        Demander au disque dur de me lire un fichier
```

**Les composants internes au CPU**

```
Composant	        Rôle
----------------------------------------------------------------------------------------------------------
Cœurs (cores)	    Chaque cœur peut exécuter une tâche indépendante (ex: 4 cœurs = 4 tâches en parallèle)
Threads	            Des "cœurs virtuels" pour mieux répartir le travail (ex: 4 cœurs / 8 threads)
Cache L1, L2, L3	Mémoire ultra-rapide intégrée au CPU (plus petite que la RAM mais bien plus rapide)
Horloge (fréquence)	Battement qui cadence le CPU (ex: 3,5 GHz = 3,5 milliards de cycles par seconde)
```

**Modèle du CPU**

`lscpu | grep "Model name"`

**Nombre de cœurs**

`nproc ou lscpu | grep "Core(s)"`

**Fréquence actuelle**

`cat /proc/cpuinfo | grep "MHz" | head -1`

**Infos détaillées**

`lscpu`

---

## RAM

La RAM (Random Access Memory), ou mémoire vive en français, est la mémoire de travail temporaire de l'ordinateur.

La RAM stocke temporairement les données et les instructions dont le CPU a besoin immédiatement ou dans les secondes qui viennent, pour y accéder extrêmement vite.

La RAM est un tampon rapide entre le CPU (ultra-rapide) et le disque (ultra-lent).

**Pourquoi on a besoin de RAM ?**

Sans RAM, le CPU devrait lire chaque instruction directement sur le disque dur. Un disque dur met 10 millisecondes à trouver une donnée. Une RAM met 10 nanosecondes

**Les types de RAM**

***DDR*** = Double Data Rate (transfère deux fois par cycle). Le chiffre = la génération.

```
Type	    Usage	                                            Fréquence typique
----------------------------------------------------------------------------------
DDR4	    PC fixes et portables (ancienne génération)	        2133 - 3200 MHz
DDR5	    PC fixes et portables (moderne)	                    4800 - 8000+ MHz
LPDDR4/5	Portables ultra-minces, smartphones	                Basse consommation
SODIMM	    Format portable (plus petite barrette)	            Variable
```

```
Support	            Temps d'accès typique
------------------------------------------
Registres CPU	    ~0,3 ns
Cache L1	        ~1 ns
Cache L2	        ~4 ns
RAM	                ~10-20 ns
SSD NVMe	        ~10 000 ns (10 µs)
SSD SATA	        ~50 000 ns (50 µs)
HDD mécanique	    ~10 000 000 ns (10 ms)
```

**RAM totale et utilisée**

`free -h`

**Détails des barrettes**

`sudo dmidecode -t memory`

**Type et fréquence**

`sudo lshw -short -class memory`

**Vitesse réelle**

`sudo dmidecode -t memory | grep -i "speed"`

---

**Ce qu'il ne faut pas confondre**

```
Terme	            Nature	                    Persistant ?	            Rôle
-------------------------------------------------------------------------------------------------
RAM	                Mémoire vive	            Non (effacée à l'arrêt)	    Travail temporaire
SSD / HDD	        Stockage	                Oui	                        Sauvegarde permanente
Cache CPU	        Mémoire vive ultra-rapide	Non	                        Travail immédiat
VRAM (carte graph)	Mémoire vive GPU	        Non	                        Textures, images 3D
```

---

## Carte graphique

`lspci -v | grep -A 10 "VGA"`

### GPU

## Carte réseau (internet)