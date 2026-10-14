# Le format Windows .exe

> **Comment lire ce document :** chaque grande section s'ouvre sur un court encadré **« En clair »**. Lis-les si tu débutes et que tu veux juste comprendre l'idée, sans jargon. Tout ce qui se trouve sous l'encadré est la référence technique complète — structures, offsets d'octets, tables de drapeaux — pour celles et ceux qui ont réellement besoin d'analyser ou de construire ces fichiers. Ne saute rien si tu es débutant et que tu veux *devenir* expert : les encadrés « En clair » sont une rampe d'accès, pas un substitut.

---

## Table des matières

1. [Pour commencer, qu'est-ce qu'un .exe](#pour-commencer-quest-ce-quun-exe)
2. [Historique et évolution](#historique-et-évolution)
3. [Le format PE (Portable Executable)](#le-format-pe-portable-executable)
4. [Structure détaillée du fichier](#structure-détaillée-du-fichier)
   - [En-tête DOS (en-tête MZ)](#1-en-tête-dos-en-tête-mz)
   - [Stub DOS](#2-stub-dos)
   - [Signature PE](#3-signature-pe)
   - [En-tête de fichier COFF](#4-en-tête-de-fichier-coff)
   - [En-tête optionnel](#5-en-tête-optionnel)
   - [Table des sections](#6-table-des-sections)
   - [Sections (données brutes)](#7-sections-données-brutes)
5. [Octets magiques et marqueurs](#octets-magiques-et-marqueurs)
6. [Référence des sections standard](#référence-des-sections-standard)
7. [Répertoires de données](#répertoires-de-données)
8. [Comment Windows charge un .exe](#comment-windows-charge-un-exe)
9. [Créer un .exe](#créer-un-exe)
   - [Depuis C/C++ (MSVC)](#depuis-cc-msvc)
   - [Depuis C/C++ (MinGW/GCC)](#depuis-cc-mingwgcc)
   - [Depuis C# (.NET)](#depuis-c-net)
   - [Depuis Rust](#depuis-rust)
   - [Depuis Python (PyInstaller)](#depuis-python-pyinstaller)
   - [Depuis l'assembleur (NASM)](#depuis-lassembleur-nasm)
10. [Inspecter un .exe](#inspecter-un-exe)
11. [Variantes du PE](#variantes-du-pe)
12. [Fonctionnalités de sécurité](#fonctionnalités-de-sécurité)
13. [Pièges courants et remarques](#pièges-courants-et-remarques)
14. [Glossaire](#glossaire)

---

## Pour commencer, qu'est-ce qu'un .exe

> **En clair :** double-cliquer sur un `.exe` ne lance pas du « code » directement — cela lance une disposition de fichier très précise et très rigide, que Windows sait lire. Cette disposition s'appelle **PE (Portable Executable)**. Imagine un `.exe` comme un conteneur qui arrive dans un port : avant que la grue (Windows) puisse le décharger, une pile de papiers est agrafée à l'avant pour indiquer aux dockers ce qu'il contient, où va chaque caisse, et quelles caisses sont fragiles (exécutables) ou simplement de la marchandise (données). Ce document passe en revue cette paperasse, caisse par caisse.

Un fichier `.exe` est le format binaire standard des programmes Windows. L'ouvrir pousse le système d'exploitation à le charger en mémoire et à commencer à exécuter son code machine. Sous le capot, pratiquement tous les `.exe` modernes utilisent le format **PE (Portable Executable)**, un descendant direct de l'ancien COFF (Common Object File Format) d'Unix.

Le même format PE est partagé par plusieurs types de fichiers que tu as probablement déjà croisés sans te rendre compte qu'ils reposent tous sur le même squelette :

| Extension | Ce que c'est | Version en clair |
|---|---|---|
| `.exe` | Exécutable autonome | Le programme lui-même |
| `.dll` | Bibliothèque à liaison dynamique | Du code partagé que d'autres programmes empruntent |
| `.sys` | Pilote du noyau | Du code qui dialogue directement avec le matériel |
| `.scr` | Économiseur d'écran | Littéralement un simple `.exe` renommé |
| `.ocx`, `.cpl`, `.efi` | Exécutables spécialisés | Contrôles ActiveX, éléments du Panneau de configuration, applications de firmware |

**Pourquoi un non-développeur s'y intéresserait-il ?** Voici quelques vraies raisons pour lesquelles ces connaissances sont utiles, même si tu n'écris pas de code :
- Comprendre pourquoi les antivirus signalent certains fichiers (des sections PE compressées ou obfusquées sont un signal d'alarme majeur).
- Savoir ce que veulent vraiment dire les avertissements « cette application n'est pas signée numériquement » (section [Fonctionnalités de sécurité](#fonctionnalités-de-sécurité)).
- La curiosité autour de ce message énigmatique, « This program cannot be run in DOS mode » : tu verras exactement où ce texte se cache, quelques sections plus loin.

---

## Historique et évolution

> **En clair :** Windows est rétrocompatible avec lui-même depuis plus de 40 ans, et le format `.exe` en est la preuve. Chaque nouveau format a conservé un minuscule fossile de l'ancien, boulonné à l'avant, si bien qu'un `.exe` compilé aujourd'hui « sait » encore techniquement dialoguer avec un ordinateur de 1981.

| Époque | Format | Remarques |
|-----|--------|-------|
| MS-DOS (1981–1990) | **MZ** | Format segmenté simple, nommé d'après Mark Zbikowski |
| Windows 3.x (1990–1995) | **NE** (New Executable) | Ajout des ressources et des imports ; 16 bits |
| Windows 9x / NT (1993+) | **PE** (Portable Executable) | 32 bits ; toujours la base des exécutables modernes |
| Windows XP 64 bits et + | **PE32+** (aussi appelé PE64) | Variante 64 bits du PE |
| Windows moderne | **PE32 / PE32+** | Même format, étendu avec des fonctionnalités de sécurité |

L'en-tête MZ est toujours présent dans tout `.exe` moderne comme fossile de compatibilité — il contient un minuscule programme 16 bits qui affiche « This program cannot be run in DOS mode ». Voilà cette fameuse ligne, et tu sais maintenant exactement pourquoi elle est là : c'est un filet de sécurité hérité de 1981, qui voyage encore dans chaque programme Windows compilé en 2026.

---

## Le format PE (Portable Executable)

> **En clair :** ici, « portable » ne veut pas dire « petit » ni « facile à copier » — cela signifie que le format a été conçu pour fonctionner sur *différents types de processeurs* (l'objectif initial était que x86, MIPS, Alpha et PowerPC partagent une seule disposition de fichier). Le schéma ci-dessous est le plan d'étage de chaque `.exe` de ton ordinateur, de haut en bas, dans l'ordre exact où les octets apparaissent sur le disque.

Le format PE a été introduit avec Windows NT 3.1 en 1993, conçu par Microsoft en collaboration avec DEC. Il devait être portable entre architectures de processeurs (x86, MIPS, Alpha, PowerPC, ARM) — d'où son nom, « Portable ».

La disposition générale se présente ainsi :

```
┌───────────────────────────────┐  Offset 0x00
│       En-tête DOS (MZ)        │  64 octets
├───────────────────────────────┤
│           Stub DOS            │  Variable (généralement ~64 octets)
├───────────────────────────────┤
│         Signature PE          │  4 octets ("PE\0\0")
├───────────────────────────────┤
│     En-tête fichier COFF      │  20 octets
├───────────────────────────────┤
│       En-tête optionnel       │  96 octets (PE32) / 112 octets (PE32+)
│   + Répertoires de données    │  + 128 octets (16 entrées × 8 octets)
├───────────────────────────────┤
│      Table des sections       │  N × 40 octets
├───────────────────────────────┤
│    Données section 1 (.text)  │
├───────────────────────────────┤
│    Données section 2 (.data)  │
├───────────────────────────────┤
│              ...              │
└───────────────────────────────┘
```

À lire de haut en bas : on trouve d'abord un fossile de 1981 (en-tête DOS + stub), puis le tampon « oui, c'est bien un fichier PE », puis deux en-têtes qui décrivent l'ensemble du programme, puis une table des matières du contenu réel, puis le contenu lui-même.

---

## Structure détaillée du fichier

### 1. En-tête DOS (en-tête MZ)

> **En clair :** c'est l'étiquette « retour à l'expéditeur » collée à l'avant de chaque `.exe`. Son seul rôle moderne est d'indiquer où commence le *véritable* en-tête. Tout le reste est vestigial — Windows n'y jette presque pas un œil, à l'exception d'un seul champ.

Situé à l'**offset 0x00**, toujours sur 64 octets. Les deux premiers octets sont toujours `4D 5A` (ASCII : `MZ`) — les initiales de Mark Zbikowski, l'ingénieur qui a conçu le format d'origine.

```c
typedef struct _IMAGE_DOS_HEADER {
    WORD  e_magic;      // 0x00 — Nombre magique : 0x5A4D ("MZ")
    WORD  e_cblp;       // 0x02 — Octets sur la dernière page du fichier
    WORD  e_cp;         // 0x04 — Pages dans le fichier
    WORD  e_crlc;       // 0x06 — Relocalisations
    WORD  e_cparhdr;    // 0x08 — Taille de l'en-tête en paragraphes
    WORD  e_minalloc;   // 0x0A — Nombre minimum de paragraphes supplémentaires requis
    WORD  e_maxalloc;   // 0x0C — Nombre maximum de paragraphes supplémentaires requis
    WORD  e_ss;         // 0x0E — Valeur initiale (relative) de SS
    WORD  e_sp;         // 0x10 — Valeur initiale de SP
    WORD  e_csum;       // 0x12 — Somme de contrôle
    WORD  e_ip;         // 0x14 — Valeur initiale de IP
    WORD  e_cs;         // 0x16 — Valeur initiale (relative) de CS
    WORD  e_lfarlc;     // 0x18 — Adresse fichier de la table de relocalisation
    WORD  e_ovno;       // 0x1A — Numéro d'overlay
    WORD  e_res[4];     // 0x1C — Mots réservés
    WORD  e_oemid;      // 0x24 — Identifiant OEM
    WORD  e_oeminfo;    // 0x26 — Informations OEM
    WORD  e_res2[10];   // 0x28 — Mots réservés
    LONG  e_lfanew;     // 0x3C — *** Offset fichier de l'en-tête PE ***
} IMAGE_DOS_HEADER;
```

> **Champ clé :** `e_lfanew` à l'offset `0x3C` (octet 60) — cette valeur de 4 octets donne l'offset fichier de l'en-tête PE. Lis-la toujours en premier quand tu analyses un fichier PE. Tous les autres champs de cette structure n'existent que pour que le vieux DOS puisse (en théorie) encore charger le fichier ; Windows moderne les ignore.

---

### 2. Stub DOS

> **En clair :** c'est un vrai petit programme 16 bits fonctionnel — pas seulement du texte. Si tu lançais par miracle un `.exe` moderne sur un vrai MS-DOS, c'est ce stub qui s'exécuterait, afficherait le message « cannot be run in DOS mode » et quitterait poliment.

Un minuscule programme 16 bits qui s'exécute si tu lances le fichier depuis un véritable environnement DOS. Il affiche le message « This program cannot be run in DOS mode. » puis se termine. Tu peux le remplacer par un stub personnalisé si besoin (certains outils le font pour du branding ou pour embarquer des métadonnées supplémentaires).

---

### 3. Signature PE

> **En clair :** un tampon d'authenticité de 4 octets. Après le fossile DOS, c'est le moment où le fichier dit « bon, l'ancien monde, c'est fini — tout ce qui suit est le vrai format moderne ».

Située à l'offset `e_lfanew`. Toujours exactement **4 octets** :

```
50 45 00 00   →  "PE\0\0"
```

---

### 4. En-tête de fichier COFF

> **En clair :** la page de garde du manifeste d'expédition. Avant même de regarder la cargaison, elle t'indique : de quel type de chariot élévateur tu auras besoin (l'architecture du processeur), combien de caisses arrivent (le nombre de sections) et quand l'expédition a été emballée (l'horodatage).

Suit immédiatement la signature PE. **20 octets**.

```c
typedef struct _IMAGE_FILE_HEADER {
    WORD  Machine;              // Architecture de processeur ciblée
    WORD  NumberOfSections;     // Combien de sections suivent
    DWORD TimeDateStamp;        // Horodatage Unix de l'édition de liens du fichier
    DWORD PointerToSymbolTable; // Généralement 0 dans les exécutables
    DWORD NumberOfSymbols;      // Généralement 0 dans les exécutables
    WORD  SizeOfOptionalHeader; // Taille de l'en-tête optionnel qui suit
    WORD  Characteristics;      // Drapeaux d'attributs du fichier
} IMAGE_FILE_HEADER;
```

#### Valeurs de Machine (courantes) :

| Valeur | Architecture |
|-------|-------------|
| `0x014C` | x86 (Intel 32 bits) |
| `0x8664` | x86-64 (AMD64, 64 bits) |
| `0xAA64` | ARM64 (AArch64) |
| `0x01C4` | ARM (Thumb-2) |
| `0x0200` | Intel Itanium (IA-64) |

#### Drapeaux de Characteristics :

| Drapeau | Valeur | Signification |
|------|-------|---------|
| `IMAGE_FILE_EXECUTABLE_IMAGE` | `0x0002` | Exécutable valide |
| `IMAGE_FILE_LARGE_ADDRESS_AWARE` | `0x0020` | Peut utiliser plus de 2 Go d'espace d'adressage |
| `IMAGE_FILE_32BIT_MACHINE` | `0x0100` | Machine 32 bits |
| `IMAGE_FILE_DLL` | `0x2000` | C'est une DLL, pas un .exe |
| `IMAGE_FILE_SYSTEM` | `0x1000` | Fichier système (pilote) |

---

### 5. En-tête optionnel

> **En clair :** malgré son nom, rien dans cet en-tête n'est réellement optionnel pour un programme exécutable — il est « optionnel » comme un moteur est un équipement « optionnel » sur une voiture (techniquement une pièce à part, mais tu n'iras nulle part sans). C'est ici que se trouve le vrai plan : à quel endroit de la mémoire charger le programme, quelle instruction exécuter en premier, combien de pile et de tas réserver.

Cet en-tête existe en deux versions :

- **PE32** (32 bits) : commence par le nombre magique `0x010B`, 96 octets + répertoires de données
- **PE32+** (64 bits) : commence par le nombre magique `0x020B`, 112 octets + répertoires de données

```c
// Version PE32 (simplifiée)
typedef struct _IMAGE_OPTIONAL_HEADER {
    WORD  Magic;                    // 0x010B = PE32, 0x020B = PE32+
    BYTE  MajorLinkerVersion;
    BYTE  MinorLinkerVersion;
    DWORD SizeOfCode;               // Taille totale de toutes les sections .text
    DWORD SizeOfInitializedData;
    DWORD SizeOfUninitializedData;
    DWORD AddressOfEntryPoint;      // RVA de la première instruction à exécuter
    DWORD BaseOfCode;               // RVA où commence la section .text
    DWORD BaseOfData;               // RVA où commence la section .data (PE32 uniquement)
    DWORD ImageBase;                // Adresse de chargement préférée (généralement 0x00400000)
    DWORD SectionAlignment;         // Alignement des sections en mémoire (généralement 0x1000 = 4 Ko)
    DWORD FileAlignment;            // Alignement des sections sur disque (généralement 0x200 = 512 o)
    WORD  MajorOperatingSystemVersion;
    WORD  MinorOperatingSystemVersion;
    WORD  MajorImageVersion;
    WORD  MinorImageVersion;
    WORD  MajorSubsystemVersion;    // Généralement 6 (Vista+) ou 10
    WORD  MinorSubsystemVersion;
    DWORD Win32VersionValue;        // Réservé, doit valoir 0
    DWORD SizeOfImage;              // Taille totale de l'image en mémoire
    DWORD SizeOfHeaders;            // Taille de tous les en-têtes (DOS + PE + table des sections)
    DWORD CheckSum;                 // Somme de contrôle CRC (requise uniquement pour les pilotes)
    WORD  Subsystem;                // Sous-système cible (GUI, console, etc.)
    WORD  DllCharacteristics;       // Drapeaux de fonctionnalités de sécurité
    DWORD SizeOfStackReserve;       // Réservation virtuelle de pile (1 Mo par défaut)
    DWORD SizeOfStackCommit;        // Allocation physique de pile (4 Ko par défaut)
    DWORD SizeOfHeapReserve;        // Réservation virtuelle de tas (1 Mo par défaut)
    DWORD SizeOfHeapCommit;         // Allocation physique de tas (4 Ko par défaut)
    DWORD LoaderFlags;              // Obsolète
    DWORD NumberOfRvaAndSizes;      // Nombre d'entrées du répertoire de données (généralement 16)
    IMAGE_DATA_DIRECTORY DataDirectory[16];
} IMAGE_OPTIONAL_HEADER;
```

#### Valeurs de Subsystem :

| Valeur | Nom | Description |
|-------|------|-------------|
| `1` | NATIVE | Pilote / composant système |
| `2` | WINDOWS_GUI | Application Windows (sans fenêtre de console) |
| `3` | WINDOWS_CUI | Application console |
| `9` | WINDOWS_CE_GUI | Windows CE |
| `10` | EFI_APPLICATION | Application de firmware UEFI |

#### DllCharacteristics (drapeaux de sécurité) :

> **En clair :** c'est une courte liste de contrôle des fonctions défensives que le programme demande à Windows d'activer pour lui — par exemple choisir au hasard l'endroit où il se charge en mémoire (plus difficile à prévoir pour un attaquant), ou refuser d'exécuter la pile (plus difficile à détourner par un dépassement de tampon).

| Drapeau | Valeur | Description |
|------|-------|-------------|
| `DYNAMIC_BASE` | `0x0040` | ASLR : l'image peut être relogée au chargement |
| `FORCE_INTEGRITY` | `0x0080` | Impose la vérification d'intégrité du code |
| `NX_COMPAT` | `0x0100` | Compatible DEP/NX (pile non exécutable) |
| `NO_ISOLATION` | `0x0200` | N'utilise pas l'isolation |
| `NO_SEH` | `0x0400` | Pas de gestion structurée des exceptions |
| `NO_BIND` | `0x0800` | Ne pas lier (bind) l'image |
| `WDM_DRIVER` | `0x2000` | Pilote WDM (modèle de pilote) |
| `GUARD_CF` | `0x4000` | Control Flow Guard activé |
| `TERMINAL_SERVER_AWARE` | `0x8000` | Compatible TS / Bureau à distance |

---

### 6. Table des sections

> **En clair :** une table des matières de la cargaison réelle. Chaque ligne indique : voici un bloc de données, voici son nom (`.text`, `.data`, etc.), voici où il se trouve sur le disque, et voici où il doit être placé en mémoire une fois chargé.

Un tableau d'en-têtes de section de **40 octets**, un par section. Le nombre d'entrées est égal à `NumberOfSections` dans l'en-tête COFF.

```c
typedef struct _IMAGE_SECTION_HEADER {
    BYTE  Name[8];               // Nom de la section (ex. : ".text\0\0\0"), PAS terminé par un nul si 8 caractères
    union {
        DWORD PhysicalAddress;
        DWORD VirtualSize;       // Taille réellement utilisée en mémoire
    } Misc;
    DWORD VirtualAddress;        // RVA du début de la section en mémoire
    DWORD SizeOfRawData;         // Taille sur disque (multiple de FileAlignment)
    DWORD PointerToRawData;      // Offset fichier des données de la section
    DWORD PointerToRelocations;  // Inutilisé dans les exécutables (0)
    DWORD PointerToLinenumbers;  // Obsolète (0)
    WORD  NumberOfRelocations;   // Inutilisé dans les exécutables (0)
    WORD  NumberOfLinenumbers;   // Obsolète (0)
    DWORD Characteristics;       // Drapeaux de la section (lecture/écriture/exécution/etc.)
} IMAGE_SECTION_HEADER;
```

#### Drapeaux de Characteristics des sections :

| Drapeau | Valeur | Signification |
|------|-------|---------|
| `IMAGE_SCN_CNT_CODE` | `0x00000020` | Contient du code exécutable |
| `IMAGE_SCN_CNT_INITIALIZED_DATA` | `0x00000040` | Contient des données initialisées |
| `IMAGE_SCN_CNT_UNINITIALIZED_DATA` | `0x00000080` | Contient des données non initialisées (BSS) |
| `IMAGE_SCN_MEM_DISCARDABLE` | `0x02000000` | Peut être abandonnée après le chargement |
| `IMAGE_SCN_MEM_NOT_CACHED` | `0x04000000` | Ne peut pas être mise en cache |
| `IMAGE_SCN_MEM_NOT_PAGED` | `0x08000000` | Ne peut pas être paginée |
| `IMAGE_SCN_MEM_SHARED` | `0x10000000` | Partagée entre processus |
| `IMAGE_SCN_MEM_EXECUTE` | `0x20000000` | Exécutable (le processeur peut l'exécuter) |
| `IMAGE_SCN_MEM_READ` | `0x40000000` | Lisible |
| `IMAGE_SCN_MEM_WRITE` | `0x80000000` | Modifiable |

Combinaisons courantes :
- `.text` : `0x60000020` (code + exécution + lecture)
- `.data` : `0xC0000040` (données initialisées + lecture + écriture)
- `.rdata` : `0x40000040` (données initialisées + lecture seule)
- `.bss` : `0xC0000080` (données non initialisées + lecture + écriture)

---

### 7. Sections (données brutes)

> **En clair :** enfin, la cargaison elle-même — pas la paperasse sur la cargaison, mais les octets eux-mêmes : les instructions compilées, les chaînes de caractères de ton programme, l'icône, les informations de version.

Le contenu binaire réel de chaque section, stocké séquentiellement. Chaque section commence à son offset `PointerToRawData` (aligné sur `FileAlignment`).

---

## Octets magiques et marqueurs

> **En clair :** les « octets magiques » sont des séquences d'octets fixes et prévisibles qui servent d'empreinte digitale — un outil peut regarder les deux premiers octets de *n'importe quel* fichier et savoir aussitôt « celui-ci se prétend un exécutable Windows », sans se fier du tout à l'extension (une extension n'est qu'une étiquette que n'importe qui peut modifier ; les octets magiques, eux, sont inscrits dans le contenu).

Voici les séquences d'octets qui identifient et délimitent un fichier PE :

| Emplacement | Octets (hex) | ASCII | Rôle |
|----------|-------------|-------|---------|
| Offset 0x00 | `4D 5A` | `MZ` | Nombre magique DOS — début de tout fichier PE |
| Offset e_lfanew | `50 45 00 00` | `PE\0\0` | Signature PE |
| En-tête optionnel | `0B 01` | — | Nombre magique PE32 (32 bits) |
| En-tête optionnel | `0B 02` | — | Nombre magique PE32+ (64 bits) |
| En-tête optionnel | `07 01` | — | Nombre magique d'image ROM (rare) |
| En-tête Rich | `52 69 63 68` | `Rich` | Marqueur de fin de l'en-tête Rich non documenté (métadonnées de l'éditeur de liens MSVC) |
| En-tête Rich (début) | `44 61 6E 53` XOR clé | `DanS` (XORé) | Début de l'en-tête Rich (après décodage) |

### L'en-tête Rich (non documenté)

> **En clair :** un « easter egg » caché et non documenté que le compilateur de Microsoft glisse discrètement dans chaque binaire qu'il produit. C'est en pratique une empreinte de la version exacte du compilateur, de l'éditeur de liens et des outils de compilation qui ont touché le fichier — utile aux chercheurs en sécurité qui veulent savoir qui (ou quelle chaîne d'outils) a fabriqué un fichier suspect.

Les binaires compilés avec MSVC contiennent un bloc caché entre le stub DOS et la signature PE, appelé **en-tête Rich**. Il enregistre les outils de compilation et les versions utilisés pour construire le binaire — utile pour l'investigation numérique (forensique) et l'attribution de logiciels malveillants.

```
[En-tête DOS]
[Stub DOS]
[Marqueur DanS (obfusqué par XOR)] ← début de l'en-tête Rich
[entrées de version d'outils...]
[Marqueur Rich : "Rich" + clé XOR de 4 octets] ← fin de l'en-tête Rich
[Signature PE : "PE\0\0"]
```

---

## Référence des sections standard

> **En clair :** les sections sont les « pièces » du bâtiment. Chacune a un rôle constant dans pratiquement tous les programmes Windows jamais compilés — une fois que tu sais ce que veulent dire `.text` et `.rsrc` ici, tu les reconnaîtras dans n'importe quel fichier PE que tu ouvriras.

| Nom | Drapeaux typiques | Contenu |
|------|--------------|----------|
| `.text` | r-x | Code machine (fonctions compilées) |
| `.data` | rw- | Variables globales et statiques initialisées |
| `.rdata` | r-- | Données en lecture seule : chaînes littérales, constantes globales, infos de débogage |
| `.bss` | rw- | Globales non initialisées (remplies de zéros au chargement ; peut ne pas avoir de données brutes) |
| `.idata` | r-- | Tables d'import (quelles DLL/fonctions sont nécessaires) |
| `.edata` | r-- | Tables d'export (fonctions exposées par ce binaire, surtout pour les DLL) |
| `.reloc` | r-- | Table de relocalisation de base (pour l'ASLR) |
| `.rsrc` | r-- | Ressources : icônes, chaînes, boîtes de dialogue, manifestes, infos de version |
| `.pdata` | r-- | Données de gestion des exceptions (64 bits et RISC) |
| `.xdata` | r-- | Informations de déroulement de pile (unwind) pour les exceptions |
| `.tls` | rw- | Données de Thread Local Storage |
| `.debug` | r-- | Infos de débogage (généralement retirées des builds de production) |
| `.sxdata` | r-- | Table des gestionnaires d'exceptions sûrs |
| `UPX0`/`UPX1` | rwx | Sections du compresseur UPX |
| `.netsect` | r-- | Métadonnées .NET (en-tête CLR, bytecode MSIL/CIL) |

---

## Répertoires de données

> **En clair :** si la table des sections est la table des matières du *bâtiment*, les répertoires de données sont des post-it qui pointent vers des *pièces dans les pièces* — « les imports se trouvent exactement ici », « la signature numérique se trouve exactement là ». Seize emplacements fixes, dont la plupart sont utilisés ou laissés vides selon les besoins du programme.

L'en-tête optionnel se termine par 16 entrées de **répertoire de données** (*Data Directory*). Chacune fait 8 octets : une RVA de 4 octets et une taille de 4 octets. Elles pointent vers des structures importantes à l'intérieur des sections.

| Index | Nom | Description |
|-------|------|-------------|
| 0 | Export Table | `.edata` — fonctions exportées par cette image |
| 1 | Import Table | `.idata` — DLL et fonctions que cette image importe |
| 2 | Resource Table | `.rsrc` — ressources embarquées |
| 3 | Exception Table | `.pdata` — infos des gestionnaires d'exceptions |
| 4 | Certificate Table | Signature numérique Authenticode |
| 5 | Base Relocation Table | `.reloc` — adresses à corriger lors d'un changement de base |
| 6 | Debug | Informations de débogage |
| 7 | Architecture | Réservé |
| 8 | Global Ptr | RVA de la valeur du registre de pointeur global |
| 9 | TLS Table | Descripteurs de Thread Local Storage |
| 10 | Load Config Table | Configuration propre au chargeur (CFG, SEH sûr, etc.) |
| 11 | Bound Import | Adresses d'import pré-résolues (héritage) |
| 12 | IAT | Import Address Table (remplie par le chargeur) |
| 13 | Delay Import Descriptor | Imports chargés à la demande |
| 14 | CLR Runtime Header | Pointeur vers les métadonnées .NET / CLR |
| 15 | Reserved | Doit valoir zéro |

---

## Comment Windows charge un .exe

> **En clair :** c'est la version « histoire » de tout ce qui précède — ce qui se passe réellement, dans l'ordre, à l'instant où tu double-cliques sur un programme. Chaque étape ci-dessous utilise un morceau de la structure de fichier que tu viens de découvrir.

1. **Shell / CreateProcess** : un processus appelle `CreateProcess()` ou le shell le lance.
2. **Ouverture du fichier** : le noyau ouvre le fichier et le projette dans un objet section.
3. **Validation** : vérifie l'octet magique `MZ`, lit `e_lfanew`, vérifie la présence de `PE\0\0`.
4. **Projection de l'image** : projette chaque section en mémoire virtuelle à `ImageBase` (ou à une nouvelle adresse aléatoire si l'ASLR est activé).
5. **Application des relocalisations** : si l'image a été chargée à une adresse différente de `ImageBase`, la section `.reloc` sert à corriger les adresses absolues.
6. **Résolution des imports** : le chargeur lit `.idata`, charge les DLL nécessaires et remplit l'IAT (Import Address Table) avec les adresses réelles des fonctions.
7. **Callbacks TLS** : les éventuels initialiseurs de Thread Local Storage sont appelés.
8. **Point d'entrée** : l'exécution saute à `AddressOfEntryPoint`. Pour les programmes MSVC, c'est le code de démarrage de la CRT, qui finit par appeler ton `main()` ou `WinMain()`.

---

## Créer un .exe

> **En clair :** cette section est un petit livre de recettes — une recette par langage de programmation — qui montre les commandes minimales pour transformer du code source en un `.exe` fonctionnel. Si tu n'as jamais rien compilé, en choisir une et l'exécuter est le moyen le plus rapide de voir tout ce qui précède se transformer en un vrai fichier que tu peux ouvrir dans un éditeur hexadécimal.

### Depuis C/C++ (MSVC)

```cmd
:: Compiler une application console simple
cl.exe hello.c /Fe:hello.exe

:: Avec optimisation et sans infos de débogage
cl.exe /O2 /GL hello.c /link /LTCG /OUT:hello.exe

:: Application graphique (sans fenêtre de console)
cl.exe hello.c /link /SUBSYSTEM:WINDOWS /OUT:hello.exe
```

Avec CMake :
```cmake
cmake_minimum_required(VERSION 3.20)
project(MyApp)
add_executable(MyApp main.cpp)
set_target_properties(MyApp PROPERTIES WIN32_EXECUTABLE TRUE) # mode graphique
```

### Depuis C/C++ (MinGW/GCC)

```bash
# 32 bits
i686-w64-mingw32-gcc hello.c -o hello.exe

# 64 bits
x86_64-w64-mingw32-gcc hello.c -o hello.exe

# Liaison statique (autonome, sans DLL MinGW nécessaires)
x86_64-w64-mingw32-gcc hello.c -o hello.exe -static

# Sous-système graphique
x86_64-w64-mingw32-gcc hello.c -o hello.exe -mwindows
```

### Depuis C# (.NET)

```csharp
// hello.cs
using System;
class Program {
    static void Main() => Console.WriteLine("Hello, World!");
}
```

```cmd
:: .NET Framework (csc.exe)
csc.exe hello.cs /out:hello.exe

:: .NET 6+ (CLI dotnet)
dotnet new console -n MyApp
dotnet publish -c Release -r win-x64 --self-contained true
```

Le `.exe` obtenu possède un en-tête CLR dans le PE et contient du bytecode CIL, pas du code machine natif (sauf compilation avec Native AOT).

### Depuis Rust

```toml
# Cargo.toml
[package]
name = "hello"
version = "0.1.0"
edition = "2021"
```

```bash
cargo build --release --target x86_64-pc-windows-gnu
# Sortie : target/x86_64-pc-windows-gnu/release/hello.exe
```

### Depuis Python (PyInstaller)

```bash
pip install pyinstaller

# Exe en un seul fichier (embarque l'interpréteur + toutes les dépendances)
pyinstaller --onefile hello.py

# Application graphique (sans fenêtre de console)
pyinstaller --onefile --windowed hello.py

# Icône personnalisée
pyinstaller --onefile --icon=app.ico hello.py
```

La sortie se trouve dans `dist/hello.exe`. À noter : cela embarque un interpréteur Python complet, donc les fichiers peuvent peser de 20 à 60 Mo — bien plus que les exemples C/C++ ci-dessus, car tout un environnement d'exécution de langage voyage à l'intérieur des sections du PE.

### Depuis l'assembleur (NASM)

Pour un exécutable console Windows 64 bits minimal, fait à la main — c'est à peu près ce qui se rapproche le plus de l'écriture des octets à la main :

```nasm
; hello.asm
bits 64
default rel

section .data
    msg db "Hello, World!", 13, 10, 0
    msg_len equ $ - msg

section .text
    global main
    extern ExitProcess
    extern GetStdHandle
    extern WriteConsoleA

main:
    ; Récupérer le handle de STDOUT
    mov rcx, -11          ; STD_OUTPUT_HANDLE
    call GetStdHandle
    mov rbx, rax

    ; Écrire dans la console
    mov rcx, rbx
    lea rdx, [msg]
    mov r8, msg_len
    lea r9, [rsp+32]
    push 0
    sub rsp, 40
    call WriteConsoleA
    add rsp, 48

    ; Quitter
    xor rcx, rcx
    call ExitProcess
```

```bash
nasm -f win64 hello.asm -o hello.obj
link hello.obj kernel32.lib /entry:main /subsystem:console /out:hello.exe
```

---

## Inspecter un .exe

> **En clair :** tu n'as pas besoin d'écrire du code pour regarder à l'intérieur de ces fichiers — il existe des outils graphiques gratuits qui te montrent simplement les en-têtes et les sections sous forme d'arbre navigable, sans calcul hexadécimal.

### Outils intégrés à Windows

```cmd
:: Vérifier qu'un fichier est un PE valide et afficher le sous-système
dumpbin /headers hello.exe

:: Lister les DLL et fonctions importées
dumpbin /imports hello.exe

:: Lister les fonctions exportées (pour les DLL)
dumpbin /exports hello.dll

:: Désassembler la section .text
dumpbin /disasm hello.exe
```

### Outils tiers gratuits

| Outil | Usage |
|------|-----|
| [PE-bear](https://github.com/hasherezade/pe-bear) | Visionneuse/éditeur PE graphique |
| [CFF Explorer](https://ntcore.com/?page_id=388) | Éditeur PE complet avec vue hexadécimale |
| [x64dbg](https://x64dbg.com/) | Débogueur dynamique |
| [Ghidra](https://ghidra-sre.org/) | Décompilateur complet (NSA, gratuit) |
| [IDA Free](https://hex-rays.com/ida-free/) | Désassembleur / décompilateur |
| [Detect-It-Easy (DiE)](https://github.com/horsicq/Detect-It-Easy) | Détection de compresseurs/compilateurs |
| [pestudio](https://www.winitor.com/) | Analyse statique de logiciels malveillants |

### Python (avec `pefile`)

```python
import pefile

pe = pefile.PE("hello.exe")

print(f"Machine :       {hex(pe.FILE_HEADER.Machine)}")
print(f"Sous-système :  {pe.OPTIONAL_HEADER.Subsystem}")
print(f"Point d'entrée : {hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint)}")
print(f"Base de l'image : {hex(pe.OPTIONAL_HEADER.ImageBase)}")
print(f"Sections :      {pe.FILE_HEADER.NumberOfSections}")

for section in pe.sections:
    name = section.Name.decode(errors='replace').strip('\x00')
    print(f"  {name:10s} VA={hex(section.VirtualAddress)} Taille={section.Misc_VirtualSize}")

print("\nImports :")
for entry in pe.DIRECTORY_ENTRY_IMPORT:
    print(f"  {entry.dll.decode()}")
    for imp in entry.imports:
        print(f"    {imp.name.decode() if imp.name else f'ord({imp.ordinal})'}")
```

---

## Variantes du PE

> **En clair :** tous les `.exe` ne sont pas « simplement » du code machine compilé qui attend de s'exécuter. Certains enveloppent un environnement d'exécution entièrement différent (.NET), et d'autres enveloppent *un autre fichier PE caché* (les compresseurs, souvent utilisés à la fois par des logiciels légitimes et des logiciels malveillants pour réduire ou cacher la vraie charge utile).

### Exécutables .NET / CLR

Le code managé (C#, VB.NET, F#) produit un PE avec :
- Un point d'entrée natif valide qui appelle `mscoree.dll!_CorExeMain`
- L'entrée 14 du répertoire de données pointant vers l'**en-tête CLR** (`IMAGE_COR20_HEADER`)
- La section `.text` contenant du bytecode CIL (Common Intermediate Language)
- Un flux de métadonnées décrivant les types, les méthodes et les assemblages

### Exécutables compressés (packed)

Des compresseurs comme UPX, MPRESS ou Themida enveloppent le PE d'origine dans un autre PE :
- Le PE enveloppe contient un stub de décompression dans `.text`
- Le PE d'origine est stocké compressé dans une section de données (par ex. `UPX1`)
- À l'exécution, le stub décompresse l'original en mémoire et y saute

C'est exactement la raison pour laquelle « packed » est un mot qui rend les antivirus nerveux — un compresseur cache la vraie section `.text` à l'analyse statique jusqu'au moment où le programme s'exécute réellement.

### 32 bits contre 64 bits

| Caractéristique | PE32 | PE32+ |
|---------|------|-------|
| Nombre magique | `0x010B` | `0x020B` |
| Taille d'un pointeur | 4 octets | 8 octets |
| `ImageBase` | DWORD (4 octets) | ULONGLONG (8 octets) |
| Tailles de pile/tas | DWORD | ULONGLONG |
| `ImageBase` par défaut | `0x00400000` | `0x0000000140000000` |
| Champ `BaseOfData` | Présent | **Absent** |

---

## Fonctionnalités de sécurité

> **En clair :** les `.exe` modernes embarquent des fonctions défensives optionnelles — comme une maison qui peut avoir un système d'alarme, des fenêtres renforcées et une serrure à pêne dormant : aucun de ces éléments n'est physiquement nécessaire pour que la maison existe, mais tous la rendent plus difficile à cambrioler.

| Fonctionnalité | Comment l'activer | Ce qu'elle fait |
|---------|--------------|--------------|
| **ASLR** | Option d'édition de liens `/DYNAMICBASE` | Rend l'adresse de chargement aléatoire ; nécessite la section `.reloc` |
| **DEP/NX** | Option d'édition de liens `/NXCOMPAT` | Marque la pile/le tas comme non exécutables |
| **SafeSEH** | `/SAFESEH` (x86 uniquement) | Établit une liste blanche des gestionnaires d'exceptions valides |
| **CFG** | `/guard:cf` | Control Flow Guard : limite les cibles valides des appels indirects |
| **CET** | `/CETCOMPAT` | Protection par pile fantôme (shadow stack) Intel CET |
| **Authenticode** | `signtool.exe` | Intègre une signature numérique dans la Certificate Table |
| **Manifeste** | Ressource embarquée | Déclare le niveau UAC, la prise en charge du DPI, la compatibilité avec l'OS |

Cette dernière ligne, **Manifeste**, est la raison technique pour laquelle Windows affiche « Voulez-vous autoriser cette application à apporter des modifications à votre appareil ? » — la ressource de manifeste déclare littéralement si le programme a besoin des droits d'administrateur.

---

## Pièges courants et remarques

> **En clair :** ce sont les embûches qui font trébucher tout le monde la première fois qu'on essaie d'analyser ou de construire un fichier PE à la main — le genre de chose qu'on n'apprend qu'après le premier plantage d'un analyseur sur un fichier réel.

- **RVA contre offset fichier** : une RVA (Relative Virtual Address) est un décalage par rapport à `ImageBase` en mémoire. L'offset fichier est un décalage brut en octets dans le fichier `.exe` sur le disque. Ils diffèrent parce que les sections sont alignées différemment en mémoire et sur le disque. Utilise la table des sections pour passer de l'un à l'autre.

- **Alignement** : toutes les sections doivent commencer sur des multiples de `FileAlignment` (disque) et de `SectionAlignment` (mémoire). Un mauvais alignement pousse le chargeur à rejeter le fichier.

- **`SizeOfImage`** doit être un multiple de `SectionAlignment` et couvrir toutes les sections.

- **TimeDateStamp** : les builds reproductibles le mettent souvent à zéro. Les outils d'analyse de logiciels malveillants l'utilisent pour l'attribution (même s'il peut être falsifié).

- **Checksum** : ne doit être valide que pour les pilotes du noyau (`.sys`). Les exécutables en mode utilisateur peuvent avoir une somme de contrôle nulle.

- **Exports redirigés (forward exports)** : une DLL peut rediriger un export vers une autre DLL. L'entrée de la table d'export contient alors une chaîne comme `"NTDLL.RtlAllocateHeap"` au lieu d'une RVA.

- **Imports par ordinal** : un import peut référencer une fonction par son numéro d'ordinal plutôt que par son nom. Le champ `Name` vaudra NULL dans `pefile`.

---

## Glossaire

> **En clair :** un aide-mémoire tout-en-un. Si tu oublies ce que signifie un acronyme plus haut, il est défini ici.

| Terme | Définition |
|------|------------|
| **RVA** | Relative Virtual Address — décalage par rapport à `ImageBase` en mémoire |
| **VA** | Virtual Address — adresse absolue en mémoire (`ImageBase + RVA`) |
| **IAT** | Import Address Table — remplie par le chargeur avec les adresses réelles des fonctions |
| **ILT / INT** | Import Lookup Table / Name Table — noms d'import d'origine avant correction |
| **PE** | Portable Executable — le format de fichier |
| **COFF** | Common Object File Format — ancêtre du PE |
| **MZ** | Octets magiques DOS, nommés d'après Mark Zbikowski |
| **Subsystem** | Indique si l'exécutable est une application graphique, console ou un pilote |
| **ASLR** | Address Space Layout Randomization — randomisation de l'agencement de l'espace d'adressage |
| **DEP/NX** | Data Execution Prevention / No-Execute (protection matérielle de la pile) |
| **CFG** | Control Flow Guard |
| **CIL / MSIL** | Common (Intermediate) Language — bytecode .NET |
| **CLR** | Common Language Runtime — le moteur d'exécution .NET |
| **UPX** | Universal Packer for eXecutables — compresseur open source populaire |
| **Stub** | Le programme DOS 16 bits au début de chaque .exe |
| **En-tête Rich** | Bloc de métadonnées non documenté de l'éditeur de liens MSVC |
| **TLS** | Thread Local Storage — variables propres à chaque thread |
| **Section** | Un bloc de mémoire nommé dans le PE (`.text`, `.data`, etc.) |
| **FileAlignment** | Granularité des données de section sur disque (généralement 512 octets) |
| **SectionAlignment** | Granularité des données de section en mémoire (généralement 4096 octets = 1 page) |

---

*Ce document couvre le format PE tel que défini dans la spécification Microsoft PE/COFF et implémenté de Windows NT à Windows 11. Pour la spécification officielle, voir [Microsoft PE and COFF Specification](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format) (en anglais).*
