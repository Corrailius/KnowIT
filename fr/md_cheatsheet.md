# Aide-mémoire de mise en forme Markdown

---

## Table des matières

- [Titres](#titres)
- [Mise en forme du texte](#mise-en-forme-du-texte)
- [Paragraphes et sauts de ligne](#paragraphes-et-sauts-de-ligne)
- [Listes](#listes)
- [Liens](#liens)
- [Images](#images)
- [Citations](#citations)
- [Code](#code)
- [Tableaux](#tableaux)
- [Lignes horizontales](#lignes-horizontales)
- [Listes de tâches](#listes-de-tâches)
- [Notes de bas de page](#notes-de-bas-de-page)
- [Échapper des caractères](#échapper-des-caractères)
- [HTML dans Markdown](#html-dans-markdown)
- [Syntaxe étendue (GitHub Flavored)](#syntaxe-étendue-github-flavored)

---

## Titres

Les titres se créent avec le symbole `#`. Le nombre de `#` détermine le niveau — pense aux balises H1 à H6 en HTML.

```markdown
# Titre 1       → Le plus grand, titre de premier niveau
## Titre 2      → Titre de section
### Titre 3     → Sous-section
#### Titre 4    → Sous-sous-section
##### Titre 5   → Rarement utilisé
###### Titre 6  → Le plus petit titre
```

**Astuce :** laisse toujours une espace entre le `#` et le texte. `#Titre` ne sera pas rendu comme un titre par les interpréteurs stricts.

Tu peux aussi souligner les titres H1 et H2 avec `===` ou `---` (style Setext), même si le style `#` est plus portable :

```markdown
Titre 1
=======

Titre 2
-------
```

---

## Mise en forme du texte

Ces styles « en ligne » peuvent être combinés et imbriqués dans des paragraphes, des éléments de liste, des cellules de tableau, etc.

```markdown
**texte en gras**          → Gras (emphase forte)
*texte en italique*        → Italique (emphase)
_texte en italique_        → Également italique (même résultat)
__texte en gras__          → Également gras
***gras et italique***     → Les deux à la fois
~~texte barré~~            → Barré (GFM)
`code en ligne`            → Extrait de code à chasse fixe
<mark>surligné</mark>      → Surligné (HTML, pas universel)
```

**Exemples rendus :**

| Syntaxe | Résultat |
|--------|--------|
| `**gras**` | **gras** |
| `*italique*` | *italique* |
| `***les deux***` | ***les deux*** |
| `` `code` `` | `code` |
| `~~barré~~` | ~~barré~~ |

---

## Paragraphes et sauts de ligne

C'est l'un des points les plus délicats de Markdown, car le comportement n'est pas toujours intuitif.

Un **paragraphe** est simplement une ou plusieurs lignes de texte séparées par une **ligne vide**. Sans ligne vide, les deux lignes fusionnent en un seul paragraphe :

```markdown
Ceci est le premier paragraphe.

Ceci est le deuxième paragraphe — remarque la ligne vide au-dessus.

Cette ligne et
cette ligne vont fusionner en un seul paragraphe (pas de ligne vide entre elles).
```

Pour forcer un **saut de ligne** à l'intérieur d'un paragraphe (sans en démarrer un nouveau), termine la ligne par **deux espaces** avant d'appuyer sur Entrée, ou utilise une balise HTML `<br>` :

```markdown
Ligne un avec deux espaces à la fin  
La ligne deux apparaît sur sa propre ligne.

Ou en HTML : Ligne un<br>Ligne deux.
```

---

## Listes

### Listes non ordonnées

Utilise `-`, `*` ou `+` comme puces. Les mélanger dans une même liste est permis, mais un style incohérent est déconseillé.

```markdown
- Élément un
- Élément deux
  - Élément imbriqué (indente de 2 espaces)
  - Un autre élément imbriqué
    - Imbrication plus profonde (4 espaces)
- Élément trois
```

### Listes ordonnées

Des nombres suivis d'un point. Détail intéressant : les nombres eux-mêmes n'ont pas d'importance — Markdown les numérote correctement tout seul. C'est tout de même une bonne pratique de les numéroter correctement pour la lisibilité du texte source.

```markdown
1. Premier élément
2. Deuxième élément
   1. Élément ordonné imbriqué
   2. Un autre élément imbriqué
3. Troisième élément
```

### Listes mixtes

```markdown
1. Premier élément ordonné
2. Deuxième élément ordonné
   - Sous-élément non ordonné
   - Un autre sous-élément
3. Troisième élément ordonné
```

**Astuce :** pour ajouter un paragraphe dans un élément de liste, indente le contenu du paragraphe de la même quantité que le contenu de la liste (généralement 4 espaces ou une tabulation) :

```markdown
- Ceci est un élément de liste.

    Ce paragraphe appartient au même élément de liste, car il est indenté.

- Ceci est un nouvel élément de liste.
```

---

## Liens

```markdown
[Texte du lien](https://example.com)                        → Lien simple
[Lien avec titre](https://example.com "Infobulle au survol")  → Avec titre au survol
<https://example.com>                                   → Lien automatique (URL brute)
<email@example.com>                                     → Lien automatique (e-mail)

[Lien de référence][ref-id]                             → Style référence
[ref-id]: https://example.com "Titre facultatif"        → Définir la référence n'importe où
```

**Astuce :** les liens de style référence sont très pratiques pour la lisibilité quand une même URL est utilisée plusieurs fois, ou quand tu veux garder le texte propre.

---

## Images

Les images suivent la même syntaxe que les liens, avec un préfixe `!`. Le texte alternatif entre `[]` est important pour l'accessibilité et s'affiche quand l'image ne se charge pas.

```markdown
![Texte alternatif](image.png)
![Texte alternatif](https://example.com/image.png "Titre facultatif")

<!-- Style référence : -->
![Texte alternatif][img-ref]
[img-ref]: image.png "Titre facultatif"
```

Pour contrôler la taille d'une image (ce n'est pas du Markdown standard — il faut du HTML) :

```html
<img src="image.png" alt="Texte alternatif" width="300">
```

---

## Citations

Utilise `>` pour créer une citation (*blockquote*). Les citations peuvent être imbriquées et contenir d'autres éléments Markdown.

```markdown
> Ceci est une citation.

> Cette citation comporte
> plusieurs lignes.

> Premier niveau
>> Deuxième niveau (imbriqué)
>>> Troisième niveau

> **Markdown fonctionne dans les citations.**
> Tu peux y mettre des listes, du code, etc.
```

Voici le rendu d'une citation :

> Premier niveau
>> Deuxième niveau (imbriqué)

---

## Code

Le code existe en deux versions : **en ligne** (pour de courts extraits au sein d'une phrase) et **en blocs délimités** (pour des exemples de code sur plusieurs lignes).

### Code en ligne

Entoure le code d'un seul accent grave (*backtick*) :

```markdown
Utilise la fonction `print()` pour afficher du texte.
```

Si ton code contient lui-même un accent grave, entoure-le de doubles accents graves :

```markdown
`` Utilise des `accents graves` dans du code en ligne ``
```

### Blocs de code délimités

Utilise trois accents graves (` ``` `) ou trois tildes (`~~~`) pour ouvrir et fermer un bloc. Indiquer un langage après la clôture ouvrante active la **coloration syntaxique** :

````markdown
```python
def greet(name):
    # Ce commentaire est coloré comme du Python
    return f"Hello, {name}!"
```

```bash
sudo apt update && sudo apt upgrade -y
```

```json
{
  "name": "Alice",
  "role": "admin"
}
```
````

**Identifiants de langage courants :** `python`, `javascript`, `js`, `bash`, `sh`, `html`, `css`, `json`, `yaml`, `sql`, `markdown`, `md`, `c`, `cpp`, `java`, `go`, `rust`, `typescript`, `ts`, `diff`.

Tu peux aussi indenter le code de 4 espaces (ancien style Markdown), mais les blocs délimités sont préférables :

```markdown
    Ceci est un bloc de code
    indenté de 4 espaces
```

---

## Tableaux

Les tableaux sont une extension de GitHub Flavored Markdown (GFM) — ils ne font pas partie de la spécification d'origine, mais sont pris en charge presque partout.

```markdown
| Colonne 1  | Colonne 2  | Colonne 3  |
|------------|------------|------------|
| Cellule A1 | Cellule B1 | Cellule C1 |
| Cellule A2 | Cellule B2 | Cellule C2 |
```

### Alignement des colonnes

Les deux-points dans la ligne de séparation contrôlent l'alignement :

```markdown
| Gauche     | Centre     | Droite     |
|:-----------|:----------:|-----------:|
| Texte à gauche | Centré | Texte à droite |
| Encore à gauche | Encore au milieu | Encore à droite |
```

**Astuce :** il n'est pas nécessaire d'aligner parfaitement les barres verticales dans le texte source — c'est juste une question de style. Le tableau s'affiche correctement même si l'espacement est irrégulier.

---

## Lignes horizontales

Chacune de ces syntaxes produit une ligne de séparation horizontale. Utilise-les pour séparer les grandes sections :

```markdown
---
***
___
```

**Astuce :** laisse une ligne vide avant et après une ligne horizontale pour éviter qu'elle soit interprétée par erreur comme un titre H2 (style Setext).

---

## Listes de tâches

Une fonctionnalité de GitHub Flavored Markdown qui affiche des cases à cocher interactives (cliquables sur GitHub et de nombreuses autres plateformes) :

```markdown
- [x] Tâche déjà terminée
- [ ] Tâche encore à faire
- [ ] Une autre tâche en attente
  - [x] Sous-tâche imbriquée terminée
  - [ ] Sous-tâche imbriquée en attente
```

---

## Notes de bas de page

Prises en charge par de nombreux moteurs de rendu (GitHub, Obsidian, Pandoc), mais pas tous :

```markdown
Voici une phrase avec une note de bas de page.[^1]
Et une autre.[^note]

[^1]: Ceci est la première note de bas de page.
[^note]: Les notes de bas de page peuvent utiliser des étiquettes plutôt que des numéros.
```

Les définitions de notes de bas de page peuvent être placées n'importe où dans le document — elles s'affichent toujours en bas.

---

## Échapper des caractères

Pour afficher un caractère que Markdown interpréterait normalement comme de la syntaxe, fais-le précéder d'une barre oblique inverse `\` :

```markdown
\*Ceci n'est pas en italique\*
\# Ceci n'est pas un titre
\[Ceci n'est pas un lien\]
\`Ceci n'est pas du code\`
\\ Ceci est une vraie barre oblique inverse
```

Caractères que tu peux échapper : `\ * _ {} [] () # + - . ! |`

---

## HTML dans Markdown

Le HTML brut est pris en charge par la plupart des interpréteurs Markdown. C'est utile quand tu as besoin de fonctionnalités que Markdown ne propose pas :

```html
<details>
  <summary>Clique pour déplier</summary>
  Contenu caché révélé au clic.
</details>

<kbd>Ctrl</kbd> + <kbd>C</kbd>   → Style de touche de clavier

<sub>indice</sub>                → Style H₂O
<sup>exposant</sup>              → Style E=mc²

<br>                             → Saut de ligne forcé
```

---

## Syntaxe étendue (GitHub Flavored)

Ces fonctionnalités font partie de **GFM (GitHub Flavored Markdown)**. Elles sont largement prises en charge, mais ne figurent pas dans la spécification d'origine.

### Coloration syntaxique dans les diffs

````markdown
```diff
- Cette ligne a été supprimée
+ Cette ligne a été ajoutée
  Cette ligne est inchangée
```
````

### Émojis

```markdown
:tada:   → 🎉
:rocket: → 🚀
:white_check_mark: → ✅
```

Liste complète : [emoji-cheat-sheet.com](https://www.webfx.com/tools/emoji-cheat-sheet/)

### Alertes / encadrés (GitHub uniquement)

````markdown
> [!NOTE]
> Une information utile.

> [!TIP]
> Une suggestion pratique.

> [!WARNING]
> Un point sur lequel le lecteur doit rester prudent.

> [!IMPORTANT]
> Une information critique.
````

### Listes de définitions (Pandoc / certains moteurs de rendu)

```markdown
Terme
:   Définition du terme.

Autre terme
:   Sa définition.
```

---

## Fiche de référence rapide

| Élément | Syntaxe |
|---|---|
| H1 | `# Titre` |
| H2 | `## Section` |
| Gras | `**texte**` |
| Italique | `*texte*` |
| Gras + italique | `***texte***` |
| Barré | `~~texte~~` |
| Code en ligne | `` `code` `` |
| Lien | `[texte](url)` |
| Image | `![alt](url)` |
| Citation | `> texte` |
| Liste non ordonnée | `- élément` |
| Liste ordonnée | `1. élément` |
| Liste de tâches | `- [x] fait` |
| Ligne horizontale | `---` |
| Bloc de code | ` ```langage ` |
| Tableau | `\| col \| col \|` |
| Note de bas de page | `[^1]` |
| Échappement | `\*littéral\*` |

---
