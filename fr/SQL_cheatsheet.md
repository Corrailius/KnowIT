# Guide SQL — De zéro à l'aise

> **Pour qui ?** Les débutants complets, et celles et ceux qui en savent un peu mais veulent consolider leurs bases. Aucune expérience préalable nécessaire.

---

## Table des matières

1. [Qu'est-ce que le SQL](#quest-ce-que-le-sql)
2. [Comment fonctionnent les bases de données](#comment-fonctionnent-les-bases-de-données)
3. [Ta première requête — SELECT](#ta-première-requête--select)
4. [Filtrer les résultats — WHERE](#filtrer-les-résultats--where)
5. [Trier et limiter les résultats](#trier-et-limiter-les-résultats)
6. [Combiner des conditions](#combiner-des-conditions)
7. [Agrégation — compter et additionner](#agrégation--compter-et-additionner)
8. [Regrouper les données — GROUP BY](#regrouper-les-données--group-by)
9. [Joindre des tables](#joindre-des-tables)
10. [Sous-requêtes et CTE](#sous-requêtes-et-cte)
11. [Fonctions de fenêtrage](#fonctions-de-fenêtrage)
12. [Modifier les données — INSERT, UPDATE, DELETE](#modifier-les-données--insert-update-delete)
13. [Créer et gérer des tables](#créer-et-gérer-des-tables)
14. [Fonctions intégrées utiles](#fonctions-intégrées-utiles)
15. [L'expression CASE](#lexpression-case)
16. [Opérations ensemblistes](#opérations-ensemblistes)
17. [Index et conseils de performance](#index-et-conseils-de-performance)
18. [Erreurs courantes à éviter](#erreurs-courantes-à-éviter)
19. [Aide-mémoire rapide](#aide-mémoire-rapide)

---

## Qu'est-ce que le SQL

Le **SQL** (*Structured Query Language*, « langage de requêtes structuré », prononcé « sé-kiou-èl » ou « sicouèl ») est le langage utilisé pour dialoguer avec les bases de données. Il permet de :

- **Poser des questions** — « Combien d'utilisateurs se sont inscrits le mois dernier ? »
- **Récupérer des données** — « Donne-moi toutes les commandes de plus de 100 €. »
- **Modifier des données** — « Mets à jour le prix du produit n° 42. »
- **Créer de la structure** — « Crée une nouvelle table pour stocker des articles de blog. »

Le SQL est utilisé partout : applications, sites web, outils d'analyse, data science, et bien plus. Apprendre le SQL est l'une des compétences les plus utiles que tu puisses acquérir.

---

## Comment fonctionnent les bases de données

Imagine une base de données comme un ensemble de **feuilles de calcul**, où chaque feuille s'appelle une **table**.

**Exemple : une petite base de données de boutique**

**Table : `customers`**

| id | name       | email              | city       |
|----|------------|--------------------|------------|
| 1  | Ana Reyes  | ana@example.com    | Madrid     |
| 2  | Tom Brown  | tom@example.com    | London     |
| 3  | Li Wei     | li@example.com     | Shanghai   |

**Table : `orders`**

| id | customer_id | total | created_at  |
|----|-------------|-------|-------------|
| 1  | 1           | 89.50 | 2024-03-01  |
| 2  | 1           | 210.00| 2024-03-15  |
| 3  | 2           | 55.00 | 2024-03-20  |

Les idées clés :
- Chaque **ligne** est un enregistrement (un client, une commande).
- Chaque **colonne** est un champ (nom, e-mail, total).
- Les tables sont reliées par des **clés** — `customer_id` dans `orders` renvoie à `id` dans `customers`.

---

## Ta première requête — SELECT

`SELECT` est l'instruction SQL la plus courante. Elle récupère des données dans une table.

**Syntaxe :**
```sql
SELECT column1, column2
FROM table_name;
```

**Récupérer toutes les colonnes :**
```sql
SELECT *
FROM customers;
```
> `*` signifie « toutes les colonnes ». Pratique pour explorer, mais à éviter dans du code de production.

**Récupérer des colonnes précises :**
```sql
SELECT name, email
FROM customers;
```

**Donner un nom plus parlant à une colonne (alias) :**
```sql
SELECT name, email AS contact_email
FROM customers;
```
> `AS` renomme la colonne dans tes résultats. Cela ne modifie pas la table elle-même.

**Supprimer les doublons avec DISTINCT :**
```sql
SELECT DISTINCT city
FROM customers;
```
> Renvoie chaque ville une seule fois, même si plusieurs clients y habitent.

---

## Filtrer les résultats — WHERE

Utilise `WHERE` pour filtrer les lignes : seules celles qui remplissent une condition sont renvoyées.

```sql
SELECT *
FROM customers
WHERE city = 'London';
```

### Opérateurs de comparaison

| Opérateur | Signification          | Exemple                    |
|-----------|------------------------|----------------------------|
| `=`       | Égal à                 | `city = 'Madrid'`          |
| `!=` ou `<>` | Différent de        | `city != 'London'`         |
| `>`       | Supérieur à            | `total > 100`              |
| `<`       | Inférieur à            | `total < 50`               |
| `>=`      | Supérieur ou égal à    | `total >= 100`             |
| `<=`      | Inférieur ou égal à    | `total <= 100`             |

### Recherche par motif avec LIKE

```sql
-- Noms commençant par 'A'
SELECT * FROM customers WHERE name LIKE 'A%';

-- Noms se terminant par 'Brown'
SELECT * FROM customers WHERE name LIKE '%Brown';

-- Noms contenant 'ei'
SELECT * FROM customers WHERE name LIKE '%ei%';
```
> `%` signifie « n'importe quel nombre de caractères ». `_` signifie « exactement un caractère ».

### Tester une liste de valeurs avec IN

```sql
SELECT * FROM customers
WHERE city IN ('Madrid', 'London', 'Berlin');
```
> Bien plus propre que d'écrire `city = 'Madrid' OR city = 'London' OR city = 'Berlin'`.

### Tester un intervalle avec BETWEEN

```sql
SELECT * FROM orders
WHERE total BETWEEN 50 AND 200;
```
> Bornes incluses — 50 et 200 sont compris.

### Détecter les valeurs manquantes avec IS NULL

```sql
-- Clients sans e-mail enregistré
SELECT * FROM customers WHERE email IS NULL;

-- Clients qui ONT un e-mail
SELECT * FROM customers WHERE email IS NOT NULL;
```
> Utilise toujours `IS NULL`, jamais `= NULL` — cela ne fonctionne pas en SQL.

---

## Trier et limiter les résultats

### ORDER BY — trier les résultats

```sql
-- Trier par nom de A à Z (ordre croissant, par défaut)
SELECT * FROM customers ORDER BY name;

-- Trier par total, du plus élevé au plus faible (ordre décroissant)
SELECT * FROM orders ORDER BY total DESC;

-- Trier selon plusieurs colonnes
SELECT * FROM orders ORDER BY customer_id ASC, total DESC;
```

### LIMIT — ne renvoyer qu'un certain nombre de lignes

```sql
-- Récupérer les 5 plus grosses commandes
SELECT * FROM orders
ORDER BY total DESC
LIMIT 5;
```

### OFFSET — sauter des lignes (utile pour la pagination)

```sql
-- Sauter les 10 premières, récupérer les 10 suivantes (page 2)
SELECT * FROM orders
ORDER BY id
LIMIT 10 OFFSET 10;
```

---

## Combiner des conditions

Utilise `AND`, `OR` et `NOT` pour combiner plusieurs conditions.

```sql
-- Clients de Londres ayant passé de grosses commandes
SELECT * FROM orders
WHERE total > 100
  AND customer_id = 2;

-- Commandes très petites ou très grosses
SELECT * FROM orders
WHERE total < 20 OR total > 500;

-- Tout le monde sauf les clients de Londres
SELECT * FROM customers
WHERE NOT city = 'London';
```

> **Astuce :** utilise des parenthèses `()` pour contrôler l'ordre de la logique, exactement comme en mathématiques :
```sql
WHERE (city = 'Madrid' OR city = 'London') AND total > 50
```

---

## Agrégation — compter et additionner

Les fonctions d'agrégation réduisent plusieurs lignes à un seul nombre.

| Fonction      | Ce qu'elle fait                              |
|---------------|----------------------------------------------|
| `COUNT(*)`    | Compte toutes les lignes                     |
| `COUNT(col)`  | Compte les valeurs non nulles d'une colonne  |
| `SUM(col)`    | Additionne toutes les valeurs                |
| `AVG(col)`    | Calcule la moyenne                           |
| `MIN(col)`    | Trouve la plus petite valeur                 |
| `MAX(col)`    | Trouve la plus grande valeur                 |

**Exemples :**
```sql
-- Combien de clients avons-nous ?
SELECT COUNT(*) AS total_customers FROM customers;

-- Quel est notre chiffre d'affaires total ?
SELECT SUM(total) AS revenue FROM orders;

-- Quelle est la valeur moyenne d'une commande ?
SELECT AVG(total) AS avg_order FROM orders;

-- Plus grosse et plus petite commande
SELECT MIN(total), MAX(total) FROM orders;
```

---

## Regrouper les données — GROUP BY

`GROUP BY` s'utilise avec les fonctions d'agrégation pour obtenir des totaux **par groupe** (par catégorie, par client, par mois, etc.).

```sql
-- Combien de commandes chaque client a-t-il passées ?
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;

-- Total dépensé par client
SELECT customer_id, SUM(total) AS total_spent
FROM orders
GROUP BY customer_id;
```

### HAVING — filtrer après le regroupement

`WHERE` filtre les lignes individuelles *avant* le regroupement. `HAVING` filtre les groupes *après* le regroupement.

```sql
-- Uniquement les clients ayant passé plus d'une commande
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

> **Règle d'or :** `WHERE` pour filtrer des lignes, `HAVING` pour filtrer des agrégats.

---

## Joindre des tables

Les jointures permettent de combiner les données de plusieurs tables. C'est là que le SQL devient vraiment puissant.

Reprends les tables `customers` et `orders` vues plus haut : une jointure te permet d'afficher le nom des clients à côté de leurs commandes.

### INNER JOIN — uniquement les lignes qui correspondent

```sql
SELECT customers.name, orders.total
FROM customers
INNER JOIN orders ON customers.id = orders.customer_id;
```

Ne renvoie que les lignes qui ont une correspondance dans les deux tables. Les clients sans commande n'apparaissent pas.

### LEFT JOIN — toutes les lignes de gauche, plus les correspondances

```sql
SELECT customers.name, orders.total
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id;
```

Renvoie chaque client, avec ses commandes s'il en a. Les clients sans commande apparaissent avec `NULL` dans les colonnes de commande.

### RIGHT JOIN — toutes les lignes de droite, plus les correspondances

Identique à LEFT JOIN, mais dans l'autre sens. Toutes les commandes sont renvoyées, même si leur client n'existe pas.

### FULL JOIN — tout, des deux tables

Renvoie toutes les lignes des deux tables, avec `NULL` quand il n'y a pas de correspondance d'un côté ou de l'autre.

### Utiliser des alias de tables pour garder des requêtes lisibles

```sql
SELECT c.name, o.total
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id
WHERE o.total > 100;
```
> Des alias comme `c` et `o` évitent de la frappe et rendent les requêtes bien plus propres.

### Joindre plus de deux tables

```sql
SELECT c.name, p.name AS product, oi.quantity
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

---

## Sous-requêtes et CTE

### Sous-requêtes — une requête dans une requête

```sql
-- Récupérer les clients qui ont passé au moins une commande
SELECT name FROM customers
WHERE id IN (
  SELECT DISTINCT customer_id FROM orders
);
```

La requête interne s'exécute en premier, et son résultat est utilisé par la requête externe.

### CTE (*Common Table Expressions*, « expressions de table commune ») — des sous-requêtes plus propres et réutilisables

Une CTE revient à donner un nom à une sous-requête. On la définit avec `WITH`.

```sql
WITH big_orders AS (
  SELECT customer_id, SUM(total) AS total_spent
  FROM orders
  GROUP BY customer_id
  HAVING SUM(total) > 200
)
SELECT customers.name, big_orders.total_spent
FROM customers
JOIN big_orders ON customers.id = big_orders.customer_id;
```

Les CTE rendent les requêtes complexes bien plus faciles à lire et à déboguer. Tu peux même en définir plusieurs :

```sql
WITH
  active_customers AS (
    SELECT id FROM customers WHERE active = true
  ),
  recent_orders AS (
    SELECT * FROM orders WHERE created_at >= '2024-01-01'
  )
SELECT ac.id, ro.total
FROM active_customers ac
JOIN recent_orders ro ON ac.id = ro.customer_id;
```

---

## Fonctions de fenêtrage

Les fonctions de fenêtrage (*window functions*) effectuent des calculs *sur des lignes liées entre elles* sans les réduire à un seul résultat comme le fait `GROUP BY`.

Pense-y ainsi : « pour chaque ligne, regarde une fenêtre de lignes liées et calcule quelque chose ».

### ROW_NUMBER — attribuer un numéro séquentiel

```sql
SELECT name, total,
  ROW_NUMBER() OVER (ORDER BY total DESC) AS rank
FROM orders;
```

### RANK et DENSE_RANK — classer avec des ex æquo

```sql
-- RANK : les ex æquo ont le même rang, le rang suivant saute un numéro (1, 1, 3)
-- DENSE_RANK : les ex æquo ont le même rang, le rang suivant ne saute rien (1, 1, 2)
SELECT name, score,
  RANK() OVER (ORDER BY score DESC) AS rank,
  DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM results;
```

### PARTITION BY — réinitialiser la fenêtre pour chaque groupe

```sql
-- Classer les commandes de chaque client indépendamment
SELECT customer_id, total,
  RANK() OVER (PARTITION BY customer_id ORDER BY total DESC) AS rank_within_customer
FROM orders;
```

### LAG et LEAD — regarder la ligne précédente / suivante

```sql
-- Comparer le total de chaque commande à celui de la commande précédente
SELECT id, total,
  LAG(total, 1) OVER (ORDER BY created_at) AS previous_total
FROM orders;
```

### Total cumulé

```sql
SELECT created_at, total,
  SUM(total) OVER (ORDER BY created_at) AS running_total
FROM orders;
```

---

## Modifier les données — INSERT, UPDATE, DELETE

### INSERT — ajouter de nouvelles lignes

```sql
-- Ajouter un client
INSERT INTO customers (name, email, city)
VALUES ('Sofia Diaz', 'sofia@example.com', 'Barcelona');

-- Ajouter plusieurs lignes d'un coup
INSERT INTO customers (name, email, city)
VALUES
  ('Ravi Kumar', 'ravi@example.com', 'Mumbai'),
  ('Emma Clarke', 'emma@example.com', 'Dublin');
```

### UPDATE — modifier des lignes existantes

```sql
-- Mettre à jour l'e-mail d'un client précis
UPDATE customers
SET email = 'newemail@example.com'
WHERE id = 1;

-- Mettre à jour plusieurs colonnes à la fois
UPDATE customers
SET email = 'new@example.com', city = 'Seville'
WHERE id = 1;
```

> ⚠️ **Utilise toujours WHERE avec UPDATE.** Sans lui, tu modifies absolument toutes les lignes.

### DELETE — supprimer des lignes

```sql
-- Supprimer un client
DELETE FROM customers WHERE id = 3;

-- Supprimer tous les clients d'une ville donnée
DELETE FROM customers WHERE city = 'London';
```

> ⚠️ **Utilise toujours WHERE avec DELETE.** Sans lui, tu vides entièrement la table.

---

## Créer et gérer des tables

### CREATE TABLE — construire une nouvelle table

```sql
CREATE TABLE products (
  id         SERIAL PRIMARY KEY,
  name       VARCHAR(200) NOT NULL,
  price      DECIMAL(10, 2) NOT NULL,
  stock      INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Types de données courants

| Type              | À utiliser pour                                   |
|-------------------|---------------------------------------------------|
| `INT`             | Nombres entiers (1, 42, -7)                       |
| `SERIAL`          | Entier auto-incrémenté (idéal pour les identifiants) |
| `DECIMAL(10, 2)`  | Nombres précis — montants, mesures                |
| `FLOAT`           | Nombres décimaux approximatifs                    |
| `VARCHAR(n)`      | Texte de n caractères maximum                     |
| `TEXT`            | Texte de longueur illimitée                       |
| `BOOLEAN`         | Vrai ou faux                                      |
| `DATE`            | Date uniquement (2024-03-15)                      |
| `TIMESTAMP`       | Date et heure                                     |

### Contraintes — des règles pour tes données

```sql
CREATE TABLE users (
  id       SERIAL PRIMARY KEY,          -- unique, non nul, auto-incrémenté
  email    VARCHAR(255) UNIQUE NOT NULL, -- doit être unique, ne peut pas être vide
  age      INT CHECK (age >= 0),         -- doit respecter cette condition
  role     VARCHAR(50) DEFAULT 'viewer', -- valeur par défaut si aucune n'est fournie
  team_id  INT REFERENCES teams(id)     -- clé étrangère vers la table teams
);
```

### ALTER TABLE — modifier une table existante

```sql
-- Ajouter une nouvelle colonne
ALTER TABLE customers ADD COLUMN phone VARCHAR(20);

-- Supprimer une colonne
ALTER TABLE customers DROP COLUMN phone;

-- Changer le type de données d'une colonne
ALTER TABLE customers ALTER COLUMN name TYPE TEXT;

-- Renommer une colonne
ALTER TABLE customers RENAME COLUMN name TO full_name;
```

### DROP TABLE — supprimer entièrement une table

```sql
-- Version sûre — pas d'erreur si la table n'existe pas
DROP TABLE IF EXISTS old_logs;
```

> ⚠️ Cela supprime la table ET toutes ses données, de façon définitive.

---

## Fonctions intégrées utiles

### Fonctions de chaînes de caractères

```sql
UPPER('hello')           -- 'HELLO'
LOWER('WORLD')           -- 'world'
LENGTH('SQL')            -- 3
TRIM('  hello  ')        -- 'hello'
LTRIM('  hello')         -- 'hello' (supprime seulement à gauche)
RTRIM('hello  ')         -- 'hello' (supprime seulement à droite)
CONCAT('Hello', ' ', 'World')  -- 'Hello World'
'Hello' || ' ' || 'World'      -- 'Hello World' (raccourci)
SUBSTRING('Hello World', 1, 5) -- 'Hello'
REPLACE('Hello World', 'World', 'SQL') -- 'Hello SQL'
POSITION('lo' IN 'Hello')      -- 4
```

### Fonctions numériques

```sql
ROUND(3.14159, 2)   -- 3.14
CEIL(4.2)           -- 5
FLOOR(4.9)          -- 4
ABS(-42)            -- 42
MOD(10, 3)          -- 1 (reste de la division)
POWER(2, 8)         -- 256
```

### Fonctions de dates

```sql
NOW()                            -- date et heure actuelles
CURRENT_DATE                     -- date du jour
CURRENT_TIME                     -- heure actuelle

DATE_PART('year', '2024-03-15')  -- 2024
DATE_PART('month', created_at)   -- numéro du mois

-- Ajouter / retrancher du temps
created_at + INTERVAL '7 days'
created_at - INTERVAL '1 month'

-- Différence entre deux dates
AGE('2024-12-31', '2024-01-01')  -- 11 mois 30 jours
```

### Gérer les valeurs NULL

```sql
-- COALESCE : renvoie la première valeur non nulle
COALESCE(phone, email, 'aucun contact')

-- NULLIF : renvoie NULL si deux valeurs sont égales (évite la division par zéro)
NULLIF(denominator, 0)

-- IFNULL / ISNULL (MySQL / SQL Server)
IFNULL(column, 'valeur par défaut')
```

### Conversion de type (*casting*)

```sql
-- Convertir une valeur vers un autre type
CAST('42' AS INT)
CAST(price AS VARCHAR)
'42'::INT       -- raccourci PostgreSQL
```

---

## L'expression CASE

`CASE` est la façon qu'a le SQL de faire du if/else à l'intérieur d'une requête.

### CASE simple

```sql
SELECT name,
  CASE grade
    WHEN 'A' THEN 'Excellent'
    WHEN 'B' THEN 'Bien'
    WHEN 'C' THEN 'Moyen'
    ELSE 'À améliorer'
  END AS feedback
FROM students;
```

### CASE avec conditions (plus flexible)

```sql
SELECT name, total,
  CASE
    WHEN total >= 500 THEN 'VIP'
    WHEN total >= 100 THEN 'Régulier'
    ELSE 'Occasionnel'
  END AS customer_tier
FROM orders;
```

### CASE dans une agrégation

```sql
-- Compter combien de commandes tombent dans chaque catégorie
SELECT
  COUNT(CASE WHEN total >= 500 THEN 1 END) AS vip_orders,
  COUNT(CASE WHEN total >= 100 THEN 1 END) AS regular_orders,
  COUNT(CASE WHEN total < 100 THEN 1 END)  AS small_orders
FROM orders;
```

---

## Opérations ensemblistes

Elles combinent les résultats de deux requêtes distinctes en un seul.

> Les deux requêtes doivent renvoyer le même nombre de colonnes, avec des types de données compatibles.

### UNION — combiner et supprimer les doublons

```sql
SELECT email FROM customers
UNION
SELECT email FROM newsletter_subscribers;
```

### UNION ALL — combiner et conserver les doublons

```sql
SELECT email FROM customers
UNION ALL
SELECT email FROM newsletter_subscribers;
```
> Plus rapide que `UNION`, car il n'a pas besoin de chercher les doublons.

### INTERSECT — les lignes présentes dans les deux

```sql
SELECT email FROM customers
INTERSECT
SELECT email FROM newsletter_subscribers;
-- Ne renvoie que les e-mails présents dans les DEUX listes
```

### EXCEPT — les lignes de la première requête absentes de la seconde

```sql
SELECT email FROM customers
EXCEPT
SELECT email FROM newsletter_subscribers;
-- Renvoie les clients qui ne sont PAS abonnés à la newsletter
```

---

## Index et conseils de performance

### Qu'est-ce qu'un index ?

Un index, c'est comme l'index à la fin d'un livre : au lieu de feuilleter chaque page, tu vas directement à ce qu'il te faut. Les index rendent les requêtes beaucoup plus rapides sur les grosses tables.

```sql
-- Créer un index simple sur la colonne email
CREATE INDEX idx_customers_email ON customers(email);

-- Index unique — impose aussi l'unicité
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Index composite — couvre les requêtes qui filtrent sur les deux colonnes
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at);

-- Supprimer un index
DROP INDEX idx_customers_email;
```

### Quand utiliser des index

**À indexer :**
- Les colonnes fréquemment utilisées dans les clauses `WHERE`
- Les colonnes utilisées dans les conditions de `JOIN`
- Les colonnes utilisées dans `ORDER BY` sur de grosses tables
- Les colonnes de clés étrangères

**Ne pas abuser des index :**
- Les index ralentissent `INSERT`, `UPDATE` et `DELETE`
- Les petites tables n'ont pas besoin d'index — un parcours complet suffit
- Les colonnes qui ont très peu de valeurs distinctes (comme `true/false`) en profitent rarement

### Conseils généraux de performance

```sql
-- Évite SELECT * en production — ne récupère que ce dont tu as besoin
SELECT id, name FROM users;  -- ✅ Mieux que SELECT *

-- Filtre tôt avec WHERE pour réduire le nombre de lignes avant une jointure
SELECT * FROM orders WHERE created_at > '2024-01-01';

-- Utilise LIMIT quand tu n'as besoin que de quelques lignes
SELECT * FROM logs ORDER BY created_at DESC LIMIT 100;

-- EXPLAIN affiche le plan d'exécution de la requête (idéal pour déboguer les requêtes lentes)
EXPLAIN SELECT * FROM orders WHERE customer_id = 5;
```

---

## Erreurs courantes à éviter

### 1. Utiliser `= NULL` au lieu de `IS NULL`

```sql
-- ❌ Faux — cela ne correspondra jamais à rien
WHERE phone = NULL

-- ✅ Correct
WHERE phone IS NULL
```

### 2. Oublier WHERE sur un UPDATE ou un DELETE

```sql
-- ❌ Supprime TOUTES les lignes de la table !
DELETE FROM users;

-- ✅ Ne supprime que la ligne visée
DELETE FROM users WHERE id = 42;
```

### 3. Confondre WHERE et HAVING

```sql
-- ❌ Faux — on ne peut pas utiliser un agrégat dans WHERE
SELECT dept, COUNT(*) FROM employees
WHERE COUNT(*) > 5
GROUP BY dept;

-- ✅ Correct — utilise HAVING pour les agrégats
SELECT dept, COUNT(*) FROM employees
GROUP BY dept
HAVING COUNT(*) > 5;
```

### 4. Oublier que NULL est contagieux

```sql
-- Tout calcul impliquant NULL renvoie NULL
SELECT 5 + NULL;    -- renvoie NULL
SELECT NULL = NULL; -- renvoie NULL (et pas TRUE !)
```

### 5. Colonnes incohérentes avec GROUP BY

```sql
-- ❌ Faux — les colonnes non agrégées doivent figurer dans GROUP BY
SELECT name, dept, COUNT(*) FROM employees GROUP BY dept;

-- ✅ Correct
SELECT dept, COUNT(*) FROM employees GROUP BY dept;
```

---

## Aide-mémoire rapide

```sql
-- ── LIRE DES DONNÉES ──────────────────────────────────
SELECT col1, col2 FROM table;             -- récupérer des colonnes
SELECT * FROM table;                      -- récupérer toutes les colonnes
SELECT DISTINCT col FROM table;           -- valeurs uniques uniquement

-- ── FILTRER ───────────────────────────────────────────
WHERE col = 'value'                       -- égal
WHERE col != 'value'                      -- différent
WHERE col > 10 AND col < 100              -- intervalle
WHERE col BETWEEN 10 AND 100              -- intervalle (bornes incluses)
WHERE col IN ('a', 'b', 'c')             -- correspond à une liste
WHERE col LIKE '%pattern%'               -- motif de texte
WHERE col IS NULL                         -- valeur manquante
WHERE col IS NOT NULL                     -- valeur présente

-- ── TRIER ET LIMITER ──────────────────────────────────
ORDER BY col ASC                          -- trier A→Z, 1→9
ORDER BY col DESC                         -- trier Z→A, 9→1
LIMIT 10                                  -- 10 premières lignes
LIMIT 10 OFFSET 20                        -- lignes 21 à 30

-- ── AGRÉGATION ────────────────────────────────────────
COUNT(*), COUNT(col), SUM(col)
AVG(col), MIN(col), MAX(col)

GROUP BY col                              -- regrouper les lignes
HAVING COUNT(*) > 5                       -- filtrer les groupes

-- ── JOINTURES ─────────────────────────────────────────
INNER JOIN t ON a.id = t.a_id            -- lignes correspondantes
LEFT JOIN  t ON a.id = t.a_id            -- tout à gauche + correspondances
RIGHT JOIN t ON a.id = t.a_id            -- tout à droite + correspondances
FULL JOIN  t ON a.id = t.a_id            -- toutes les lignes des deux côtés

-- ── CTE ───────────────────────────────────────────────
WITH cte_name AS (
  SELECT ...
)
SELECT * FROM cte_name;

-- ── MODIFIER DES DONNÉES ──────────────────────────────
INSERT INTO t (col1, col2) VALUES ('a', 'b');
UPDATE t SET col = 'val' WHERE id = 1;
DELETE FROM t WHERE id = 1;

-- ── TABLES ────────────────────────────────────────────
CREATE TABLE t (id SERIAL PRIMARY KEY, name VARCHAR(100));
ALTER TABLE t ADD COLUMN col INT;
DROP TABLE IF EXISTS t;

-- ── CASE ──────────────────────────────────────────────
CASE
  WHEN condition THEN result
  ELSE default_result
END

-- ── OPÉRATIONS ENSEMBLISTES ───────────────────────────
query_A UNION query_B                     -- combiner, sans doublons
query_A UNION ALL query_B                 -- combiner, avec doublons
query_A INTERSECT query_B                 -- lignes présentes dans les deux
query_A EXCEPT query_B                    -- lignes de A absentes de B
```

---

*Bonnes requêtes ! La meilleure façon d'apprendre le SQL, c'est de pratiquer — essaie ces exemples sur une vraie base de données.*
