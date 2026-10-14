# KnowIT

> **Né de la curiosité, affiné par l'expérimentation.**

[🇬🇧 English](README.md) · 🇫🇷 Français

KnowIT est une collection de guides techniques en constante expansion, qui couvre de nombreux sujets informatiques, en français et en anglais.

Elle est née d'une habitude toute simple :

> **Si j'ai dû passer des heures à comprendre quelque chose, je m'assure que la personne suivante n'aura pas à le faire.**

Chaque guide poursuit un seul objectif : rendre un sujet plus facile à comprendre qu'il ne l'était la première fois que je l'ai rencontré. L'idée n'est pas de remplacer les documentations officielles, mais de combler l'écart entre la référence et la vraie compréhension, en expliquant non seulement **ce que** fait quelque chose, mais aussi **pourquoi** cela fonctionne ainsi.

Vois ça moins comme un manuel que comme les notes que tu aurais aimé qu'on te tende avant de te lancer.

---

## Les guides

| Sujet | Ce que tu y trouveras | 🇬🇧 English | 🇫🇷 Français |
|---|---|---|---|
| **SQL** | De ton premier `SELECT` aux jointures, fonctions de fenêtrage, index et erreurs classiques | [Read](en/SQL_cheatsheet.md) | [Lire](fr/SQL_cheatsheet.md) |
| **Markdown** | Un aide-mémoire complet de mise en forme, y compris GitHub Flavored Markdown | [Read](en/md_cheatsheet.md) | [Lire](fr/md_cheatsheet.md) |
| **Le format Windows `.exe`** | La structure d'un fichier PE, de l'en-tête DOS aux répertoires de données, avec une rampe d'accès « en clair » pour les débutants | [Read](en/The%20Windows%20Executable%20%28.exe%29%20Format.md) | [Lire](fr/The%20Windows%20Executable%20%28.exe%29%20Format.md) |
| **Samba 4 Active Directory** | Monter un contrôleur de domaine AD et y joindre un client Ubuntu avec Samba, Kerberos, realmd et SSSD | [Read](en/Samba_setup.md) | [Lire](fr/Samba_setup.md) |
| **C & C++** | Du pipeline de compilation et des pointeurs au RAII, aux templates, au comportement indéfini et au débogage, pour débutants comme vétérans | [Read](en/C_and_Cpp_guide.md) | [Lire](fr/C_and_Cpp_guide.md) |

---

## À qui s'adresse KnowIT ?

À toute personne qui a envie d'apprendre. Que tu découvres un sujet pour la première fois, que tu en retrouves un que tu n'as pas touché depuis des années, ou que tu cherches simplement un petit rappel, KnowIT est conçu pour t'aider à comprendre, pas seulement à mémoriser.

- **Les débutants** qui apprennent un concept depuis les bases.
- **Les étudiants** qui veulent des explications centrées sur la compréhension.
- **Les professionnels** qui ont parfois besoin d'une remise à niveau.
- **Les curieux** qui aiment simplement savoir comment les choses fonctionnent.

---

## Comment est-ce organisé ?

KnowIT suit un principe simple :

> **Un sujet, un document.**

Chaque document traite d'un seul sujet et s'ouvre sur sa propre table des matières. Les notions sont introduites progressivement : un guide se lit bien du début à la fin, et reste utile comme aide-mémoire par la suite.

Chaque langue vit dans son propre dossier, et **un guide garde le même nom de fichier dans toutes les langues** : pour changer de langue, il suffit de remplacer `fr` par `en` dans le chemin.

```
KnowIT/
├── README.md              ← English version
├── README.fr.md           ← tu es ici
├── en/                    ← English guides
│   ├── C_and_Cpp_guide.md
│   ├── SQL_cheatsheet.md
│   ├── md_cheatsheet.md
│   ├── Samba_setup.md
│   └── The Windows Executable (.exe) Format.md
└── fr/                    ← guides en français
    ├── C_and_Cpp_guide.md
    ├── SQL_cheatsheet.md
    ├── md_cheatsheet.md
    ├── Samba_setup.md
    └── The Windows Executable (.exe) Format.md
```

---

## Contributions et demandes

KnowIT est un projet vivant. Si tu veux suggérer un sujet, améliorer une explication, signaler une erreur ou proposer une traduction, n'hésite pas à [ouvrir une issue](../../issues) ou à soumettre une pull request. Toute contribution qui rend un sujet plus clair, plus juste ou plus accessible est la bienvenue.

**Tu veux ajouter une traduction ?** Copie le guide dans le dossier de l'autre langue, sous le même nom de fichier, et garde une structure parallèle : mêmes sections dans le même ordre, code laissé tel quel, seuls les commentaires et le texte sont traduits.

**Tu veux ajouter une nouvelle langue ?** Crée un dossier portant son code à deux lettres (par exemple `es/`) et ajoute une colonne au tableau ci-dessus.

---

## La philosophie derrière KnowIT

La plupart des guides ici n'ont pas été écrits parce que je maîtrisais déjà le sujet, mais parce que je voulais le comprendre.

Pour moi, écrire fait partie de l'apprentissage. Cela m'oblige à me poser des questions, à remettre en cause mes hypothèses, à relier les notions entre elles et à découvrir ce que je ne comprends pas encore. Quand un guide est terminé, j'ai généralement autant appris en l'écrivant qu'en lisant sur le sujet.

Ce processus façonne chaque document : chaque guide suit le chemin qui va de la curiosité à la compréhension, pour que la personne suivante puisse partir un peu plus loin sur ce chemin.

> **Écrire, c'est ma façon de tester ma propre compréhension. Si je n'arrive pas à expliquer clairement un concept, c'est que je ne le comprends probablement pas encore assez bien.**
