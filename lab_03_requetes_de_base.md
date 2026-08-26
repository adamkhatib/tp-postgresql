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
