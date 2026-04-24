# Hardware PC

- (Carte mère)[#carte-mère]
- CPU
- RAM
- Carte réseau
- (Carte graphique)[#carte-graphique]
- GPU
- Alimentation
- Slot
- Pile CMOS

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
ATX	            30,5 × 24,4 cm	    PC fixe standard
Micro-ATX	    24,4 × 24,4 cm	    PC fixe compact
Mini-ITX	    17 × 17 cm	        PC tout petit (HTPC, serveur maison)
Propriétaire	Variable	        Laptops, PC Dell/HP/Lenovo préfabriqués (non standard)
```

## Carte graphique

`lspci -v | grep -A 10 "VGA"`
