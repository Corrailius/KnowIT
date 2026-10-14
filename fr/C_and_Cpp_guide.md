# C & C++ — De ton premier programme à l'aisance

> **Pour qui ?** Pour tout le monde, de la personne qui écrit sa première ligne de code aux développeurs qui livrent du C depuis des décennies. Chaque section s'ouvre sur un encadré **« En clair »** qui explique l'idée sans jargon. Les notes **« Pour aller plus loin »** apportent les détails précis (comportement indéfini, différences entre normes, particularités des plateformes) qui intéressent les développeurs expérimentés. Survole ce que tu connais déjà, ralentis là où tu en as besoin.

> **À propos du code :** tous les programmes complets de ce guide (ceux qui contiennent un `main`) ont été compilés avec `-Wall -Wextra -Wpedantic` puis exécutés, avec AddressSanitizer et UndefinedBehaviorSanitizer activés. Les exemples C utilisent `-std=c17`, les exemples C++ utilisent `-std=c++20`. Deux exemples déclenchent volontairement un avertissement du compilateur, et le texte le précise. Quand le résultat dépend de ta plateforme, le texte le précise aussi.

---

## Table des matières

1. [Que sont le C et le C++](#1-que-sont-le-c-et-le-c)
2. [Du code source à l'exécutable](#2-du-code-source-à-lexécutable)
3. [Ton premier programme](#3-ton-premier-programme)
4. [Les briques de base](#4-les-briques-de-base)
5. [Les fonctions](#5-les-fonctions)
6. [Mémoire et pointeurs — le cœur du C](#6-mémoire-et-pointeurs--le-cœur-du-c)
7. [Tableaux et chaînes de caractères](#7-tableaux-et-chaînes-de-caractères)
8. [Structures, énumérations et unions](#8-structures-énumérations-et-unions)
9. [Mémoire dynamique](#9-mémoire-dynamique)
10. [En-têtes, fichiers multiples et outils de build](#10-en-têtes-fichiers-multiples-et-outils-de-build)
11. [L'essentiel du C++ — ce qui change](#11-lessentiel-du-c--ce-qui-change)
12. [C ou C++ : côte à côte](#12-c-ou-c--côte-à-côte)
13. [Comportement indéfini et pièges classiques](#13-comportement-indéfini-et-pièges-classiques)
14. [Débogage et outils](#14-débogage-et-outils)
15. [Normes du langage — laquelle choisir](#15-normes-du-langage--laquelle-choisir)
16. [Référence rapide et glossaire](#16-référence-rapide-et-glossaire)

---

## 1. Que sont le C et le C++

> **En clair :** le C est un langage petit et rapide, très proche du matériel : c'est toi qui décides exactement comment la mémoire est utilisée, et presque rien ne se passe dans ton dos. Le C++ est né comme du « C avec des classes » avant de devenir un langage bien plus vaste, doté d'outils pour gérer la complexité (classes, templates, riche bibliothèque standard), tout en te laissant redescendre au niveau du C quand tu le veux. Aujourd'hui, ce sont deux langages distincts qui partagent un noyau commun.

| | **C** | **C++** |
|---|---|---|
| Créé par | Dennis Ritchie, Bell Labs (début des années 1970) | Bjarne Stroustrup, Bell Labs (démarré en 1979 sous le nom « C with Classes », rebaptisé C++ en 1983) |
| Style | Procédural | Multi-paradigme : procédural, orienté objet, générique, style fonctionnel |
| Gestion de la mémoire | Manuelle (`malloc` / `free`) | Manuelle **ou** automatique via RAII et pointeurs intelligents |
| Bibliothèque standard | Réduite (`stdio.h`, `string.h`, `stdlib.h`, ...) | Vaste (conteneurs, algorithmes, chaînes, threads, ...) |
| Usages typiques | Systèmes d'exploitation, pilotes, embarqué, environnements d'exécution de langages, bibliothèques à ABI stable | Jeux, navigateurs, bases de données, applications de bureau, systèmes à hautes performances et temps réel |

### Le C++ n'est pas un sur-ensemble strict du C

On lit souvent que « le C++, c'est du C avec des extras ». C'est *globalement* vrai, mais pas toujours. Certains codes C valides ne compilent pas en C++, et d'autres compilent dans les deux langages mais **se comportent différemment** :

| Construction | En C | En C++ |
|---|---|---|
| `int *p = malloc(4);` | Valide (`void *` se convertit implicitement) | **Erreur** : il faut un cast (ou, mieux, `new` / un conteneur) |
| `sizeof('a')` | `sizeof(int)`, généralement 4 (un littéral caractère a le type `int`) | `1` (un littéral caractère a le type `char`) |
| Appeler une fonction non déclarée | Invalide depuis C99 (beaucoup de compilateurs ne font que prévenir ; GCC 14+ refuse) | Toujours une erreur |
| Tableaux de taille variable `int a[n];` | Valides depuis C99 (optionnels depuis C11) | Pas du C++ standard (certains compilateurs les acceptent en extension) |
| `class`, `new`, `delete`, `template` comme noms de variables | Autorisé (ce ne sont pas des mots-clés en C) | Erreur (mots réservés) |
| Pointeurs `restrict` | Pris en charge (C99) | Non standard (les compilateurs proposent `__restrict`) |

> **Pour aller plus loin :** à cause de différences de ce genre, les deux langages ont des comités de normalisation distincts (groupes de travail WG14 pour le C et WG21 pour le C++, au sein d'ISO/IEC JTC 1/SC 22). Mélanger les deux dans un même projet est courant et fonctionne bien, à condition de compiler chaque fichier avec le bon compilateur ou le bon mode et d'utiliser `extern "C"` à la frontière (voir la [section 10](#10-en-têtes-fichiers-multiples-et-outils-de-build)).

### Lequel apprendre

- **Apprends le C d'abord** si tu veux comprendre comment les ordinateurs fonctionnent vraiment : la mémoire, les pointeurs et le modèle de la machine y sont à nu. Tout ce que tu apprends se transpose au C++.
- **Apprends le C++** si tu veux construire de grosses applications rapides en limitant le risque de bugs mémoire. Le C++ moderne (C++11 et au-delà) n'a plus grand-chose à voir avec le C++ des années 1990, et il est bien plus sûr.
- **Beaucoup de développeurs apprennent les deux**, et ce guide est organisé pour ça : les chapitres C viennent d'abord, puis la [section 11](#11-lessentiel-du-c--ce-qui-change) montre ce que le C++ change.

---

## 2. Du code source à l'exécutable

> **En clair :** un ordinateur ne sait pas exécuter le texte que tu tapes. Un ensemble d'outils appelé **chaîne de compilation** (*toolchain*) traduit tes fichiers sources en code machine en plusieurs étapes. Connaître ces étapes rend les messages d'erreur bien moins mystérieux : « erreur de compilation » et « erreur d'édition de liens » sont deux échecs différents, à deux étapes différentes.

```
 main.c ──► Préprocesseur ──► Compilateur ──► Assembleur ──► main.o ──┐
 util.c ──► Préprocesseur ──► Compilateur ──► Assembleur ──► util.o ──┼─► Éditeur de liens ──► programme
                                          bibliothèques (.a/.lib/.so/.dll) ───┘
```

| Étape | Ce qui se passe | Option GCC pour s'arrêter ici | Résultat |
|---|---|---|---|
| 1. **Préprocesseur** | Traite les lignes commençant par `#` : insère les fichiers `#include`, développe les macros `#define`, supprime les commentaires | `-E` | `.i` (C) / `.ii` (C++) |
| 2. **Compilateur** | Vérifie la syntaxe et les types, traduit le C/C++ en assembleur | `-S` | `.s` |
| 3. **Assembleur** | Transforme l'assembleur en code machine dans un *fichier objet* | `-c` | `.o` / `.obj` |
| 4. **Éditeur de liens** (*linker*) | Assemble tes fichiers objet et les bibliothèques, résout les références de fonctions entre eux | (par défaut) | exécutable |

Tu peux lancer chaque étape à la main pour voir ce qu'elle produit :

```bash
gcc -E main.c -o main.i     # 1. préprocesser  → source développée
gcc -S main.i -o main.s     # 2. compiler      → assembleur
gcc -c main.s -o main.o     # 3. assembler     → fichier objet
gcc main.o -o main          # 4. lier          → exécutable
```

Sous Linux, le résultat est un fichier **ELF** ; sous Windows, c'est un fichier **PE**, le format disséqué dans [Le format Windows .exe](The%20Windows%20Executable%20%28.exe%29%20Format.md). Les noms de sections que tu rencontreras dans la [section 6](#6-mémoire-et-pointeurs--le-cœur-du-c) (`.text`, `.data`, `.rdata`, `.bss`) sont exactement ceux qui y sont décrits.

### Les compilateurs courants

| Compilateur | Commandes | Plateforme |
|---|---|---|
| **GCC** | `gcc` (C), `g++` (C++) | Linux, macOS (via Homebrew), Windows (MinGW / MSYS2) |
| **Clang / LLVM** | `clang`, `clang++` | Linux, macOS, Windows |
| **MSVC** | `cl.exe` | Windows (Visual Studio / Build Tools) |

> **Pour aller plus loin :** chaque fichier `.c` / `.cpp`, une fois préprocessé, s'appelle une **unité de traduction** (*translation unit*), et il est compilé de façon totalement indépendante des autres. C'est pourquoi un en-tête doit *déclarer* ce qu'un fichier utilise, et pourquoi c'est l'éditeur de liens, et non le compilateur, qui signale `undefined reference to 'foo'` : ce n'est qu'à l'édition de liens que quelqu'un vérifie que `foo` existe quelque part. En C++, la **règle de définition unique (ODR)** impose que chaque fonction et chaque variable non `inline` soit définie exactement une fois dans tout le programme.

---

## 3. Ton premier programme

**En C :**

```c
#include <stdio.h>

int main(void)
{
    printf("Hello, World!\n");
    return 0;
}
```

**En C++ :**

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, World!\n";
    return 0;
}
```

**Compiler et exécuter :**

```bash
gcc -std=c17   -Wall -Wextra -Wpedantic hello.c   -o hello    # C
g++ -std=c++20 -Wall -Wextra -Wpedantic hello.cpp -o hello    # C++

./hello          # Linux / macOS
hello.exe        # Windows
```

> **✔ Astuce :** compile toujours avec `-Wall -Wextra -Wpedantic` (ou `/W4` avec MSVC). Les avertissements sont des rapports de bugs gratuits, et la plupart des bugs réels en C/C++ auraient été signalés par l'un d'eux.

| Élément | Signification |
|---|---|
| `#include <stdio.h>` | Insère les déclarations des fonctions d'entrée/sortie standard (`printf`, ...). `<...>` cherche dans les en-têtes système. |
| `int main(void)` | Le point d'entrée. L'`int` renvoyé est le **code de sortie** (`0` = succès). |
| `printf(...)` / `std::cout <<` | Affiche du texte. `\n` est un saut de ligne. |
| `return 0;` | Code de sortie. Dans `main` uniquement, cette ligne est facultative depuis C99 et C++98 : atteindre l'accolade fermante renvoie `0`. |
| `std::` | L'**espace de noms** de la bibliothèque standard C++. Il évite les collisions de noms. |

> **Pour aller plus loin :** en C avant C23, `int main()` signifie « paramètres non spécifiés » alors que `int main(void)` signifie « aucun paramètre » : `(void)` est donc la forme correcte et explicite. C23 rend les deux équivalentes, comme le C++ l'a toujours fait.

### Lire les arguments de la ligne de commande

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("nom du programme : %s\n", argv[0]);
    for (int i = 1; i < argc; i++)
        printf("argument %d : %s\n", i, argv[i]);
    return 0;
}
```

`argc` est le nombre d'arguments (nom du programme compris), `argv` est un tableau de chaînes C.

---

## 4. Les briques de base

### 4.1 Variables et types fondamentaux

> **En clair :** une variable est une boîte nommée en mémoire. Son **type** indique au compilateur la taille de la boîte et la façon d'interpréter son contenu : nombre entier, nombre décimal, caractère...

| Type | Taille typique | Garanti par la norme |
|---|---|---|
| `char` | 1 octet | Exactement 1 octet par définition (`sizeof(char) == 1`), au moins 8 bits |
| `short` | 2 octets | Au moins 16 bits |
| `int` | 4 octets | Au moins 16 bits |
| `long` | 4 ou 8 octets | Au moins 32 bits (**4 sous Windows 64 bits, 8 sous Linux/macOS 64 bits**) |
| `long long` | 8 octets | Au moins 64 bits |
| `float` | 4 octets | Généralement IEEE 754 simple précision |
| `double` | 8 octets | Généralement IEEE 754 double précision |
| `bool` | 1 octet | `<stdbool.h>` en C (mot-clé en C++ et en C23) |

Chaque type entier existe aussi en variante `unsigned` (pas de valeurs négatives, plage positive doublée).

```c
#include <stdio.h>
#include <stdint.h>

int main(void)
{
    printf("char      : %zu\n", sizeof(char));
    printf("short     : %zu\n", sizeof(short));
    printf("int       : %zu\n", sizeof(int));
    printf("long      : %zu\n", sizeof(long));
    printf("long long : %zu\n", sizeof(long long));
    printf("float     : %zu\n", sizeof(float));
    printf("double    : %zu\n", sizeof(double));
    printf("void *    : %zu\n", sizeof(void *));
    printf("int32_t   : %zu\n", sizeof(int32_t));
    return 0;
}
```

Lance-le sur ta machine : les tailles de `long` et de `void *` révèlent ta plateforme.

> **✔ Astuce :** quand la taille exacte compte (formats de fichiers, protocoles réseau, registres matériels), utilise les **types à largeur fixe** de `<stdint.h>` / `<cstdint>` : `int8_t`, `uint8_t`, `int16_t`, `uint16_t`, `int32_t`, `uint32_t`, `int64_t`, `uint64_t`. Pour les tailles et les indices de tableaux, utilise `size_t` (un type non signé, affiché avec `%zu`).

> **Pour aller plus loin :**
> - **Le dépassement d'un entier signé est un comportement indéfini** ; **le dépassement d'un entier non signé boucle** modulo 2<sup>N</sup> (parfaitement défini). Le compilateur peut supposer qu'un dépassement signé n'arrive jamais et optimiser en conséquence.
> - Savoir si un `char` simple est signé ou non est **défini par l'implémentation** (signé sur x86 Linux, non signé sur ARM Linux).
> - Quand un opérande signé et un opérande non signé se rencontrent, le signé est converti en non signé. Cela surprend presque tout le monde, une fois :

```c
#include <stdio.h>

int main(void)
{
    int a = -1;
    unsigned int b = 1;

    if (a < b)
        printf("-1 < 1\n");
    else
        printf("surprise : -1 n'est PAS inférieur à 1u\n");   /* c'est cette branche qui s'exécute */
    return 0;
}
```

`-1` devient `4294967295` avant la comparaison. GCC et Clang le signalent avec `-Wextra` (`-Wsign-compare`), une raison de plus d'activer les avertissements.

### 4.2 Opérateurs

| Catégorie | Opérateurs | Remarques |
|---|---|---|
| Arithmétique | `+ - * / %` | Le `/` entier **tronque vers zéro** (`7 / 2 == 3`, `-7 / 2 == -3`). `%` est le reste. |
| Comparaison | `== != < > <= >=` | Donnent `1` (vrai) ou `0` (faux) en C, un `bool` en C++. |
| Logique | `&& \|\| !` | **Évaluation paresseuse** : dans `a && b`, `b` n'est pas évalué si `a` est faux. |
| Bit à bit | `& \| ^ ~ << >>` | Travaillent sur les bits individuels. |
| Affectation | `= += -= *= /= %= &= \|= ^= <<= >>=` | `x += 2` signifie `x = x + 2`. |
| Incrémentation | `++ --` | `i++` renvoie l'ancienne valeur, `++i` la nouvelle. |
| Autres | `?:` `sizeof` `,` `&` `*` `->` `.` | Ternaire, taille en octets, adresse de, déréférencement, accès aux membres. |

> **⚠ Piège classique :** `=` affecte, `==` compare. Écrire `if (x = 5)` affecte 5 à `x` et la condition est toujours vraie. Les compilateurs le signalent avec `-Wall`.

Les opérateurs bit à bit servent à stocker des drapeaux de façon compacte. Les registres matériels, les formats de fichiers et les champs `Characteristics` du PE dans le [guide `.exe`](The%20Windows%20Executable%20%28.exe%29%20Format.md) utilisent tous cette technique :

```c
#include <stdio.h>

#define FLAG_READ  (1u << 0)   /* 0001 */
#define FLAG_WRITE (1u << 1)   /* 0010 */
#define FLAG_EXEC  (1u << 2)   /* 0100 */

int main(void)
{
    unsigned int perms = FLAG_READ | FLAG_WRITE;   /* active deux drapeaux  */
    perms |= FLAG_EXEC;                            /* ajoute un drapeau     */
    perms &= ~FLAG_WRITE;                          /* retire un drapeau     */

    if (perms & FLAG_EXEC)                         /* teste un drapeau      */
        printf("exécution autorisée\n");
    printf("perms = %u\n", perms);                 /* 5 (binaire 0101)      */
    return 0;
}
```

> **✔ Astuce :** ne mémorise pas le tableau complet des priorités. Ajoute des parenthèses dès qu'une expression mélange des familles d'opérateurs : `(a & MASK) == 0`, et non `a & MASK == 0` (qui s'interprète comme `a & (MASK == 0)`).

### 4.3 Structures de contrôle

```c
#include <stdio.h>

int main(void)
{
    for (int i = 1; i <= 15; i++) {          /* for : boucle comptée */
        if (i % 15 == 0)      printf("FizzBuzz\n");
        else if (i % 3 == 0)  printf("Fizz\n");
        else if (i % 5 == 0)  printf("Buzz\n");
        else                  printf("%d\n", i);
    }

    int n = 3;
    while (n > 0) {                          /* while : test d'abord */
        printf("%d... ", n);
        n--;
    }
    printf("décollage\n");

    do {                                     /* do-while : le corps s'exécute au moins une fois */
        n++;
    } while (n < 3);
    printf("n = %d\n", n);
    return 0;
}
```

`switch` choisit entre des valeurs constantes. **Chaque `case` enchaîne sur le suivant sauf si tu mets un `break` :**

```c
switch (command) {
    case 'a':
        add_item();
        break;                 /* sans cela, l'exécution continuerait dans 'd' */
    case 'd':
        delete_item();
        break;
    default:
        puts("commande inconnue");
}
```

`break` quitte la boucle ou le `switch` le plus interne, `continue` passe à l'itération suivante. `goto` existe et a quelques usages légitimes (comme sauter vers un unique bloc de nettoyage en fin de fonction C), mais il est rare.

### 4.4 Entrées et sorties

| Format `printf` | Affiche | Type de l'argument |
|---|---|---|
| `%d` / `%i` | décimal signé | `int` |
| `%u` | décimal non signé | `unsigned int` |
| `%ld` / `%lld` | décimal signé | `long` / `long long` |
| `%zu` | décimal non signé | `size_t` |
| `%f` | nombre décimal | `double` (un `float` est promu) |
| `%c` | un caractère | `int` / `char` |
| `%s` | chaîne | `const char *` |
| `%x` / `%X` | hexadécimal | `unsigned int` |
| `%p` | pointeur | `void *` (à convertir explicitement) |
| `%%` | un `%` littéral | — |

> **⚠ Important :** `printf` ne peut pas vérifier à l'exécution que ton format correspond à tes arguments. Une incohérence comme `%d` avec un `double` est un comportement indéfini. `-Wall` fait vérifier par GCC et Clang les chaînes de format littérales à ta place.

Lire une saisie en C : préfère `fgets` à `scanf("%s", ...)`, qui n'a aucune limite de longueur et constitue une source classique de dépassement de tampon.

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char name[32];

    printf("Votre nom : ");
    if (fgets(name, sizeof name, stdin) != NULL) {
        name[strcspn(name, "\n")] = '\0';        /* retire le saut de ligne final */
        printf("Bonjour, %s !\n", name);
    }
    return 0;
}
```

L'équivalent en C++ n'exige aucune gestion de taille :

```cpp
#include <iostream>
#include <string>

int main()
{
    std::string name;
    std::cout << "Votre nom : ";
    std::getline(std::cin, name);
    std::cout << "Bonjour, " << name << " !\n";
}
```

---

## 5. Les fonctions

> **En clair :** une fonction est un bloc de code nommé et réutilisable. Tu lui donnes des entrées (les **paramètres**), elle peut renvoyer une sortie (la **valeur de retour**). Découper un programme en petites fonctions est le moyen le plus efficace de le garder compréhensible.

```c
int add(int a, int b);        /* déclaration (prototype) : promet que la fonction existe */

int add(int a, int b)         /* définition : le code lui-même */
{
    return a + b;
}
```

Une déclaration indique au compilateur *comment appeler* une fonction ; la définition, *c'est* la fonction. Une fonction doit être déclarée avant d'être utilisée.

### Les paramètres sont des copies

Le C passe toujours les arguments **par valeur** : la fonction travaille sur une copie. Pour qu'une fonction modifie la variable de l'appelant, passe son **adresse** (un pointeur, [section 6](#6-mémoire-et-pointeurs--le-cœur-du-c)) :

```c
#include <stdio.h>

void swap_wrong(int a, int b) { int t = a; a = b; b = t; }     /* n'échange que les copies   */
void swap_ok(int *a, int *b)  { int t = *a; *a = *b; *b = t; } /* échange les originaux      */

int main(void)
{
    int x = 1, y = 2;

    swap_wrong(x, y);
    printf("après swap_wrong : x=%d y=%d\n", x, y);   /* x=1 y=2 */

    swap_ok(&x, &y);
    printf("après swap_ok :    x=%d y=%d\n", x, y);   /* x=2 y=1 */
    return 0;
}
```

Le C++ ajoute les **références**, qui font le même travail avec une syntaxe plus propre :

```cpp
#include <iostream>

void swap_ref(int &a, int &b) { int t = a; a = b; b = t; }

int main()
{
    int x = 1, y = 2;
    swap_ref(x, y);                       // pas de & à l'appel
    std::cout << x << ' ' << y << '\n';   // 2 1
}
```

Le C++ gère aussi la **surcharge de fonctions** (plusieurs fonctions de même nom avec des paramètres différents) et les **arguments par défaut**. Le C n'a ni l'une ni l'autre.

### Pointeurs de fonctions

Une fonction a aussi une adresse, et tu peux la transmettre. L'usage classique est le rappel (*callback*), comme la fonction de comparaison de `qsort` :

```c
#include <stdio.h>
#include <stdlib.h>

static int cmp_int(const void *pa, const void *pb)
{
    int a = *(const int *)pa;
    int b = *(const int *)pb;
    return (a > b) - (a < b);          /* -1, 0 ou 1 ; ne déborde jamais, contrairement à a - b */
}

int main(void)
{
    int v[] = {42, 7, 19, -3, 7};
    size_t n = sizeof v / sizeof v[0];

    qsort(v, n, sizeof v[0], cmp_int);

    for (size_t i = 0; i < n; i++)
        printf("%d ", v[i]);
    putchar('\n');                      /* -3 7 7 19 42 */
    return 0;
}
```

> **Pour aller plus loin :** `static` sur une fonction (ou une variable globale) lui donne une **liaison interne** : elle n'est visible qu'à l'intérieur de son propre fichier, ce qui évite les collisions de noms à l'édition de liens. `static` sur une variable *locale* lui donne une **durée de vie statique** : elle conserve sa valeur d'un appel à l'autre. Un même mot-clé, deux sens ; le C++ en ajoute un troisième (les membres de classe partagés par tous les objets).

---

## 6. Mémoire et pointeurs — le cœur du C

> **En clair :** imagine la mémoire comme une très longue rue de boîtes aux lettres numérotées. Chaque boîte contient un octet et possède une **adresse** (son numéro). Un **pointeur** est simplement une variable qui stocke une adresse : il « pointe vers » une autre variable. Les pointeurs sont ce qui rend le C puissant, et aussi dangereux : le langage te fait une confiance totale.

### Où vivent les variables

```
 Adresses hautes
┌────────────────────────────────────────────┐
│ Pile (stack)                               │  variables locales, appels de fonctions
│   ↓ croît (en général) vers le bas         │  créée/détruite automatiquement
│                                            │
│   ↑ croît vers le haut                     │
│ Tas (heap)                                 │  malloc / new : TOI tu décides quand il meurt
├────────────────────────────────────────────┤
│ .bss     globales non initialisées         │
│ .data    globales initialisées             │
│ .rodata  littéraux de chaîne, constantes   │
│ .text    ton code machine                  │
└────────────────────────────────────────────┘
 Adresses basses
```

| Région | Contient | Durée de vie |
|---|---|---|
| **Pile** (*stack*) | Variables locales, arguments de fonctions, adresses de retour | Jusqu'au retour de la fonction |
| **Tas** (*heap*) | La mémoire demandée avec `malloc` / `new` | Jusqu'à ce que tu la libères |
| **`.data` / `.bss`** | Variables globales et `static` | Tout le programme |
| **`.rodata`** | Littéraux de chaîne et constantes | Tout le programme (lecture seule) |
| **`.text`** | Le code compilé | Tout le programme (lecture seule, exécutable) |

Les noms `.text`, `.data`, `.rdata` (le nom Windows de `.rodata`) et `.bss` sont les mêmes noms de sections que ceux stockés dans un fichier exécutable : voir [Référence des sections standard](The%20Windows%20Executable%20%28.exe%29%20Format.md#référence-des-sections-standard).

### Les bases des pointeurs

```c
#include <stdio.h>

int main(void)
{
    int x = 10;
    int *p = &x;                      /* p contient l'adresse de x */

    printf("x     = %d\n", x);
    printf("&x    = %p\n", (void *)&x);
    printf("p     = %p\n", (void *)p);   /* même adresse que &x */
    printf("*p    = %d\n", *p);          /* suivre le pointeur : 10 */

    *p = 25;                             /* écrire à travers le pointeur */
    printf("x vaut maintenant %d\n", x); /* 25 */

    int arr[3] = {10, 20, 30};
    int *q = arr;                        /* un tableau se dégrade en pointeur vers son premier élément */
    printf("%d %d %d\n", *q, *(q + 1), *(q + 2));   /* q + 1 avance d'un int entier */
    return 0;
}
```

| Syntaxe | Signification |
|---|---|
| `int *p;` | Déclare `p` comme pointeur vers un `int` |
| `&x` | **Adresse de** `x` |
| `*p` | **Déréférencement** : l'objet vers lequel `p` pointe |
| `p->field` | Raccourci pour `(*p).field` (accès à un membre de structure via un pointeur) |
| `p + 1` | Adresse de l'*élément suivant* : avance de `sizeof(*p)` octets, pas de 1 |
| `NULL` (`nullptr` en C++11, et en C23) | « Ne pointe vers rien ». Le déréférencer est un comportement indéfini. |

### `const` et pointeurs : lis de droite à gauche

| Déclaration | Se lit | Tu peux modifier... |
|---|---|---|
| `const int *p` | « p est un pointeur vers un int constant » | le pointeur, pas la valeur pointée |
| `int *const p` | « p est un pointeur constant vers un int » | la valeur pointée, pas le pointeur |
| `const int *const p` | « pointeur constant vers un int constant » | ni l'un ni l'autre |

### Les bugs classiques de pointeurs

```c
int *dangling(void)
{
    int local = 42;
    return &local;        /* BUG : local est détruite quand la fonction retourne */
}

void uninitialized(void)
{
    int *p;               /* BUG : p contient n'importe quoi */
    *p = 1;               /* écrit à une adresse aléatoire */
}
```

> **⚠ Important :** un **pointeur pendant** (vers quelque chose qui n'existe plus), un **pointeur non initialisé** et un **pointeur nul** sont les trois façons de détenir une mauvaise adresse. La parade est une habitude : initialise chaque pointeur (avec une adresse valide ou `NULL`), et ne renvoie jamais l'adresse d'une variable locale.

> **Pour aller plus loin :**
> - **Aliasing strict :** accéder à un objet via un pointeur de type incompatible (par exemple lire un `float` à travers un `int *`) est un comportement indéfini, sur lequel les optimiseurs s'appuient. Pour réinterpréter légalement des octets, utilise `memcpy`, une `union` (C uniquement) ou `std::bit_cast` (C++20), ou accède-y via un `char *`, toujours autorisé.
> - **Alignement :** la plupart des types doivent résider à des adresses multiples de leur alignement (souvent leur taille). Un accès non aligné est un comportement indéfini et fait planter certains processeurs.
> - L'arithmétique de pointeurs n'est définie qu'à l'intérieur d'un tableau (ou un élément après sa fin). Calculer un pointeur plus loin est un comportement indéfini, même si tu ne le déréférences jamais.

---

## 7. Tableaux et chaînes de caractères

### Les tableaux

> **En clair :** un tableau est une rangée de boîtes du même type, côte à côte en mémoire. Tu atteins la *n*-ième boîte avec `a[n]`, en comptant à partir de **0**. Le C ne vérifie **pas** que `n` est dans les limites.

```c
#include <stdio.h>

int main(void)
{
    int scores[5] = {90, 85, 70};                      /* les autres sont initialisés à zéro */
    size_t count = sizeof scores / sizeof scores[0];   /* 5 : octets totaux / octets par élément */

    for (size_t i = 0; i < count; i++)
        printf("scores[%zu] = %d\n", i, scores[i]);
    return 0;
}
```

Deux faits à garder en tête :

- **Les indices valides vont de `0` à `taille - 1`.** Écrire `scores[5]` compile sans problème et corrompt ce qui se trouve après le tableau (voir la [section 13](#13-comportement-indéfini-et-pièges-classiques)).
- **Les tableaux se dégradent en pointeurs quand on les passe à des fonctions.** À l'intérieur de la fonction, `sizeof` donne la taille d'un *pointeur*, pas celle du tableau : il faut donc passer la longueur séparément :

```c
void print_all(const int *values, size_t count)   /* "int values[]" signifie exactement la même chose */
{
    for (size_t i = 0; i < count; i++)
        printf("%d\n", values[i]);
}
```

### Les chaînes en C

> **En clair :** le C n'a pas de type chaîne. Une « chaîne » est un tableau de `char` dont la fin est marquée par un octet nul, `'\0'` (le **terminateur nul**). `"Hi"` occupe **3** octets : `'H'`, `'i'`, `'\0'`.

```c
char  a[] = "Hi";          /* une COPIE en mémoire modifiable : 3 octets, tu peux les changer       */
const char *b = "Hi";      /* pointe vers un littéral en lecture seule : ne le modifie jamais        */
```

> **⚠ Important :** modifier un littéral de chaîne (`char *s = "Hi"; s[0] = 'h';`) est un comportement indéfini et plante généralement. Déclare ces pointeurs en `const char *` pour que le compilateur t'arrête.

| Fonction (`<string.h>`) | Rôle | Attention |
|---|---|---|
| `strlen(s)` | Longueur, sans compter `'\0'` | Parcourt la chaîne : O(n) |
| `strcmp(a, b)` | `0` si égales, négatif / positif sinon | Ne compare jamais des chaînes avec `==` (cela compare des adresses) |
| `strcpy(dst, src)` | Copie une chaîne | **Dangereuse** : aucune vérification de taille |
| `strncpy(dst, src, n)` | Copie au plus `n` caractères | **N'ajoute pas toujours `'\0'`** ; complète avec des zéros : facile à mal utiliser |
| `strcat(dst, src)` | Ajoute à la fin | **Dangereuse** : aucune vérification de taille |
| `snprintf(dst, size, fmt, ...)` | Copie formatée, **ne déborde jamais**, termine toujours la chaîne | Le choix sûr par défaut |

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char buf[8];
    const char *src = "Hello, World!";               /* 13 caractères + '\0' */

    snprintf(buf, sizeof buf, "%s", src);            /* tronque proprement à 7 caractères + '\0' */
    printf("buf = \"%s\" (longueur %zu)\n", buf, strlen(buf));
    return 0;
}
```

Le résultat est `buf = "Hello, " (longueur 7)`. Le tampon n'est jamais dépassé et toujours terminé. GCC remarque même à la compilation que le texte ne tiendra pas et prévient (`-Wformat-truncation`) : la fonction protège ta mémoire, l'avertissement protège ton intention.

> **✔ Astuce :** en C++, utilise `std::string`. Elle grandit automatiquement, connaît sa longueur, se compare avec `==` et ne peut pas déborder dans un usage normal.

---

## 8. Structures, énumérations et unions

### Les structures : regrouper des données liées

```c
#include <stdio.h>

typedef struct {
    char name[20];
    int  age;
} Person;

static void print_person(const Person *p)
{
    printf("%s (%d)\n", p->name, p->age);     /* p->age est un raccourci pour (*p).age */
}

int main(void)
{
    Person alice = { "Alice", 30 };
    print_person(&alice);                      /* on passe l'adresse : pas de grosse copie */
    return 0;
}
```

### Remplissage et alignement

> **En clair :** le compilateur peut laisser des trous dans une structure pour que chaque membre se trouve à une adresse qui plaît à son type. Une structure peut donc être **plus grosse que la somme de ses membres**, et l'*ordre* des membres compte.

```c
#include <stdio.h>

struct Bad  { char a; int b; char c; };   /* a, 3 octets de remplissage, b, c, 3 octets de remplissage */
struct Good { int b; char a; char c; };   /* b, a, c, 2 octets de remplissage                          */

int main(void)
{
    printf("Bad : %zu octets\n", sizeof(struct Bad));    /* typiquement 12 */
    printf("Good: %zu octets\n", sizeof(struct Good));   /* typiquement 8  */
    return 0;
}
```

La même idée, aligner les données sur une frontière, est derrière `FileAlignment` et `SectionAlignment` dans le [guide `.exe`](The%20Windows%20Executable%20%28.exe%29%20Format.md). Règle pratique : déclare les membres du plus grand au plus petit.

> **Pour aller plus loin :** ne suppose jamais que la disposition d'une structure correspond à un format de fichier ou à un paquet réseau. Le remplissage, l'endianness et l'ordre des membres relèvent du compilateur et du processeur. Sérialise champ par champ, ou utilise des types explicites à largeur fixe avec un empaquetage propre au compilateur (`#pragma pack`, `__attribute__((packed))`) en acceptant les compromis de portabilité et de vitesse.

### Les énumérations : des constantes nommées

```c
enum Color { RED, GREEN, BLUE };          /* RED = 0, GREEN = 1, BLUE = 2 */
enum Color c = GREEN;
```

En C, une énumération est essentiellement un `int` avec des noms parlants, et ses constantes fuient dans la portée englobante. Le C++11 ajoute **`enum class`**, qui est cloisonnée (`Color::Green`) et ne se convertit pas implicitement en `int`. Préfère-la en C++.

### Les unions : un seul stockage, plusieurs vues

```c
#include <stdio.h>
#include <stdint.h>

union Bits { float f; uint32_t u; };       /* les deux membres partagent les mêmes 4 octets */

int main(void)
{
    union Bits b = { .f = 1.0f };
    printf("1.0f en hexadécimal : 0x%08X\n", (unsigned)b.u);   /* 0x3F800000 avec des float IEEE 754 */
    return 0;
}
```

> **Pour aller plus loin :** lire un membre d'union autre que le dernier écrit (« type punning ») est permis en C mais est un **comportement indéfini en C++**. La façon portable, dans les deux langages, est `memcpy` :

```cpp
#include <cstdint>
#include <cstdio>
#include <cstring>

int main()
{
    float f = 1.0f;
    std::uint32_t u;
    static_assert(sizeof f == sizeof u, "les tailles doivent être identiques");
    std::memcpy(&u, &f, sizeof u);                      // bien défini en C et en C++
    std::printf("0x%08X\n", static_cast<unsigned>(u));  // 0x3F800000
}
```

---

## 9. Mémoire dynamique

> **En clair :** les variables de la pile disparaissent quand leur fonction se termine. Quand des données doivent survivre à la fonction, ou que leur taille n'est connue qu'à l'exécution, tu demandes de la mémoire au système sur le **tas**, et **c'est à toi de la rendre**.

### En C : `malloc` et `free`

| Fonction (`<stdlib.h>`) | Rôle |
|---|---|
| `malloc(n)` | Réserve `n` octets. Le contenu est **non initialisé**. Renvoie `NULL` en cas d'échec. |
| `calloc(count, size)` | Réserve `count × size` octets, **mis à zéro**. |
| `realloc(ptr, n)` | Redimensionne un bloc, en le déplaçant éventuellement. Renvoie `NULL` en cas d'échec (l'ancien bloc reste valide). |
| `free(ptr)` | Libère un bloc. `free(NULL)` est permis et ne fait rien. |

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    size_t capacity = 2, size = 0;
    int *data = malloc(capacity * sizeof *data);        /* sizeof *data survit à un changement de type */
    if (data == NULL) { perror("malloc"); return 1; }   /* TOUJOURS vérifier */

    for (int i = 1; i <= 5; i++) {
        if (size == capacity) {                         /* plein : on agrandit le tableau */
            size_t new_cap = capacity * 2;
            int *tmp = realloc(data, new_cap * sizeof *data);
            if (tmp == NULL) { free(data); perror("realloc"); return 1; }
            data = tmp;                  /* on n'écrase 'data' qu'une fois realloc réussi */
            capacity = new_cap;
        }
        data[size++] = i * i;
    }

    for (size_t i = 0; i < size; i++)
        printf("%d ", data[i]);
    putchar('\n');                                      /* 1 4 9 16 25 */

    free(data);
    data = NULL;                                        /* aucun pointeur pendant derrière nous */
    return 0;
}
```

### Les cinq bugs classiques du tas

| Bug | Ce qui se passe | Parade |
|---|---|---|
| **Fuite mémoire** | Tu n'appelles jamais `free` : la consommation de mémoire grandit jusqu'à la fin du programme | Chaque `malloc` exige exactement un `free` correspondant |
| **Double libération** | `free(p); free(p);` : corrompt l'allocateur, souvent exploitable | Mets `p = NULL` après la libération |
| **Utilisation après libération** | Lire ou écrire dans un bloc déjà libéré | Pareil : annule le pointeur et garde la propriété claire |
| **Dépassement de tampon** | Écrire au-delà de la fin du bloc | Suis les tailles, utilise des opérations qui vérifient les bornes |
| **Lecture non initialisée** | La mémoire de `malloc` a un contenu aléatoire | Utilise `calloc`, ou initialise avant de lire |

### En C++ : laisse les objets faire le ménage (RAII)

> **En clair :** le C++ attache la durée de vie d'une ressource à celle d'un objet. Quand l'objet sort de sa portée, son **destructeur** s'exécute automatiquement et libère la ressource. Cette idée s'appelle **RAII** (*Resource Acquisition Is Initialization*, « l'acquisition d'une ressource est son initialisation »), et c'est l'idiome le plus important du C++ : il supprime la plupart des appels manuels à `free` / `delete`, et avec eux la plupart des fuites.

Le C++ dispose de `new` / `delete` (et de `new[]` / `delete[]` pour les tableaux ; les confondre est un comportement indéfini), mais le code moderne les appelle rarement directement. Il utilise plutôt des **pointeurs intelligents** :

| Pointeur intelligent (`<memory>`) | Propriété | À utiliser quand |
|---|---|---|
| `std::unique_ptr<T>` | **Un seul** propriétaire, non copiable (déplaçable), aucun surcoût | Le choix par défaut |
| `std::shared_ptr<T>` | Propriété **partagée** par comptage de références | Plusieurs propriétaires ont réellement besoin de l'objet |
| `std::weak_ptr<T>` | Observateur sans propriété d'un `shared_ptr` | Casser les cycles de propriété |

```cpp
#include <iostream>
#include <memory>

struct Widget {
    explicit Widget(int id) : id_(id) { std::cout << "Widget " << id_ << " créé\n"; }
    ~Widget()                         { std::cout << "Widget " << id_ << " détruit\n"; }
    int id_;
};

int main()
{
    auto w = std::make_unique<Widget>(1);     // libéré automatiquement à la fin de main
    {
        auto s1 = std::make_shared<Widget>(2);
        auto s2 = s1;                         // le compteur de références vaut maintenant 2
        std::cout << "use_count = " << s1.use_count() << '\n';
    }                                         // dernier propriétaire parti : Widget 2 détruit ici
    std::cout << "fin de main\n";
}                                             // Widget 1 détruit ici
```

> **✔ Astuce :** en C++ moderne, si tu tapes toi-même `new` ou `delete`, arrête-toi et demande-toi si `std::vector`, `std::string`, `std::make_unique` ou `std::make_shared` ferait l'affaire. Presque toujours, l'un d'eux la fait.

---

## 10. En-têtes, fichiers multiples et outils de build

> **En clair :** les vrais programmes sont répartis sur de nombreux fichiers. Un **en-tête** (`.h` / `.hpp`) liste *ce qui existe* (les déclarations) ; un **fichier source** (`.c` / `.cpp`) contient *comment ça marche* (les définitions). Les autres fichiers font un `#include` de l'en-tête pour savoir comment appeler ton code, et l'éditeur de liens assemble le tout.

**math_utils.h**

```c
#ifndef MATH_UTILS_H        /* garde d'inclusion : évite la double inclusion */
#define MATH_UTILS_H

int add(int a, int b);      /* déclarations seulement */
int square(int x);

#endif /* MATH_UTILS_H */
```

**math_utils.c**

```c
#include "math_utils.h"

int add(int a, int b) { return a + b; }       /* définitions */
int square(int x)     { return x * x; }
```

**main.c**

```c
#include <stdio.h>
#include "math_utils.h"

int main(void)
{
    printf("%d\n", add(2, 3));       /* 5  */
    printf("%d\n", square(4));       /* 16 */
    return 0;
}
```

Compile à la main, une unité de traduction à la fois, puis lie :

```bash
gcc -std=c17 -Wall -Wextra -Wpedantic -c main.c
gcc -std=c17 -Wall -Wextra -Wpedantic -c math_utils.c
gcc main.o math_utils.o -o app
```

| Règle | Pourquoi |
|---|---|
| `#include <fichier>` pour les en-têtes système / de bibliothèques, `#include "fichier"` pour les tiens | `<>` cherche dans les chemins système ; `""` cherche d'abord dans ton projet |
| Chaque en-tête reçoit une **garde d'inclusion** (`#ifndef` / `#define` / `#endif`, ou `#pragma once`) | Évite les erreurs de « redéfinition » quand un en-tête est inclus deux fois. `#pragma once` n'est pas dans la norme mais tous les grands compilateurs le gèrent. |
| Mets les **déclarations** dans les en-têtes, les **définitions** dans les fichiers sources | Une fonction définie dans un en-tête et incluse dans deux fichiers est définie deux fois, et l'éditeur de liens se plaint |
| Marque `static` les fonctions auxiliaires privées à un fichier | Évite les collisions de noms entre fichiers |
| Lie les bibliothèques supplémentaires avec `-l` (par ex. `-lm` pour `<math.h>` sous Linux) | La bibliothèque mathématique standard est une bibliothèque à part sur beaucoup de systèmes |

### Automatiser le build : Make

Taper des commandes de compilation ne passe pas à l'échelle. **Make** ne reconstruit que ce qui a changé. Enregistre ceci dans un fichier `Makefile` (**les lignes de recette doivent commencer par une tabulation, pas par des espaces**) :

```make
CC      = gcc
CFLAGS  = -std=c17 -Wall -Wextra -Wpedantic -g

app: main.o math_utils.o
	$(CC) $(CFLAGS) $^ -o $@

%.o: %.c math_utils.h
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f app *.o
```

Lance `make` pour construire et `make clean` pour supprimer les fichiers produits.

### Automatiser le build : CMake

Pour les projets plus gros ou multiplateformes, **CMake** génère les fichiers de build natifs (Makefiles, Ninja, projets Visual Studio) à partir d'une description unique :

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyApp C)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_executable(app main.c math_utils.c)

if(MSVC)
  target_compile_options(app PRIVATE /W4)
else()
  target_compile_options(app PRIVATE -Wall -Wextra -Wpedantic)
endif()
```

```bash
cmake -S . -B build
cmake --build build
```

### Mélanger C et C++ : `extern "C"`

Le C++ encode les types des paramètres dans les noms de fonctions (**décoration des noms**, *name mangling*) pour que les surcharges puissent coexister ; le C ne le fait pas. Pour appeler du code C depuis du C++ (ou exposer du code C++ au C), les déclarations doivent être marquées avec la liaison C. Le motif d'en-tête standard est :

```c
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

#ifdef __cplusplus
extern "C" {                /* seul le compilateur C++ voit ceci */
#endif

int add(int a, int b);
int square(int x);

#ifdef __cplusplus
}
#endif

#endif /* MATH_UTILS_H */
```

---

## 11. L'essentiel du C++ — ce qui change

> **En clair :** tout ce qui figure dans les sections 1 à 10 fonctionne aussi en C++. Cette section couvre ce que le C++ *ajoute* pour que tu écrives moins de code, fasses moins d'erreurs de mémoire et restes tout aussi rapide. Compile avec `g++ -std=c++20 -Wall -Wextra -Wpedantic`.

### 11.1 Les références

Une **référence** est un alias d'une variable existante. Contrairement à un pointeur, elle ne peut pas être nulle et ne peut pas être réorientée ailleurs après son initialisation.

```cpp
int x = 10;
int &r = x;        // r est un autre nom pour x
r = 20;            // x vaut maintenant 20

void print(const std::string &s);   // passe les gros objets par référence constante :
                                    // pas de copie, et la fonction ne peut pas les modifier
```

### 11.2 Classes et RAII

Une **classe** regroupe des données et les fonctions qui travaillent dessus. Les **constructeurs** préparent un objet ; le **destructeur** le nettoie. Combiné au RAII ([section 9](#9-mémoire-dynamique)), cela rend la gestion des ressources automatique et sûre face aux exceptions :

```cpp
#include <cstdio>
#include <stdexcept>

class File {
public:
    File(const char *path, const char *mode) : fp_(std::fopen(path, mode))
    {
        if (!fp_) throw std::runtime_error("impossible d'ouvrir le fichier");
    }
    ~File() { if (fp_) std::fclose(fp_); }            // s'exécute toujours quand File sort de sa portée

    File(const File &) = delete;                      // interdit la copie : deux propriétaires
    File &operator=(const File &) = delete;           // fermeraient deux fois le même FILE

    std::FILE *get() const { return fp_; }

private:
    std::FILE *fp_;
};

int main()
{
    File out("hello.txt", "w");
    std::fputs("écrit en toute sécurité\n", out.get());
}   // fichier fermé ici, même si une exception avait été levée plus haut
```

| Mot-clé | Signification |
|---|---|
| `public` / `private` / `protected` | Qui peut accéder à un membre (`struct` est public par défaut, `class` privé) |
| `explicit` | Un constructeur à un argument ne peut pas servir à des conversions implicites silencieuses |
| `const` après une méthode | La méthode promet de ne pas modifier l'objet |
| `= delete` / `= default` | Interdit / demande explicitement une fonction spéciale générée par le compilateur |
| `override` | Indique au compilateur que cette méthode doit redéfinir une méthode virtuelle (détecte les fautes de frappe) |
| `virtual` | Active le polymorphisme à l'exécution via des pointeurs/références vers la classe de base |

### 11.3 La bibliothèque standard : conteneurs, chaînes, algorithmes

Tu as rarement besoin d'écrire toi-même un tableau dynamique, une chaîne ou une table triée. La bibliothèque standard t'en fournit de testés et de rapides :

| Besoin | Utilise | En-tête |
|---|---|---|
| Texte | `std::string` | `<string>` |
| Tableau extensible | `std::vector<T>` | `<vector>` |
| Tableau de taille fixe | `std::array<T, N>` | `<array>` |
| Clé → valeur triée | `std::map<K, V>` | `<map>` |
| Clé → valeur par hachage | `std::unordered_map<K, V>` | `<unordered_map>` |
| Trier, chercher, compter, transformer... | `std::sort`, `std::find`, `std::count_if`, ... | `<algorithm>` |

```cpp
#include <algorithm>
#include <iostream>
#include <map>
#include <string>
#include <utility>
#include <vector>

int main()
{
    std::vector<std::string> words = {"pear", "apple", "fig", "apple", "pear", "apple"};

    std::map<std::string, int> counts;                 // trié par clé
    for (const auto &w : words)                        // for basé sur intervalle : pas d'indices à la main
        ++counts[w];                                   // une clé absente démarre à 0

    std::vector<std::pair<std::string, int>> ranked(counts.begin(), counts.end());
    std::sort(ranked.begin(), ranked.end(),
              [](const auto &a, const auto &b) { return a.second > b.second; });   // lambda

    for (const auto &[word, n] : ranked)               // liaisons structurées (C++17)
        std::cout << word << ": " << n << '\n';
}
```

Le même programme en C demanderait une table de hachage ou un tableau trié écrits à la main, de la comparaison de chaînes et de la gestion manuelle de la mémoire. Cette différence est la raison d'être du C++.

> **⚠ Important :** `vector[i]` et `string[i]` ne vérifient **pas** les bornes (comme les tableaux C). `.at(i)` les vérifie, et lève `std::out_of_range` quand `i` est invalide.

### 11.4 Les templates : du code qui fonctionne pour n'importe quel type

Un **template** (patron) est une recette que le compilateur instancie pour chaque type que tu utilises. C'est ainsi que sont écrits les conteneurs standard, et cela ne coûte rien à l'exécution :

```cpp
#include <iostream>
#include <string>

template <typename T>
T maximum(const T &a, const T &b) { return (a < b) ? b : a; }

int main()
{
    std::cout << maximum(3, 7) << '\n';                        // T déduit : int
    std::cout << maximum(2.5, 1.5) << '\n';                    // T déduit : double
    std::cout << maximum<std::string>("abc", "abd") << '\n';   // T donné explicitement
}
```

> **Pour aller plus loin :** les templates sont vérifiés à l'instanciation, ce qui a longtemps produit des pages de messages d'erreur. Les **concepts** (C++20) permettent d'énoncer des exigences (`template <std::totally_ordered T>`) et d'obtenir des diagnostics lisibles à la place.

### 11.5 Les exceptions

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>

int main()
{
    std::vector<int> v = {1, 2, 3};
    try {
        std::cout << v.at(10) << '\n';               // at() vérifie l'indice et lève une exception
    } catch (const std::out_of_range &e) {
        std::cout << "attrapé : " << e.what() << '\n';
    }
}
```

Quand une exception est levée, le C++ **déroule la pile**, en exécutant le destructeur de chaque objet rencontré. C'est précisément pourquoi le RAII rend le nettoyage fiable. Ne laisse jamais une exception s'échapper d'un destructeur.

### 11.6 Copie, déplacement, et règle du zéro / des trois / des cinq

Copier un gros objet coûte cher. Le C++11 a introduit la **sémantique de déplacement** : si la source est un temporaire (ou si tu la marques avec `std::move`), la destination peut *voler* ses ressources au lieu de les copier, comme lorsqu'on renvoie un `std::vector` depuis une fonction sans coût réel.

| Règle | Énoncé |
|---|---|
| **Règle du zéro** | Construis tes classes à partir de membres qui se gèrent déjà eux-mêmes (`std::string`, `std::vector`, pointeurs intelligents) et n'écris **aucune** des fonctions spéciales ci-dessous. C'est l'objectif. |
| **Règle des trois** | Si tu écris un destructeur, un constructeur de copie ou un opérateur d'affectation par copie, tu as presque certainement besoin des trois. |
| **Règle des cinq** | Ajoute le constructeur de déplacement et l'affectation par déplacement aux trois précédents. |

### 11.7 Le C++ moderne en un coup d'œil

| Fonctionnalité | Depuis | Ce qu'elle t'apporte |
|---|---|---|
| `auto`, `for` basé sur intervalle, lambdas, `nullptr`, `constexpr`, `enum class`, `override`, pointeurs intelligents, sémantique de déplacement, `std::thread` | **C++11** | La base du « C++ moderne » |
| `std::make_unique`, lambdas génériques, séparateurs de chiffres (`1'000'000`) | **C++14** | Des finitions sur C++11 |
| Liaisons structurées, `std::optional`, `std::variant`, `std::string_view`, `std::filesystem`, `if constexpr`, variables inline | **C++17** | Des commodités de tous les jours |
| Concepts, ranges, `std::span`, `<=>`, coroutines, modules, `std::format`, `std::bit_cast` | **C++20** | Des ajouts majeurs au langage |
| `std::expected`, `std::print`, `std::mdspan`, `import std;` | **C++23** | Des raffinements sur la gestion d'erreurs et les E/S |

---

## 12. C ou C++ : côte à côte

| Tâche | La manière C | La manière C++ |
|---|---|---|
| Afficher | `printf("%d\n", n);` | `std::cout << n << '\n';` |
| Texte | `char buf[N]` + `snprintf` / `strlen` | `std::string` |
| Tableau extensible | `malloc` + `realloc` + `free` + une variable de taille | `std::vector<T>` |
| Objet unique sur le tas | `malloc(sizeof *p)` ... `free(p)` | `std::make_unique<T>()` |
| Modifier la variable de l'appelant | Paramètre pointeur | Paramètre référence |
| Code générique | `void *` + rappels, ou macros, ou `_Generic` | Templates |
| Gestion d'erreurs | Codes de retour, `errno` | Exceptions (ou codes de retour / `std::expected`) |
| Éviter les collisions de noms | Préfixer chaque nom (`mylib_init`) | `namespace` |
| Libérer les ressources | À la main, idéalement un seul bloc de nettoyage | Destructeurs (RAII) |
| Constante | `#define N 10` ou `const int` | `constexpr int N = 10;` |
| Pointeur nul | `NULL` (`nullptr` en C23) | `nullptr` |

> **✔ Astuce :** choisir le C est rarement une question de vitesse (les deux produisent le même genre de code machine). C'est une question de **simplicité, d'environnement d'exécution minuscule, d'ABI stable, ou de cible où seul le C est disponible**. Choisis le C++ quand la taille et la complexité du programme justifient des abstractions et un nettoyage automatique.

---

## 13. Comportement indéfini et pièges classiques

> **En clair :** dans la plupart des autres langages, une erreur produit un message d'erreur. En C et en C++, beaucoup d'erreurs produisent un **comportement indéfini** (*undefined behavior*, UB) : le langage ne fait *aucune promesse* sur ce qui se passe. Le programme peut planter, afficher n'importe quoi, ou sembler fonctionner parfaitement pendant des mois, et se comporter autrement dès que tu changes le niveau d'optimisation ou de compilateur. Le comportement indéfini est la principale raison pour laquelle le C et le C++ sont jugés dangereux, il mérite donc sa propre section.

Trois sortes de « non précisé » méritent d'être distinguées :

| Terme | Signification | Exemple |
|---|---|---|
| **Défini par l'implémentation** | Peut varier, mais le compilateur doit documenter son choix | Taille d'un `int` ; `char` signé ou non |
| **Non spécifié** | Plusieurs résultats sont permis ; aucune documentation requise | Ordre d'évaluation des arguments d'une fonction |
| **Indéfini** | Aucune exigence, n'importe quoi peut arriver | Tout le tableau ci-dessous |

### Sources courantes de comportement indéfini

| Cause | Exemple |
|---|---|
| Accès hors limites à un tableau / pointeur | `int a[3]; a[3] = 0;` |
| Déréférencer `NULL` ou un pointeur pendant | `*p` après `free(p)` ; renvoyer `&local` |
| Dépassement d'entier signé | `INT_MAX + 1` |
| Lire une variable non initialisée | `int x; printf("%d", x);` |
| Modifier un littéral de chaîne | `char *s = "hi"; s[0] = 'H';` |
| Décaler d'une quantité négative ou ≥ à la largeur du type | `1 << 32` pour un `int` de 32 bits |
| Violer l'aliasing strict | Lire un `float` via un `int *` |
| Course aux données (deux threads, une écriture, aucune synchronisation) | Compteur partagé non protégé |
| Atteindre la fin d'une fonction non `void` | Un `return` manquant (UB toujours en C++ ; en C seulement si l'appelant utilise la valeur) |
| Modifier une variable deux fois sans séquencement entre les deux | Les règles diffèrent selon les langages et les normes : ne le fais pas |

> **Pour aller plus loin :** puisque le comportement indéfini « ne peut pas arriver » dans un programme correct, les optimiseurs ont le droit de *supposer* qu'il n'arrive jamais. Un test comme `if (x + 1 < x)`, censé détecter un dépassement signé, peut être purement supprimé, car pour un `x` signé il est « toujours faux ». Un code peut sembler fonctionner en mode debug et casser en `-O2`. Ce n'est pas un bug du compilateur : le programme était déjà faux.

### Les pièges de tous les jours

| Piège | Pourquoi il mord | Parade |
|---|---|---|
| `if (x = 5)` | Affecte au lieu de comparer | `==` ; compile avec `-Wall` |
| `if (x == 5);` | Le `;` égaré rend le corps du `if` vide | Pas de point-virgule ; `-Wall` prévient |
| `break` manquant dans un `switch` | L'exécution enchaîne sur le `case` suivant | Ajoute `break` ; utilise `-Wimplicit-fallthrough` |
| `sizeof(pointeur)` au lieu de la taille du tableau | Donne 4 ou 8, pas la taille du tableau | Passe la longueur explicitement |
| Erreur d'un cran (`i <= n` au lieu de `i < n`) | Lit ou écrit un élément de trop | Préfère `for (i = 0; i < n; i++)` |
| `7 / 2` en attendant `3.5` | La division entière tronque | `7 / 2.0` ou `(double)7 / 2` |
| `0.1 + 0.2 == 0.3` | Le binaire flottant ne peut pas représenter 0,1 exactement | Compare avec une tolérance |
| `a == b` sur deux chaînes C | Compare des adresses, pas des contenus | `strcmp(a, b) == 0` |
| Arguments de macro avec effets de bord ou opérateurs | Les macros font de la substitution de texte, pas des appels | Parenthèse tout, ou utilise `static inline` / `constexpr` |

Le piège des macros mérite une démonstration :

```c
#include <stdio.h>

#define SQUARE_BAD(x)  x * x
#define SQUARE_OK(x)   ((x) * (x))

int main(void)
{
    printf("%d\n", SQUARE_BAD(1 + 2));   /* se développe en 1 + 2 * 1 + 2 = 5, pas 9 */
    printf("%d\n", SQUARE_OK(1 + 2));    /* 9 */
    return 0;
}
```

---

## 14. Débogage et outils

> **En clair :** tu n'as pas à traquer les bugs mémoire en fixant le code. Les compilateurs et les outils peuvent en détecter la plupart à ta place, en général en quelques secondes. Utilise-les dès le premier jour.

### Étape 1 : laisse le compilateur t'avertir

```bash
gcc -std=c17   -Wall -Wextra -Wpedantic -g file.c        # GCC / Clang, C
g++ -std=c++20 -Wall -Wextra -Wpedantic -g file.cpp      # GCC / Clang, C++
cl /W4 /Zi file.cpp                                      # MSVC
```

`-g` ajoute des informations de débogage pour que les débogueurs puissent afficher tes lignes de source et les noms de tes variables.

### Étape 2 : utilise les sanitizers

Les **sanitizers** instrumentent ton programme pour que les bugs soient détectés *à l'instant où ils se produisent* et signalés avec un emplacement précis. Prends ce programme volontairement cassé :

<!-- no-test -->
```c
#include <stdlib.h>

int main(void)
{
    int *a = malloc(3 * sizeof *a);
    a[3] = 1;                /* écrit un élément après la fin */
    free(a);
    return 0;
}
```

```bash
gcc -g -fsanitize=address,undefined bug.c -o bug && ./bug
```

```
==12345==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x... 
WRITE of size 4 at 0x... thread T0
    #0 0x... in main bug.c:6
```

Le rapport nomme le type de bug, l'opération, et le fichier et la ligne exacts.

| Option | Détecte |
|---|---|
| `-fsanitize=address` | Accès hors limites, utilisation après libération, double libération, (sous Linux) fuites mémoire |
| `-fsanitize=undefined` | Dépassement signé, décalages invalides, accès non alignés, déréférencement nul... |
| `-fsanitize=thread` | Courses aux données (ne se combine pas avec `address`) |

MSVC gère aussi AddressSanitizer (`/fsanitize=address`). **Valgrind** (`valgrind ./app`, Linux/macOS) trouve des erreurs mémoire similaires sans recompiler, au prix de la vitesse.

### Étape 3 : un débogueur quand tu as besoin de regarder à l'intérieur

| Commande `gdb` | Rôle |
|---|---|
| `gdb ./app` | Lance le débogueur sur un programme construit avec `-g` |
| `break main` / `break file.c:42` | Pose un point d'arrêt |
| `run` | Démarre le programme |
| `next` / `step` | Exécute une ligne (`step` entre dans les fonctions, `next` les enjambe) |
| `print x` | Affiche la valeur d'une variable |
| `backtrace` | Affiche la chaîne d'appels de fonctions qui a mené ici |
| `continue` | Exécute jusqu'au prochain point d'arrêt |
| `quit` | Quitte |

### Étape 4 : analyse statique et formatage

- **clang-tidy** et **cppcheck** analysent ton code sans l'exécuter et signalent les motifs suspects.
- **clang-format** applique automatiquement un style de code cohérent.

---

## 15. Normes du langage — laquelle choisir

Les deux langages sont définis par des normes ISO, révisées régulièrement. Tu en choisis une avec l'option `-std=`. **Indique-la toujours explicitement** : les valeurs par défaut des compilateurs changent d'une version à l'autre.

| Norme C | Année | Principaux ajouts | Option GCC / Clang |
|---|---|---|---|
| **C89 / C90** | 1989/90 | La norme ANSI/ISO d'origine | `-std=c89` |
| **C99** | 1999 | Commentaires `//`, déclarations n'importe où, `long long`, `<stdint.h>`, `<stdbool.h>`, tableaux de taille variable, initialiseurs désignés, `inline` | `-std=c99` |
| **C11** | 2011 | `_Static_assert`, `_Generic`, `_Atomic`, threads optionnels, structures/unions anonymes | `-std=c11` |
| **C17** | 2018 | Seulement des corrections de défauts, aucune nouveauté | `-std=c17` |
| **C23** | 2024 | `nullptr`, mots-clés `bool`/`true`/`false`, `constexpr`, `typeof`, `#embed`, `[[attributs]]`, littéraux binaires (`0b101`), `()` signifie `(void)` | `-std=c23` (`-std=c2x` sur les compilateurs plus anciens) |

| Norme C++ | Année | Option |
|---|---|---|
| **C++98 / C++03** | 1998 / 2003 | `-std=c++98` / `-std=c++03` |
| **C++11** | 2011 | `-std=c++11` |
| **C++14** | 2014 | `-std=c++14` |
| **C++17** | 2017 | `-std=c++17` |
| **C++20** | 2020 | `-std=c++20` |
| **C++23** | 2023 (publiée en 2024) | `-std=c++23` |

(Ce qu'a apporté chaque version du C++ est listé à la [section 11.7](#117-le-c-moderne-en-un-coup-dœil). Les nouvelles révisions du C++ suivent un cycle d'environ trois ans.)

**Que choisir :** pour un nouveau projet C, prends **C17**, ou **C23** si tous tes compilateurs le gèrent. Pour un nouveau projet C++, prends **C++20**, ou **C++17** quand une chaîne d'outils est plus ancienne. Ne reste sur une norme plus ancienne que si la plateforme t'y oblige.

Pour voir quelle norme ton compilateur utilise par défaut :

```bash
echo | gcc -dM -E - | grep __STDC_VERSION__              # par ex. 201710L = C17
echo | g++ -dM -E -x c++ - | grep __cplusplus             # par ex. 201703L = C++17
```

---

## 16. Référence rapide et glossaire

### Aide-mémoire

```bash
# ── COMPILER ─────────────────────────────────────────────────────
gcc -std=c17   -Wall -Wextra -Wpedantic -g a.c -o a      # C
g++ -std=c++20 -Wall -Wextra -Wpedantic -g a.cpp -o a    # C++
gcc -c a.c                    # compile seulement → a.o
gcc a.o b.o -o app            # édition de liens
gcc a.c -lm -o a              # lie la bibliothèque mathématique
gcc -g -fsanitize=address,undefined a.c -o a   # build de debug avec sanitizers
gcc -O2 a.c -o a              # optimise (build de release)
```

```c
/* ── MÉMOIRE (C) ────────────────────────────────────────────── */
int *p = malloc(n * sizeof *p);  if (!p) { /* gérer l'erreur */ }
free(p);  p = NULL;

/* ── MÉMOIRE (C++) ──────────────────────────────────────────── */
auto up = std::make_unique<T>(args);        // un seul propriétaire
auto sp = std::make_shared<T>(args);        // propriétaires partagés
std::vector<int> v(n);                      // tableau dynamique

/* ── TAILLES ────────────────────────────────────────────────── */
size_t n = sizeof arr / sizeof arr[0];      // éléments d'un vrai tableau
```

### Glossaire

| Terme | Définition |
|---|---|
| **ABI** | Application Binary Interface : le contrat au niveau binaire (convention d'appel, disposition des types, décoration des noms) entre des morceaux de code compilés |
| **Adresse** | Le nombre qui identifie un octet en mémoire |
| **Déclaration / définition** | Une déclaration dit qu'un nom existe et quel est son type ; une définition le crée (code ou stockage) |
| **Pointeur pendant** | Un pointeur vers une mémoire libérée ou sortie de sa portée |
| **Tas / pile** | Mémoire que tu gères à la main / mémoire gérée automatiquement à chaque appel de fonction |
| **En-tête** (*header*) | Un fichier de déclarations destiné à être inclus avec `#include` |
| **Éditeur de liens** (*linker*) | L'outil qui assemble les fichiers objet et les bibliothèques en un seul exécutable |
| **Macro** | Un nom remplacé par du texte *avant* la compilation (`#define`) |
| **Espace de noms** (*namespace*) | Une portée nommée qui évite les collisions de noms en C++ |
| **Pointeur nul** | Un pointeur qui ne pointe vers rien (`NULL` / `nullptr`) |
| **Fichier objet** | La sortie compilée mais pas encore liée d'un fichier source (`.o` / `.obj`) |
| **ODR** | One Definition Rule : chaque entité ne peut être définie qu'une seule fois dans tout le programme |
| **Pointeur** | Une variable qui contient une adresse |
| **RAII** | Attacher la durée de vie d'une ressource à la portée d'un objet pour que le nettoyage soit automatique |
| **Référence** | Un alias C++ d'une variable existante |
| **STL** | Standard Template Library : conteneurs, itérateurs et algorithmes du C++ |
| **Template** | Une recette C++ que le compilateur instancie pour chaque type utilisé |
| **Unité de traduction** | Un fichier source après préprocesseur ; l'unité sur laquelle travaille le compilateur |
| **UB** | Comportement indéfini (*undefined behavior*) : le langage ne donne aucune garantie |

---

*Ce guide couvre le C (jusqu'à C23) et le C++ (jusqu'à C++23) tels que définis par les normes ISO et implémentés par GCC, Clang et MSVC. En cas de doute, l'autorité finale est la norme du langage, et le professeur le plus pratique est un compilateur dont les avertissements sont activés.*
