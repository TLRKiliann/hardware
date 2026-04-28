# Kernel RAM CPU Process PID

## 1. Tu donnes l’ordre de démarrer l’application

Exemple : tu tapes `firefox` dans un terminal ou tu cliques sur une icône.  
Le shell (ou l’environnement graphique) appelle un appel système `fork()` puis `exec()`.

## 2. Le Kernel crée un **Process**

- **Process** = une instance en cours d’exécution d’un programme.
- Le kernel (noyau) alloue un **PID** (Process IDentifier), un numéro unique pour identifier ce process.
- Il crée une structure de données interne (`task_struct`) qui contient :
  - le PID
  - l’état (en cours d’exécution, endormi, zombie...)
  - les pointeurs vers la mémoire utilisée
  - le contexte CPU (registres, compteur programme)

## 3. Gestion de la **RAM** par le Kernel

- Le kernel réserve (ou mappe) des zones de **mémoire virtuelle** pour :
  - le code du programme (exécutable)
  - ses données initialisées / non initialisées
  - la pile (stack)
  - le tas (heap) pour allocations dynamiques (`malloc`)
- Cela se fait via la MMU (Memory Management Unit) et des tables de pages.
- La RAM physique peut ne pas être attribuée immédiatement en totalité : Linux utilise l’**allocation paresseuse** + la **demand paging**.
- Si la RAM est pleine, le kernel peut **swapper** (envoyer des pages sur le disque/swap) ou tuer un process (OOM Killer).

## 4. Utilisation du **CPU**

- Le process finit par recevoir du temps CPU pour exécuter ses instructions.
- L’ordonnanceur (scheduler) du kernel alterne très vite entre tous les processus actifs (multitâche préemptif).
- Chaque process a un **compteur de programme** et des **registres** sauvegardés/restaurés à chaque changement de contexte.
- Si le process fait une entrée/sortie (disque, réseau), il est mis en attente (état “bloqué”) et le CPU passe à un autre process.

## 5. Ce que tu vois en pratique sur Debian

```bash
ps aux | grep firefox
```
→ colonne PID, %CPU, %MEM, VSZ (mémoire virtuelle), RSS (RAM physique réelle).

```bash
htop
```
→ arbre des processus, chaque application = un processus principal + souvent plusieurs threads (visibles avec `-T`).

```bash
cat /proc/<PID>/status
```
→ contient les infos exactes que le kernel maintient pour ce process (mémoire, état, UID, etc.)

## Résumé de la chaîne

**Application → Kernel → Process (PID) → RAM allouée (virtuelle puis physique) → CPU (temps d’exécution)**.

Le kernel contrôle tout : protection mémoire, isolation des processus, ordonnancement, communication si nécessaire (pipes, signaux, sockets).

**Important** : un même binaire lancé deux fois → deux PIDs différents, deux espaces mémoire distincts, mais le code en RAM peut être partagé (mémoire partagée pour les bibliothèques).

---

## Le CPU, en une phrase

> **Le CPU exécute *mécaniquement* des instructions très simples stockées en RAM.**

Sans CPU, votre application n’est qu’un tas de données qui dort dans la RAM ou sur le disque. Le CPU est le seul composant qui *fait avancer* le programme.

---

## 1. Ce que le CPU sait faire (jeu d’instructions)

Un CPU ne comprend que des opérations basiques, par exemple :

- Lire un nombre depuis la RAM vers une **zone de travail interne** (registre)
- Additionner deux nombres
- Comparer deux nombres (ex : est-ce que X > Y ?)
- Sauter à une autre instruction (si condition vraie ou fausse)
- Écrire un nombre de la RAM vers un périphérique (écran, disque…)

Tout programme – Firefox, Nano, un jeu – est traduit en *millions* de ces petites instructions.

---

## 2. Le rôle fondamental : le "cycle de lecture/exécution"

Le CPU tourne en boucle :

1. **Lire** l'instruction suivante (dans la RAM, à l'adresse indiquée par un registre spécial = compteur ordinal)
2. **Décoder** l'instruction (ex: "ajouter ces deux nombres")
3. **Exécuter** (ex: l'unité arithmétique fait l'addition)
4. **Écrire** le résultat dans un registre ou la RAM
5. Passer à l'instruction suivante

C’est **cette répétition frénétique** (plusieurs milliards de fois par seconde) qui donne l’illusion qu’un programme avance.

---

## 3. Ce que le CPU fait pour *votre* application

Quand vous démarrez une application (ex: `firefox`) :

1. Le kernel charge le code de Firefox dans la RAM
2. Le kernel dit au CPU : "va à l'adresse mémoire où commence le code de Firefox"
3. Le CPU commence à exécuter *les instructions de Firefox*

Pendant l'exécution :
- Firefox a besoin d'**additionner** des coordonnées d'affichage → le CPU le fait
- Firefox doit **comparer** si une touche du clavier a été pressée → le CPU le fait
- Firefox doit **sauter** à la partie du code qui gère le réseau → le CPU le fait

Sans CPU, rien ne se passe. La RAM n’a aucune capacité à "s'exécuter" toute seule.

---

## 4. Pourquoi plusieurs processus (PID) peuvent tourner "en même temps"

Un seul CPU ne peut exécuter qu’**une seule instruction à la fois**. Mais :

- Le CPU va très vite (milliards d’instructions par seconde)
- Le kernel interrompt très régulièrement le processus en cours et donne le CPU à un autre

C’est l’**ordonnancement** :
- Firefox a le CPU pendant 0,01 s
- Puis le CPU bascule sur votre terminal
- Puis sur le serveur web
- etc.

Comme ces changements sont ultra-rapides, on a l’impression que tout tourne simultanément (multitâche).

---

## 5. Analogie simple pour tout lier

Imaginez une **cuisine** :

- **RAM** = la grande table avec toutes les recettes (code) et les ingrédients (données)
- **CPU** = le cuisinier
- **Processus (PID)** = un ticket de commande en cours
- **Ordonnancement** = le cuisinier passe 2 minutes sur un ticket, puis 2 minutes sur un autre

---

## En résumé pour votre système Ubuntu/Debian

Quand vous lancez une application :

- Le kernel prépare la RAM (code, données, pile)
- Il crée un PID
- Puis il **donne le CPU** à ce nouveau processus
- Le CPU exécute mécaniquement les instructions une par une
- Le kernel interrompt régulièrement pour partager le CPU entre tous les processus

**Vous avez tout compris si vous retenez :**  
*La RAM stocke passivement le programme ; le CPU l’exécute activement, instruction par instruction.*

## Les commandes Linux

Voici un tableau récapitulatif des méthodes les plus efficaces :

| Méthode | Type | Facile à utiliser | Idéal pour... |
| :--- | :--- | :--- | :--- |
| **htop** | Interactif | Très facile | Une vue d'ensemble claire et interactive |
| **top** | Interactif | Facile | Un suivi rapide sans installation |
| **ps** | Commande unique | Très facile | Un instantané précis d'un processus spécifique |
| **pidstat** | Commande unique | Moyen | Une analyse statistique détaillée dans le temps |

---

### 💻 Méthode 1 : htop - La référence conviviale

`htop` est une version améliorée et bien plus agréable de `top`. Il affiche une liste colorée et interactive des processus que vous pouvez trier facilement.

**Utilisation :**
Lancez simplement `htop` dans votre terminal. Une fois dans l'interface :
- **Pour trier par utilisation de la RAM** : Appuyez sur la touche **`F6`** , puis sélectionnez **`PERCENT_MEM`** ou **`M_RESIDENT`** avec les flèches, et validez avec `Entrée`. Les processus les plus gourmands remonteront en tête.
- **Pour chercher votre application** : Appuyez sur **`F3`** et tapez le nom de votre processus (ex: `firefox`).
- **Quitter** : Appuyez sur la touche **`F10`** ou `q`.

> 💡 **Que signifient les colonnes mémoire ?**
> - **`VIRT`** (Virtual) : Toute la mémoire que le processus a réservée (y compris ce qui est dans le swap).
> - **`RES`** (Resident) : La mémoire physique (RAM) réellement utilisée, que le processus ne peut pas échanger.
> - **`MEM%`** : Le pourcentage de votre RAM totale utilisé par le processus.
> C'est la colonne **`RES`** qui vous intéresse le plus.

---

### 📊 Méthode 2 : top - L'outil classique présent par défaut

`top` est installé par défaut sur toutes les distributions Debian. C'est un excellent outil pour un diagnostic rapide si vous ne voulez rien installer.

**Utilisation :**
Tapez `top` dans le terminal.
- **Trier par mémoire** : Une fois dans `top`, appuyez sur la touche **`Shift` + `M`** (M majuscule) pour trier les processus par utilisation de la mémoire, du plus gros au plus petit.
- **Quitter** : Appuyez sur la touche **`q`**.

---

### 🎯 Méthode 3 : ps - Pour obtenir une valeur précise et unique

Cette méthode est parfaite si vous connaissez le nom ou l'ID (PID) du processus et que vous voulez une valeur ponctuelle, sans interface interactive.

**1. Trouvez le PID de votre application :**
```bash
ps aux | grep -i "nom_de_votre_application"
```
(Remplacez `nom_de_votre_application` par le nom du processus, comme `firefox` ou `chrome`).

**2. Affichez la mémoire utilisée par ce PID :**
Remplacez `1234` par le PID trouvé à l'étape précédente.
```bash
ps -o rss,vsz,cmd -p 1234
```
- **`RSS`** (Resident Set Size) : C'est la RAM physique utilisée par le processus (en Kilooctets). La valeur la plus pertinente.
- **`VSZ`** (Virtual Set Size) : La mémoire virtuelle allouée.

---

### 📈 Méthode 4 : pidstat - Pour une analyse statistique continue

`pidstat` fait partie du paquet `sysstat` et est très utile pour voir l'évolution de la consommation mémoire à intervalles réguliers.

**Installation :**
```bash
sudo apt install sysstat
```

**Utilisation :**
```bash
pidstat -p PID -r 2
```
(Remplacez `PID` par le numéro du processus).
- L'option `-r` affiche les rapports de mémoire.
- Le `2` signifie que la commande va afficher un rapport toutes les 2 secondes.

La colonne **`RSS`** vous donnera la RAM physique utilisée.

---

En résumé, si vous devez choisir :
- **Pour une analyse rapide et confortable** : Installez et utilisez **`htop`** . C'est le plus intuitif.
- **Si vous ne pouvez pas installer d'outils** : Utilisez **`top`** (avec `Shift+M` pour trier).
- **Pour un script ou une valeur ponctuelle** : Utilisez **`ps`** .

Tous ces outils sont disponibles dans les dépôts officiels Debian

Voici les principales commandes `ps` à connaître sous Linux/Debian, organisées par usage courant.

## 📋 Tableau des commandes essentielles

| Commande | Effet | Quand l'utiliser |
| :--- | :--- | :--- |
| `ps aux` | Affiche TOUS les processus (utilisateur, système) | **90% des cas** - La plus utile |
| `ps axjf` | Arborescence des processus (fils/parents) | Pour visualiser les relations entre processus |
| `ps -e` | Tous les processus (format simple) | Version courte de `ps aux` |
| `ps u -p PID` | Détails d'un processus spécifique | Pour analyser un PID précis |
| `ps --sort=-%mem` | Trier par consommation mémoire | Pour trouver les plus gourmands |
| `ps -L PID` | Afficher les threads d'un processus | Pour debugger les applications multithreads |

---

## 🎯 Les 3 commandes indispensables

### 1️⃣ `ps aux` - La plus complète (à retenir ABSOLUMENT)

```bash
ps aux
```

**Que montrent les colonnes ?**

- `USER` : propriétaire du processus
- `PID` : identifiant unique
- `%CPU` : utilisation CPU
- `%MEM` : utilisation RAM (en pourcentage)
- `VSZ` : mémoire virtuelle totale (Ko)
- `RSS` : mémoire physique RAM (Ko) - **le plus important**
- `TTY` : terminal associé
- `STAT` : état du processus (R=running, S=sleep, Z=zombie)
- `START` : heure de démarrage
- `TIME` : temps CPU cumulé
- `COMMAND` : commande complète

**Variante utile :**

```bash
ps aux --sort=-%mem | head -10  # Top 10 des plus gourmands en RAM
ps aux --sort=-%cpu | head -10  # Top 10 des plus gourmands en CPU
```

---

### 2️⃣ `ps axjf` - L'arborescence des processus

```bash
ps axjf
```

Cette commande montre **qui a créé quoi** sous forme d'arbre :
```
  PID  PPID  ... COMMAND
    1     0      /sbin/init
  800     1      ├─ sshd
  801   800      │  └─ bash
 1024   801      │     └─ firefox
  850     1      ├─ cron
```

**Utile pour :**
- Comprendre les dépendances entre processus
- Trouver le parent d'un processus orphelin
- Visualiser la structure des services système

---

### 3️⃣ `ps u -p PID` - Cibler un processus spécifique

```bash
ps u -p 1234  # Remplacez 1234 par le vrai PID
```

**Exemple concret :**

```bash
# Trouver le PID de Firefox
pgrep firefox

# Analyser sa mémoire
ps u -p 2847
```

---

## 🔧 Commandes avancées utiles

### Filtrer par utilisateur

```bash
ps -u username           # Processus d'un utilisateur spécifique
ps -U root -u root       # Processus root (réel et effectif)
```

### Formater la sortie soi-même

```bash
ps -e -o pid,user,%mem,comm --sort=-%mem | head -5
```
Les champs disponibles : `pid`, `user`, `%cpu`, `%mem`, `vsz`, `rss`, `comm` (commande), `etime` (temps écoulé)

### Afficher la mémoire en Mo (plus lisible)

```bash
ps aux | awk '{print $2, $3, $4, $6/1024 " Mo", $11}' | head -10
```

### Trouver les processus zombies

```bash
ps aux | awk '$8=="Z" {print}'
```

### Surveillance immédiate (équivalent de `top`)

```bash
watch -n 2 'ps aux --sort=-%mem | head -15'
```

---

## 💡 Différence entre `ps aux` et `ps -ef`

| Option | Origine | Style | Identique ? |
| :--- | :--- | :--- | :--- |
| `ps aux` | BSD (Unix) | Plus lisible, colonnes explicites | ✅ Oui, même infos |
| `ps -ef` | System V (Linux) | Format standard, plus technique | ✅ Oui, mais colonnes différentes |

**Mon conseil :** Utilisez `ps aux`, c'est plus convivial.

---

## 🎓 Résumé mnémotechnique

```
ps aux          → TOUS les processus (a=all, u=user détaillé, x=processus sans terminal)
ps axjf         → Arbre Joli Format (jf = job format + forest)
ps -p PID       → précis sur un PID
ps -L PID       → Lister les threads
ps --sort=-%mem → trié par mémoire
```

**La seule à retenir pour 95% des cas : `ps aux`**