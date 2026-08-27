# laboratoire 3 : requêtes de base - select, joins et agrégations

pour ce tp, je continue à utiliser postgresql 17 sur ma vm debian 13, la base pagila préparée pendant le lab 2 et le rôle labuser.


## exercice 3.1 : pratique de select de base

### tâche 1 : sélectionner le titre, l'année de sortie et le tarif de location

je sélectionne uniquement les trois colonnes demandées dans la table 'film'.

```sql
select title, release_year, rental_rate
from film;
```

la requête retourne les 1003 films présents dans pagila. le total comprend les trois films de test ajoutés pendant le lab 2.

```text
            title            | release_year | rental_rate
-----------------------------+--------------+-------------
 STALLION SUNDANCE           |         2006 |        0.99
 BIKINI BORROWERS            |         2006 |        4.99
 GARDEN ISLAND               |         2006 |        4.99
 SAINTS BRIDE                |         2006 |        2.99
 LUCK OPUS                   |         2006 |        2.99
 ...                         |          ... |         ...
 LAB02 DDL                   |         2026 |        2.99
 LAB02 DML                   |         2026 |        3.99
 LAB02 RETURNING             |         2026 |        4.99
(1003 lignes)
```

### tâche 2 : afficher le nom complet et l'adresse e-mail en majuscules

j'utilise '||' pour rassembler le prénom et le nom avec un espace. la fonction 'upper' convertit l'adresse e-mail en majuscules.

```sql
select
    first_name || ' ' || last_name as full_name,
    upper(email) as email
from customer;
```

```text
       full_name       |                  email
-----------------------+------------------------------------------
 PATRICIA JOHNSON      | PATRICIA.JOHNSON@SAKILACUSTOMER.ORG
 LINDA WILLIAMS        | LINDA.WILLIAMS@SAKILACUSTOMER.ORG
 BARBARA JONES         | BARBARA.JONES@SAKILACUSTOMER.ORG
 ELIZABETH BROWN       | ELIZABETH.BROWN@SAKILACUSTOMER.ORG
 JENNIFER DAVIS        | JENNIFER.DAVIS@SAKILACUSTOMER.ORG
 ...                   | ...
 MARY SMITH            | CLIENT1.LAB02@EXAMPLE.COM
(599 lignes)
```

### tâche 3 : obtenir les classifications de films uniques

j'utilise 'distinct' pour supprimer les doublons. j'ajoute ensuite un tri pour montrer le résultat de manière lisible.

```sql
select distinct rating
from film
order by rating;
```

```text
 rating
--------
 G
 PG
 PG-13
 R
 NC-17
(5 lignes)
```

## exercice 3.2 : pratique de la clause where

### tâche 1 : trouver les films classés r avec un tarif supérieur à 3.00

je combine les deux conditions avec l'opérateur 'and'.

```sql
select title, rating, rental_rate
from film
where rating = 'R'
  and rental_rate > 3.00;
```

```text
         title         | rating | rental_rate
-----------------------+--------+-------------
 GUYS FALCON           | R      |        4.99
 SLACKER LIAISONS      | R      |        4.99
 DOUBTFIRE LABYRINTH   | R      |        4.99
 DESERT POSEIDON       | R      |        4.99
 KISSING DOLLS         | R      |        4.99
 MASSACRE USUAL        | R      |        4.99
 COMMANDMENTS EXPRESS  | R      |        4.99
 CROSSING DIVORCE      | R      |        4.99
 CROWDS TELEMARK       | R      |        4.99
 TRAIN BUNCH           | R      |        4.99
 ...                   | ...    |         ...
(65 lignes)
```

### tâche 2 : trouver les clients dont le nom commence par s

le sujet demande une recherche insensible à la casse. j'utilise donc 'ilike' avec le motif 'S%'.

```sql
select customer_id, first_name, last_name, email
from customer
where last_name ilike 'S%';
```

```text
 customer_id | first_name | last_name |                  email
-------------+------------+-----------+------------------------------------------
          34 | REBECCA    | SCOTT     | REBECCA.SCOTT@sakilacustomer.org
          51 | ALICE      | STEWART   | ALICE.STEWART@sakilacustomer.org
          52 | JULIE      | SANCHEZ   | JULIE.SANCHEZ@sakilacustomer.org
          75 | TAMMY      | SANDERS   | TAMMY.SANDERS@sakilacustomer.org
          92 | TINA       | SIMMONS   | TINA.SIMMONS@sakilacustomer.org
         105 | DAWN       | SULLIVAN  | DAWN.SULLIVAN@sakilacustomer.org
         126 | ELLEN      | SIMPSON   | ELLEN.SIMPSON@sakilacustomer.org
         127 | ELAINE     | STEVENS   | ELAINE.STEVENS@sakilacustomer.org
         144 | CLARA      | SHAW      | CLARA.SHAW@sakilacustomer.org
         158 | VERONICA   | STONE     | VERONICA.STONE@sakilacustomer.org
         ... | ...        | ...       | ...
           1 | MARY       | SMITH     | client1.lab02@example.com
(54 lignes)
```

### tâche 3 : trouver les films entre 100 et 120 minutes classés pg-13 ou r

'between' a les deux limites donc les films de 100 et de 120 minutes font partie du résultat. 'in' permet de tester les deux classifications demandées.

```sql
select title, length, rating
from film
where length between 100 and 120
  and rating in ('PG-13', 'R');
```

```text
         title          | length | rating
------------------------+--------+--------
 SEATTLE EXPECATIONS    |    110 | PG-13
 BANGER PINOCCHIO       |    113 | R
 GHOSTBUSTERS ELF       |    101 | R
 UNDEFEATED DALMATIONS  |    107 | PG-13
 ATTACKS HATE           |    113 | PG-13
 CROWDS TELEMARK        |    112 | R
 LOCK REAR              |    120 | R
 KARATE MOON            |    120 | PG-13
 DELIVERANCE MULHOLLAND |    100 | R
 RUGRATS SHAKESPEARE    |    109 | PG-13
 SCARFACE BANG          |    102 | PG-13
 STAGECOACH ARMAGEDDON  |    112 | R
 ...                    |    ... | ...
(71 lignes)
```

## exercice 3.3 : tri et pagination

### tâche 1 : obtenir les 10 films les plus chers à louer

je trie les films par tarif décroissant et je limite le résultat à 10 lignes. j'ajoute le titre comme second critère pour rendre le résultat lisible lorsque plusieurs films ont le même tarif.

```sql
select title, rental_rate
from film
order by rental_rate desc, title asc
limit 10;
```

```text
        title         | rental_rate
----------------------+-------------
 ACE GOLDFINGER       |        4.99
 AIRPLANE SIERRA      |        4.99
 AIRPORT POLLOCK      |        4.99
 ALADDIN CALENDAR     |        4.99
 ALI FOREVER          |        4.99
 AMELIE HELLFIGHTERS  |        4.99
 AMERICAN CIRCUS      |        4.99
 ANTHEM LUKE          |        4.99
 APACHE DIVINE        |        4.99
 APOCALYPSE FLAMINGOS |        4.99
(10 lignes)
```

### tâche 2 : obtenir la page 3 des clients, 20 par page, triés par nom de famille

la troisième page commence après les 40 premiers clients. j'utilise 'limit 20 offset 40'. les critères en plus permettent de départager les noms identiques.

```sql
select customer_id, first_name, last_name, email
from customer
order by last_name asc, first_name asc, customer_id asc
limit 20 offset 40;
```

```text
 customer_id | first_name |  last_name  |                email
-------------+------------+-------------+--------------------------------------
         448 | MIGUEL     | BETANCOURT  | MIGUEL.BETANCOURT@sakilacustomer.org
         344 | HENRY      | BILLINGSLEY | HENRY.BILLINGSLEY@sakilacustomer.org
         217 | AGNES      | BISHOP      | AGNES.BISHOP@sakilacustomer.org
         149 | VALERIE    | BLACK       | VALERIE.BLACK@sakilacustomer.org
         461 | DEREK      | BLAKELY     | DEREK.BLAKELY@sakilacustomer.org
         539 | MATHEW     | BOLIN       | MATHEW.BOLIN@sakilacustomer.org
         433 | DON        | BONE        | DON.BONE@sakilacustomer.org
         460 | LEON       | BOSTIC      | LEON.BOSTIC@sakilacustomer.org
         381 | BOBBY      | BOUDREAU    | BOBBY.BOUDREAU@sakilacustomer.org
         476 | DERRICK    | BOURQUE     | DERRICK.BOURQUE@sakilacustomer.org
         447 | CLIFFORD   | BOWENS      | CLIFFORD.BOWENS@sakilacustomer.org
         248 | CAROLINE   | BOWMAN      | CAROLINE.BOWMAN@sakilacustomer.org
         573 | BYRON      | BOX         | BYRON.BOX@sakilacustomer.org
         134 | EMMA       | BOYD        | EMMA.BOYD@sakilacustomer.org
         181 | ANA        | BRADLEY     | ANA.BRADLEY@sakilacustomer.org
         538 | TED        | BREAUX      | TED.BREAUX@sakilacustomer.org
         251 | VICKIE     | BREWER      | VICKIE.BREWER@sakilacustomer.org
         380 | RUSSELL    | BRINSON     | RUSSELL.BRINSON@sakilacustomer.org
          73 | BEVERLY    | BROOKS      | BEVERLY.BROOKS@sakilacustomer.org
         394 | CHRIS      | BROTHERS    | CHRIS.BROTHERS@sakilacustomer.org
(20 lignes)
```

### tâche 3 : obtenir les 5 films les plus longs en ignorant le premier

je trie les films par durée décroissante, j'ignore la première ligne avec 'offset 1' et j'en conserve cinq. plusieurs films ont la durée de 185 minutes
```sql
select title, length
from film
order by length desc, title asc
limit 5 offset 1;
```

```text
     title      | length
----------------+--------
 CONTROL ANTHEM |    185
 DARN FORRESTER |    185
 GANGS PRIDE    |    185
 HOME PITY      |    185
 MUSCLE BRIGHT  |    185
(5 lignes)
```

## exercice 3.4 : pratique d'inner join

### tâche 1 : lister tous les films avec leur catégorie

je relie 'film' à 'category' en passant par la table de jonction 'film_category'.

```sql
select f.title, c.name as category
from film f
join film_category fc on f.film_id = fc.film_id
join category c on fc.category_id = c.category_id
order by f.title asc;
```

les trois films de test ajoutés pendant le lab 2 n'ont pas de catégorie. l'inner join retourne donc les 1000 films du jeu de données pagila qui possèdent une correspondance.

```text
            title            |  category
-----------------------------+-------------
 ACADEMY DINOSAUR            | Documentary
 ACE GOLDFINGER              | Horror
 ADAPTATION HOLES            | Documentary
 AFFAIR PREJUDICE            | Horror
 AFRICAN EGG                 | Family
 AGENT TRUMAN                | Foreign
 AIRPLANE SIERRA             | Comedy
 AIRPORT POLLOCK             | Horror
 ALABAMA DEVIL               | Horror
 ALADDIN CALENDAR            | Sports
 ...                         | ...
 ZOOLANDER FICTION           | Children
 ZORRO ARK                   | Comedy
(1000 lignes)
```

### tâche 2 : trouver les paiements effectués par mary smith

je relie chaque paiement à son client puis je filtre sur le prénom et le nom demandés.

```sql
select
    p.payment_date,
    p.amount,
    c.first_name || ' ' || c.last_name as customer_name
from customer c
join payment p on c.customer_id = p.customer_id
where c.first_name = 'MARY'
  and c.last_name = 'SMITH'
order by p.payment_date asc;
```

```text
         payment_date          | amount | customer_name
-------------------------------+--------+---------------
 2022-01-28 21:10:06.039818+01 |   4.99 | MARY SMITH
 2022-01-29 14:03:02.267403+01 |   4.99 | MARY SMITH
 2022-02-04 03:24:08.306998+01 |   3.99 | MARY SMITH
 2022-02-05 23:36:58.72973+01  |   4.99 | MARY SMITH
 2022-02-19 20:34:21.993082+01 |   5.99 | MARY SMITH
 2022-02-24 22:41:18.230466+01 |   2.99 | MARY SMITH
 ...                           |    ... | ...
 2022-07-23 09:44:21.0171+02   |   4.99 | MARY SMITH
 2022-07-23 11:13:13.975359+02 |   0.99 | MARY SMITH
(32 lignes)
```

### tâche 3 : lister les films, leurs acteurs et leurs catégories

la requête utilise les deux tables de jonction 'film_actor' et 'film_category'. je limite ensuite le résultat à 20 lignes comme demandé.

```sql
select
    f.title,
    a.first_name || ' ' || a.last_name as actor_name,
    c.name as category
from film f
join film_actor fa on f.film_id = fa.film_id
join actor a on fa.actor_id = a.actor_id
join film_category fc on f.film_id = fc.film_id
join category c on fc.category_id = c.category_id
order by f.title asc, actor_name asc
limit 20;
```

```text
      title       |    actor_name    |  category
------------------+------------------+-------------
 ACADEMY DINOSAUR | CHRISTIAN GABLE  | Documentary
 ACADEMY DINOSAUR | JOHNNY CAGE      | Documentary
 ACADEMY DINOSAUR | LUCILLE TRACY    | Documentary
 ACADEMY DINOSAUR | MARY KEITEL      | Documentary
 ACADEMY DINOSAUR | MENA TEMPLE      | Documentary
 ACADEMY DINOSAUR | OPRAH KILMER     | Documentary
 ACADEMY DINOSAUR | PENELOPE GUINESS | Documentary
 ACADEMY DINOSAUR | ROCK DUKAKIS     | Documentary
 ACADEMY DINOSAUR | SANDRA PECK      | Documentary
 ACADEMY DINOSAUR | WARREN NOLTE     | Documentary
 ACE GOLDFINGER   | BOB FAWCETT      | Horror
 ACE GOLDFINGER   | CHRIS DEPP       | Horror
 ACE GOLDFINGER   | MINNIE ZELLWEGER | Horror
 ACE GOLDFINGER   | SEAN GUINESS     | Horror
 ADAPTATION HOLES | BOB FAWCETT      | Documentary
 ADAPTATION HOLES | CAMERON STREEP   | Documentary
 ADAPTATION HOLES | JULIANNE DENCH   | Documentary
 ADAPTATION HOLES | NICK WAHLBERG    | Documentary
 ADAPTATION HOLES | RAY JOHANSSON    | Documentary
 AFFAIR PREJUDICE | FAY WINSLET      | Horror
(20 lignes)
```

## exercice 3.5 : pratique des joins externes

### tâche 1 : trouver les acteurs qui n'ont jamais joué dans un film

le 'left join' conserve tous les acteurs. je recherche ensuite les lignes qui n'ont aucune correspondance dans 'film_actor'.

```sql
select a.actor_id, a.first_name, a.last_name
from actor a
left join film_actor fa on a.actor_id = fa.actor_id
where fa.actor_id is null
order by a.actor_id;
```

tous les acteurs présents sont associés à au moins un film.

```text
 actor_id | first_name | last_name
----------+------------+-----------
(0 ligne)
```

### tâche 2 : lister toutes les catégories avec leur nombre de films

j'utilise 'count(fc.film_id)' plutôt que 'count(*)' pour qu'une catégorie sans film soit bien comptée à zéro.

```sql
select
    c.name as category,
    count(fc.film_id) as film_count
from category c
left join film_category fc on c.category_id = fc.category_id
group by c.category_id, c.name
order by film_count desc, c.name asc;
```

```text
  category   | film_count
-------------+------------
 Sports      |         74
 Foreign     |         73
 Family      |         69
 Documentary |         68
 Animation   |         66
 Action      |         64
 New         |         63
 Drama       |         62
 Games       |         61
 Sci-Fi      |         61
 Children    |         60
 Comedy      |         58
 Classics    |         57
 Travel      |         57
 Horror      |         56
 Music       |         51
(16 lignes)
```

## exercice 3.6 : pratique des fonctions d'agrégation

### tâche 1 : calculer le total, la moyenne, le minimum et le maximum des paiements

je calcule les quatre valeurs dans une seule requête. j'arrondis uniquement la moyenne à deux décimales pour faciliter sa lecture.

```sql
select
    sum(amount) as total_amount,
    round(avg(amount), 2) as average_amount,
    min(amount) as min_amount,
    max(amount) as max_amount
from payment;
```

```text
 total_amount | average_amount | min_amount | max_amount
--------------+----------------+------------+------------
     67416.51 |           4.20 |       0.00 |      11.99
(1 ligne)
```

### tâche 2 : compter les clients distincts qui ont effectué des paiements

'count distinct' évite de compter plusieurs fois un client qui a fait plusieurs paiements.

```sql
select count(distinct customer_id) as paying_customers
from payment;
```

```text
 paying_customers
------------------
              599
(1 ligne)
```

## exercice 3.7 : pratique de group by et having

### tâche 1 : compter les films par catégorie

je regroupe les associations de 'film_category' par catégorie et je trie les comptages du plus grand au plus petit.

```sql
select
    c.name as category,
    count(fc.film_id) as film_count
from category c
join film_category fc on c.category_id = fc.category_id
group by c.category_id, c.name
order by film_count desc, c.name asc;
```

```text
  category   | film_count
-------------+------------
 Sports      |         74
 Foreign     |         73
 Family      |         69
 Documentary |         68
 Animation   |         66
 Action      |         64
 New         |         63
 Drama       |         62
 Games       |         61
 Sci-Fi      |         61
 Children    |         60
 Comedy      |         58
 Classics    |         57
 Travel      |         57
 Horror      |         56
 Music       |         51
(16 lignes)
```

### tâche 2 : trouver les acteurs qui ont joué dans plus de 30 films

je groupe les associations par acteur puis j'utilise 'having' pour conserver que les groupes dont le comptage dépasse 30

```sql
select
    a.actor_id,
    a.first_name,
    a.last_name,
    count(fa.film_id) as film_count
from actor a
join film_actor fa on a.actor_id = fa.actor_id
group by a.actor_id, a.first_name, a.last_name
having count(fa.film_id) > 30
order by film_count desc, a.last_name asc, a.first_name asc;
```

```text
 actor_id | first_name |  last_name  | film_count
----------+------------+-------------+------------
      107 | GINA       | DEGENERES   |         42
      102 | WALTER     | TORN        |         41
      198 | MARY       | KEITEL      |         40
      181 | MATTHEW    | CARREY      |         39
       23 | SANDRA     | KILMER      |         37
       81 | SCARLETT   | DAMON       |         36
      158 | VIVIEN     | BASINGER    |         35
       60 | HENRY      | BERRY       |         35
       37 | VAL        | BOLGER      |         35
      106 | GROUCHO    | DUNST       |         35
      144 | ANGELA     | WITHERSPOON |         35
       13 | UMA        | WOOD        |         35
      ... | ...        | ...         |        ...
(56 lignes)
```

### tâche 3 : produire le rapport de revenus mensuel pour 2007

je filtre  l'année 2007 puis je regroupe les paiements par mois avec 'date_trunc'.

```sql
select
    date_trunc('month', payment_date)::date as month,
    sum(amount) as total_revenue
from payment
where payment_date >= timestamp '2007-01-01'
  and payment_date < timestamp '2008-01-01'
group by month
order by month;
```

```text
 month | total_revenue
-------+---------------
(0 ligne)
```

je vérifie la plage de dates présente dans la base pour d'expliquer ce résultat vide.

```sql
select
    min(payment_date)::date as first_payment,
    max(payment_date)::date as last_payment
from payment;
```

```text
 first_payment | last_payment
---------------+--------------
 2022-01-23    | 2022-07-27
(1 ligne)
```

les données appartiennent donc à l'année 2022 et ne contiennent aucun paiement en 2007. je garde le filtre demandé par le sujet.

## exercice 3.8 : pratique des fonctions intégrées

### tâche 1 : formater les noms et les adresses e-mail des clients

j'utilise 'initcap' pour présenter les noms proprement.
'upper' pour l'adresse e-mail.
'||' pour construire la chaîne finale.

```sql
select
    initcap(last_name) || ', ' || initcap(first_name)
    || ' (' || upper(email) || ')' as customer_contact
from customer
order by customer_id
limit 10;
```

```text
                    customer_contact
---------------------------------------------------------
 Smith, Mary (CLIENT1.LAB02@EXAMPLE.COM)
 Johnson, Patricia (PATRICIA.JOHNSON@SAKILACUSTOMER.ORG)
 Williams, Linda (LINDA.WILLIAMS@SAKILACUSTOMER.ORG)
 Jones, Barbara (BARBARA.JONES@SAKILACUSTOMER.ORG)
 Brown, Elizabeth (ELIZABETH.BROWN@SAKILACUSTOMER.ORG)
 Davis, Jennifer (JENNIFER.DAVIS@SAKILACUSTOMER.ORG)
 Miller, Maria (MARIA.MILLER@SAKILACUSTOMER.ORG)
 Wilson, Susan (SUSAN.WILSON@SAKILACUSTOMER.ORG)
 Moore, Margaret (MARGARET.MOORE@SAKILACUSTOMER.ORG)
 Taylor, Dorothy (DOROTHY.TAYLOR@SAKILACUSTOMER.ORG)
(10 lignes)
```

### tâche 2 : calculer la durée de chaque location en jours

je remplace une date de retour manquante par l'heure actuelle avec coalesce. je transforme ensuite en secondes et je le convertir en jours

```sql
select
    rental_id,
    rental_date,
    return_date,
    round(
        extract(
            epoch from (
                coalesce(return_date, current_timestamp) - rental_date
            )
        ) / 86400,
        2
    ) as duration_days
from rental
order by rental_id;
```

```text
 rental_id |      rental_date       |      return_date       | duration_days
-----------+------------------------+------------------------+---------------
         1 | 2022-05-24 23:53:30+02 | 2022-05-26 23:04:30+02 |          1.97
         2 | 2022-05-24 23:54:33+02 | 2022-05-28 20:40:33+02 |          3.87
         3 | 2022-05-25 00:03:39+02 | 2022-06-01 23:12:39+02 |          7.96
         4 | 2022-05-25 00:04:41+02 | 2022-06-03 02:43:41+02 |          9.11
         5 | 2022-05-25 00:05:21+02 | 2022-06-02 05:33:21+02 |          8.23
         6 | 2022-05-25 00:08:07+02 | 2022-05-27 02:32:07+02 |          2.10
         7 | 2022-05-25 00:11:53+02 | 2022-05-29 21:34:53+02 |          4.89
         8 | 2022-05-25 00:31:46+02 | 2022-05-28 00:33:46+02 |          3.00
         9 | 2022-05-25 01:00:40+02 | 2022-05-28 01:22:40+02 |          3.02
        10 | 2022-05-25 01:02:21+02 | 2022-05-31 23:44:21+02 |          6.95
       ... | ...                    | ...                    |           ...
(16044 lignes)
```


### tâche 3 : catégoriser les films par durée

```sql
select
    title,
    length,
    case
        when length < 90 then 'court'
        when length <= 120 then 'moyen'
        else 'long'
    end as duration_category
from film
order by title;
```

```text
            title            | length | duration_category
-----------------------------+--------+-------------------
 ACADEMY DINOSAUR            |     86 | court
 ACE GOLDFINGER              |     48 | court
 ADAPTATION HOLES            |     50 | court
 AFFAIR PREJUDICE            |    117 | moyen
 AFRICAN EGG                 |    130 | long
 AGENT TRUMAN                |    169 | long
 AIRPLANE SIERRA             |     62 | court
 AIRPORT POLLOCK             |     54 | court
 ALABAMA DEVIL               |    114 | moyen
 ALADDIN CALENDAR            |     63 | court
 ...                         |    ... | ...
(1003 lignes)
```

## partie 9 : requêtes d'application web

```console
pagila=> \c blogapp
vous êtes maintenant connecté à la base de données 'blogapp'.
```

la base contient 3 utilisateurs, 3 catégories, 12 posts, 5 tags, 13 associations dans 'post_tags' et 3 commentaires.
²
### requête 1 : profil utilisateur

```sql
select
    u.user_id,
    u.username,
    u.email,
    u.prenom || ' ' || u.nom as nom_complet,
    u.bio,
    u.cree_le as membre_depuis,
    count(distinct p.post_id) as total_posts,
    count(distinct p.post_id)
        filter (where p.statut = 'publie') as posts_publies,
    coalesce(sum(p.compteur_vues), 0) as total_vues
from utilisateurs u
left join posts p on u.user_id = p.user_id
where u.username = 'alice'
group by u.user_id;
```

```text
 user_id | username |        email        | nom_complet  | total_posts | posts_publies | total_vues
---------+----------+---------------------+--------------+-------------+---------------+-----------
       1 | alice    | alice@blogapp.local | Alice Martin |           4 |             4 |        330
(1 ligne)
```

### requête 2 : liste paginée des posts

```sql
select
    p.post_id,
    p.titre,
    p.slug,
    p.extrait,
    p.publie_le,
    p.compteur_vues,
    u.username as auteur,
    c.nom as categorie,
    count(distinct cm.comment_id) as nombre_commentaires
from posts p
join utilisateurs u on p.user_id = u.user_id
left join categories c on p.category_id = c.category_id
left join commentaires cm on p.post_id = cm.post_id
where p.statut = 'publie'
  and p.publie_le <= current_timestamp
group by
    p.post_id, p.titre, p.slug, p.extrait,
    p.publie_le, p.compteur_vues, u.username, c.nom
order by p.publie_le desc, p.post_id desc
limit 10 offset 0;
```

```text
 post_id |      titre       | auteur  |    categorie     | nombre_commentaires
---------+------------------+---------+------------------+---------------------
      12 | Article Lab02 10 | bob     | Programmation    |                   0
      11 | Article Lab02 9  | alice   | PostgreSQL       |                   0
      10 | Article Lab02 8  | charlie | Bases de donnees |                   0
       9 | Article Lab02 7  | bob     | Programmation    |                   0
       8 | Article Lab02 6  | alice   | PostgreSQL       |                   0
       7 | Article Lab02 5  | charlie | Bases de donnees |                   0
       6 | Article Lab02 4  | bob     | Programmation    |                   0
       5 | Article Lab02 3  | alice   | PostgreSQL       |                   0
       4 | Article Lab02 2  | charlie | Bases de donnees |                   0
       3 | Article Lab02 1  | bob     | Programmation    |                   0
(10 lignes)
```

### requête 3 : détail d'un post et ses commentaires

je récupère d'abord le post correspondant au slug de l'url, son auteur, sa catégorie et ses tags
```sql
select
    p.post_id,
    p.titre,
    p.contenu,
    p.publie_le,
    p.compteur_vues,
    u.username as auteur,
    u.prenom || ' ' || u.nom as nom_auteur,
    c.nom as categorie,
    coalesce(
        array_agg(distinct t.nom order by t.nom)
            filter (where t.tag_id is not null),
        array[]::varchar[]
    ) as tags
from posts p
join utilisateurs u on p.user_id = u.user_id
left join categories c on p.category_id = c.category_id
left join post_tags pt on p.post_id = pt.post_id
left join tags t on pt.tag_id = t.tag_id
where p.slug = 'getting-started-postgresql'
  and p.statut = 'publie'
group by p.post_id, u.user_id, c.nom;
```

```text
 post_id |             titre             | compteur_vues | auteur | nom_auteur   | categorie  |         tags
---------+-------------------------------+---------------+--------+--------------+------------+-----------------------
       1 | Bien demarrer avec PostgreSQL |           150 | alice  | Alice Martin | PostgreSQL | {PostgreSQL,Tutoriel}
(1 ligne)
```

ensuite je récupère uniquement les commentaires approuvés du même post.

```sql
select
    cm.comment_id,
    cm.contenu,
    cm.cree_le,
    u.username,
    u.prenom || ' ' || u.nom as nom_commentateur
from commentaires cm
join utilisateurs u on cm.user_id = u.user_id
where cm.post_id = (
    select post_id
    from posts
    where slug = 'getting-started-postgresql'
)
  and cm.est_approuve = true
order by cm.cree_le asc, cm.comment_id asc;
```

```text
 comment_id |            contenu             | username | nom_commentateur
------------+--------------------------------+----------+------------------
          1 | Excellent tutoriel PostgreSQL. | bob      | Bob Durand
          2 | Merci pour ce retour.          | alice    | Alice Martin
(2 lignes)
```

### requête 4 : rechercher des posts

```sql
select
    p.post_id,
    p.titre,
    p.slug,
    p.extrait,
    u.username as auteur,
    p.publie_le
from posts p
join utilisateurs u on p.user_id = u.user_id
where p.statut = 'publie'
  and (
      p.titre ilike '%postgresql%'
      or p.contenu ilike '%postgresql%'
  )
order by p.publie_le desc, p.post_id desc
limit 20;
```

la recherche retourne les dix articles ce que l'on a vu au lab 2

```text
 post_id |             titre             | auteur
---------+-------------------------------+---------
      12 | Article Lab02 10              | bob
      11 | Article Lab02 9               | alice
      10 | Article Lab02 8               | charlie
       9 | Article Lab02 7               | bob
       8 | Article Lab02 6               | alice
       7 | Article Lab02 5               | charlie
       6 | Article Lab02 4               | bob
       5 | Article Lab02 3               | alice
       4 | Article Lab02 2               | charlie
       3 | Article Lab02 1               | bob
       1 | Bien demarrer avec PostgreSQL | alice
(11 lignes)
```

### requête 5 : posts par catégorie

le slug réel de la catégorie est 'programmation' pas 'programming' comme dans l'exemple du sujet
```sql
select
    p.post_id,
    p.titre,
    p.slug,
    p.extrait,
    p.publie_le,
    u.username as auteur
from posts p
join utilisateurs u on p.user_id = u.user_id
join categories c on p.category_id = c.category_id
where c.slug = 'programmation'
  and p.statut = 'publie'
order by p.publie_le desc, p.post_id desc
limit 20;
```

```text
 post_id |             titre              | auteur
---------+--------------------------------+--------
      12 | Article Lab02 10               | bob
       9 | Article Lab02 7                | bob
       6 | Article Lab02 4                | bob
       3 | Article Lab02 1                | bob
       2 | Construire une application web | bob
(5 lignes)
```

### requête 6 : posts par tag

le tag créé au lab 2 utilise le slug 'tutoriel'.

```sql
select
    p.post_id,
    p.titre,
    p.slug,
    p.extrait,
    p.publie_le,
    u.username as auteur
from posts p
join utilisateurs u on p.user_id = u.user_id
join post_tags pt on p.post_id = pt.post_id
join tags t on pt.tag_id = t.tag_id
where t.slug = 'tutoriel'
  and p.statut = 'publie'
order by p.publie_le desc, p.post_id desc
limit 20;
```

```text
 post_id |             titre             | auteur
---------+-------------------------------+--------
       8 | Article Lab02 6               | alice
       3 | Article Lab02 1               | bob
       1 | Bien demarrer avec PostgreSQL | alice
(3 lignes)
```

## partie 10 : requêtes analytiques

### analytique 1 : indicateurs clés de performance

je rassemble les indicateurs de l'application dans une seule ligne

```sql
select
    (select count(*) from utilisateurs where est_actif = true) as utilisateurs_actifs,
    (select count(*) from posts where statut = 'publie') as posts_publies,
    (select count(*) from posts where statut = 'brouillon') as brouillons,
    (select count(*) from commentaires where est_approuve = true) as commentaires_approuves,
    (select coalesce(sum(compteur_vues), 0) from posts) as total_vues,
    (
        select count(distinct user_id)
        from posts
        where cree_le > current_date - interval '30 days'
    ) as auteurs_actifs_30j;
```

```text
 utilisateurs_actifs | posts_publies | brouillons | commentaires_approuves | total_vues | auteurs_actifs_30j
---------------------+----------------+------------+-------------------------+------------+---------------------
                   3 |             12 |          0 |                       2 |        700 |                   3
(1 ligne)
```

### analytique 2 : meilleurs auteurs

je classe les auteurs selon le nombre total de vues obtenu avec leurs posts publiés.

```sql
select
    u.user_id,
    u.username,
    count(p.post_id) as nombre_posts,
    coalesce(sum(p.compteur_vues), 0) as total_vues,
    round(avg(p.compteur_vues), 2) as moyenne_vues,
    max(p.publie_le) as derniere_publication
from utilisateurs u
join posts p on u.user_id = p.user_id
where p.statut = 'publie'
group by u.user_id, u.username
order by total_vues desc, nombre_posts desc, u.username;
```

```text
 user_id | username | nombre_posts | total_vues | moyenne_vues |     derniere_publication
---------+----------+--------------+------------+---------------+-------------------------------
       2 | bob      |            5 |        220 |         44.00 | 2026-08-25 15:45:00+02
       1 | alice    |            4 |        330 |         82.50 | 2026-08-25 15:45:00+02
       3 | charlie  |            3 |        150 |         50.00 | 2026-08-25 15:45:00+02
(3 lignes)
```

### analytique 3 : performance des posts

je compare le nombre de vues et le nombre de commentaires de chaque post publié.

```sql
select
    p.post_id,
    p.titre,
    u.username as auteur,
    p.compteur_vues,
    count(c.comment_id) as nombre_commentaires,
    p.publie_le
from posts p
join utilisateurs u on p.user_id = u.user_id
left join commentaires c on p.post_id = c.post_id
where p.statut = 'publie'
group by
    p.post_id,
    p.titre,
    u.username,
    p.compteur_vues,
    p.publie_le
order by p.compteur_vues desc, p.post_id asc
limit 20;
```

```text
 post_id |             titre             | auteur  | compteur_vues | nombre_commentaires
---------+-------------------------------+---------+----------------+---------------------
       1 | Bien demarrer avec PostgreSQL | alice   |            150 |                   3
      12 | Article Lab02 10              | bob     |            100 |                   0
      11 | Article Lab02 9               | alice   |             90 |                   0
      10 | Article Lab02 8               | charlie |             80 |                   0
       9 | Article Lab02 7               | bob     |             70 |                   0
       8 | Article Lab02 6               | alice   |             60 |                   0
       7 | Article Lab02 5               | charlie |             50 |                   0
       6 | Article Lab02 4               | bob     |             40 |                   0
       5 | Article Lab02 3               | alice   |             30 |                   0
       4 | Article Lab02 2               | charlie |             20 |                   0
       3 | Article Lab02 1               | bob     |             10 |                   0
       2 | Construire une application web| bob     |              0 |                   0
(12 lignes)
```

### analytique 4 : répartition des posts par catégorie


```sql
select
    c.nom as categorie,
    count(p.post_id) as nombre_posts,
    round(
        100.0 * count(p.post_id)
        / nullif((select count(*) from posts where statut = 'publie'), 0),
        2
    ) as pourcentage
from categories c
left join posts p
    on c.category_id = p.category_id
   and p.statut = 'publie'
group by c.category_id, c.nom
order by nombre_posts desc, c.nom;
```

```text
     categorie      | nombre_posts | pourcentage
--------------------+--------------+-------------
 Programmation      |            5 |       41.67
 PostgreSQL         |            4 |       33.33
 Bases de donnees   |            3 |       25.00
(3 lignes)
```

### analytique 5 : publications au cours des douze derniers mois

```sql
select
    date_trunc('month', publie_le)::date as mois,
    count(*) as nombre_posts
from posts
where statut = 'publie'
  and publie_le >= current_date - interval '12 months'
group by mois
order by mois;
```

```text
    mois    | nombre_posts
------------+--------------
 2026-08-01 |           12
(1 ligne)
```

### analytique 6 : popularité des tags

je compare la quantité d'associations réellement présentes dans 'post_tags' avec le compteur enregistré dans la table 'tags'.

```sql
select
    t.nom as tag,
    count(pt.post_id) as nombre_utilisations,
    t.compteur_utilisation
from tags t
left join post_tags pt on t.tag_id = pt.tag_id
group by t.tag_id, t.nom, t.compteur_utilisation
order by nombre_utilisations desc, t.nom asc
limit 30;
```

```text
    tag     | nombre_utilisations | compteur_utilisation
------------+---------------------+-----------------------
 PostgreSQL |                   3 |                     3
 SQL        |                   3 |                     3
 Tutoriel   |                   3 |                     3
 Backend    |                   2 |                     2
 Web        |                   2 |                     2
(5 lignes)
```

## défi final

### 1. posts publiés au cours des sept derniers jours avec plus de 100 vues

```sql
select
    post_id,
    titre,
    compteur_vues,
    publie_le
from posts
where statut = 'publie'
  and publie_le >= current_timestamp - interval '7 days'
  and compteur_vues > 100
order by compteur_vues desc, post_id asc;
```

```text
 post_id |             titre             | compteur_vues |          publie_le
---------+-------------------------------+----------------+-------------------------------
       1 | Bien demarrer avec PostgreSQL |            150 | 2026-08-23 17:30:00+02
(1 ligne)
```

### 2. cinq posts ayant reçu le plus de commentaires

```sql
select
    p.post_id,
    p.titre,
    count(c.comment_id) as nombre_commentaires
from posts p
left join commentaires c on p.post_id = c.post_id
group by p.post_id, p.titre
order by nombre_commentaires desc, p.post_id asc
limit 5;
```

```text
 post_id |             titre              | nombre_commentaires
---------+--------------------------------+---------------------
       1 | Bien demarrer avec PostgreSQL  |                   3
       2 | Construire une application web |                   0
       3 | Article Lab02 1                |                   0
       4 | Article Lab02 2                |                   0
       5 | Article Lab02 3                |                   0
(5 lignes)
```

### 3. utilisateurs n'ayant jamais publié de post

je place la condition sur le statut dans la jointure afin de conserver les utilisateurs sans publication.

```sql
select
    u.user_id,
    u.username,
    u.email
from utilisateurs u
left join posts p
    on u.user_id = p.user_id
   and p.statut = 'publie'
where p.post_id is null
order by u.user_id;
```

```text
 user_id | username | email
---------+----------+-------
(0 ligne)
```

tous les utilisateurs présents ont donc déjà publié au moins un post.

### 4. moyenne de commentaires par post et par catégorie

la requête intermédiaire compte d'abord les commentaires de chaque post y compris lorsqu'il n'en possède aucun. la moyenne est ensuite calculée par catégorie.

```sql
with commentaires_par_post as (
    select
        c.category_id,
        c.nom as categorie,
        p.post_id,
        count(cm.comment_id) as nombre_commentaires
    from categories c
    join posts p on c.category_id = p.category_id
    left join commentaires cm on p.post_id = cm.post_id
    group by c.category_id, c.nom, p.post_id
)
select
    categorie,
    round(avg(nombre_commentaires), 2) as moyenne_commentaires_par_post
from commentaires_par_post
group by category_id, categorie
order by categorie;
```

```text
     categorie      | moyenne_commentaires_par_post
--------------------+-------------------------------
 Bases de donnees   |                          0.00
 PostgreSQL         |                          0.75
 Programmation      |                          0.00
(3 lignes)
```

### 5. tags utilisés dans plus de cinq posts

```sql
select
    t.tag_id,
    t.nom,
    count(pt.post_id) as nombre_posts
from tags t
join post_tags pt on t.tag_id = pt.tag_id
group by t.tag_id, t.nom
having count(pt.post_id) > 5
order by nombre_posts desc, t.nom asc;
```

```text
 tag_id | nom | nombre_posts
--------+-----+--------------
(0 ligne)
```

aucun tag n'est encore associé à plus de cinq posts dans les données actuelles.

le laboratoire 3 est  terminé.(