# laboratoire 2 : ddl & dml - conception de schéma et manipulation de données

pour ce tp, je continue à utiliser postgresql 17 sur ma vm debian 13 et le rôle labuser créé pendant le lab 1.

## préparation de la base pagila

le laboratoire demande d'utiliser la base d'exemple pagila. elle n'est pas encore présente sur ma vm, je récupère donc la version officielle 3.1.0, compatible avec le schéma classique présenté dans le sujet et avec postgresql 17.

https://github.com/devrimgunduz/pagila/releases/tag/pagila-v3.1.0

le dump officiel force normalement les objets à appartenir au rôle postgres. je filtre uniquement les lignes OWNER TO postgres pendant l'import afin que les tables appartiennent à labuser.

```bash
sudo -u postgres psql -d postgres \
  -c "create database pagila owner labuser;"

sed '/OWNER TO postgres;/d' pagila-schema.sql |
  psql -h 127.0.0.1 -U labuser -d pagila

psql -h 127.0.0.1 -U labuser -d pagila \
  -f pagila-data.sql
```

je vérifie ensuite le chargement.

```text
base     : pagila
rôle     : labuser
tables   : 22
actors   : 200
customers: 599
films    : 1000
rentals  : 16044
payments : 16049
```

## exercice 2.1 : explorer le schéma pagila

je me connecte à pagila puis je liste les tables.

```console
$ psql -h 127.0.0.1 -U labuser -d pagila
pagila=> \dt
```

```text
                      liste des relations
 schéma |       nom        |        type        | propriétaire
--------+------------------+--------------------+--------------
 public | actor            | table              | labuser
 public | address          | table              | labuser
 public | category         | table              | labuser
 public | city             | table              | labuser
 public | country          | table              | labuser
 public | customer         | table              | labuser
 public | film             | table              | labuser
 public | film_actor       | table              | labuser
 public | film_category    | table              | labuser
 public | inventory        | table              | labuser
 public | language         | table              | labuser
 public | payment          | table partitionnée | labuser
 public | payment_p2022_01 | table              | labuser
 public | payment_p2022_02 | table              | labuser
 public | payment_p2022_03 | table              | labuser
 public | payment_p2022_04 | table              | labuser
 public | payment_p2022_05 | table              | labuser
 public | payment_p2022_06 | table              | labuser
 public | payment_p2022_07 | table              | labuser
 public | rental           | table              | labuser
 public | staff            | table              | labuser
 public | store            | table              | labuser
(22 lignes)
```

je décris ensuite la table Customer:

```console
pagila=> \d customer
```

- customer_id est la clé primaire et elle est généré par une séquence 
- store_id et address_id sont des clés étrangères 
- activebool possède la valeur par défaut true 
- create_date utilise current_date 
- last_update est mis à jour par le trigger last_updated 
- la suppression d'une adresse / d'un magasin référencé est refusé par delete restrict

## exercice 2.2 : analyser les relations

je regarde le catalogue information_schema pour trouver les relations principales.

```sql
select
    tc.table_name as table_enfant,
    kcu.column_name as colonne_enfant,
    ccu.table_name as table_parent,
    ccu.column_name as colonne_parent,
    rc.update_rule,
    rc.delete_rule
from information_schema.table_constraints tc
join information_schema.key_column_usage kcu
  on tc.constraint_name = kcu.constraint_name
 and tc.constraint_schema = kcu.constraint_schema
join information_schema.constraint_column_usage ccu
  on ccu.constraint_name = tc.constraint_name
 and ccu.constraint_schema = tc.constraint_schema
join information_schema.referential_constraints rc
  on rc.constraint_name = tc.constraint_name
 and rc.constraint_schema = tc.constraint_schema
where tc.constraint_type = 'FOREIGN KEY'
  and tc.table_schema = 'public'
  and tc.table_name in ('customer', 'rental', 'payment', 'film_actor')
order by table_enfant, colonne_enfant;
```

```text
 table_enfant | colonne_enfant | table_parent | colonne_parent | update_rule | delete_rule
--------------+----------------+--------------+----------------+-------------+------------
 customer     | address_id     | address      | address_id     | CASCADE     | RESTRICT
 customer     | store_id       | store        | store_id       | CASCADE     | RESTRICT
 film_actor   | actor_id       | actor        | actor_id       | CASCADE     | RESTRICT
 film_actor   | film_id        | film         | film_id        | CASCADE     | RESTRICT
 rental       | customer_id    | customer     | customer_id    | CASCADE     | RESTRICT
 rental       | inventory_id   | inventory    | inventory_id   | CASCADE     | RESTRICT
 rental       | staff_id       | staff        | staff_id       | CASCADE     | RESTRICT
(7 lignes)
```

je vois que restrict protège les tables parents qui sont encore référencés, cascade lance une suppression ou une mise à jour et set null garde la ligne enfant en supprimant seulement sa référence quand la colonne autorise.

sur cette version de pagila, payment est partitionnée. les clés étrangères vers rental se trouvent sur les partitions et non sur la table parent payment, ce qui explique qu'elles ne soient pas affichées par la requête précédente.

une clé étrangère ne crée pas automatiquement d'index sur la colonne de la table enfant. en revanche, une clé primaire ou une contrainte unique crée déjà son index. je tiens compte de cette différence dans le schéma blogapp pour ne pas créer d'index en double.

https://www.postgresql.org/docs/17/ddl-constraints.html

## types de données postgresql

pour créer le schéma, je garde les choix suivants :

- serial pour les identifiants auto-incrémentés ;
- varchar lorsque la longueur maximale est connue ;
- text pour les contenus sans taille maximale utile ;
- numeric pour les valeurs qui demandent une précision exacte ;
- boolean pour les états vrai ou faux ;
- date pour une date seule ;
- timestamptz pour les horodatages applicatifs ;
- jsonb, uuid, array et enum pour les besoins spécifiques à postgresql.

## exercice 2.3 : créer la table tags

d'abord je crée la base finale blogapp appartenant à labuser.

```sql
create database blogapp owner labuser;
```

je m'y connecte puis je crée la première version de tags.

```sql
create table tags (
    tag_id serial primary key,
    nom varchar(50) not null unique,
    slug varchar(50) not null unique,
    cree_le timestamptz default now()
);
```

```text
CREATE TABLE
                                          table « public.tags »
 colonne |           type           | null-able |              par défaut
---------+--------------------------+-----------+--------------------------------------
 tag_id  | integer                  | not null  | nextval('tags_tag_id_seq'::regclass)
 nom     | character varying(50)    | not null  |
 slug    | character varying(50)    | not null  |
 cree_le | timestamp with time zone |           | now()
```

## exercice 2.5 : modifier la table tags

j'ajoute les colonnes et ce qui est demandées et je renomme cree_le.

```sql
alter table tags add column description text;

alter table tags
add column compteur_utilisation integer default 0;

alter table tags
add constraint chk_compteur_nonnegatif
check (compteur_utilisation >= 0);

alter table tags
add constraint chk_tag_nom_longueur
check (length(nom) >= 2);

alter table tags
rename column cree_le to date_creation;
```

```text
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
```

la table contient maintenant tag_id, nom, slug, date_creation, description et compteur_utilisation.

## exercice 2.6 : pratiquer les opérations insert

### insertion d'un client pagila

je met un client de test avec store_id 1 et address_id 1 et j'utilise returning pour récupérer son identifiant.

```sql
insert into customer (
    store_id, first_name, last_name, email,
    address_id, activebool, active
)
values (
    1, 'LAB', 'DEUX', 'lab02.client@example.com',
    1, true, 1
)
returning customer_id, store_id, first_name,
          last_name, email, address_id;
```

```text
 customer_id | store_id | first_name | last_name |          email           | address_id
-------------+----------+------------+-----------+--------------------------+-----------
         600 |        1 | LAB        | DEUX      | lab02.client@example.com |          1
(1 ligne)
```

### insertion de trois films

le sujet indique language_id 1 pour le français. je vérifie d'abord la table language au lieu de reprendre cette valeur sans contrôle.

```sql
select language_id, btrim(name) as langue
from language
where language_id = 1
   or btrim(name) = 'French'
order by language_id;
```

```text
 language_id | langue
-------------+---------
           1 | English
           5 | French
(2 lignes)
```

le français correspond donc à language_id 5 dans la version de pagila installée. je met trois films en une instruction et récupére leurs informations.

```sql
insert into film (
    title, description, release_year, language_id,
    rental_duration, rental_rate, length,
    replacement_cost, rating
)
values
    ('LAB02 DDL', 'Film de test sur le DDL',
     2026, 5, 3, 2.99, 90, 19.99, 'PG'),
    ('LAB02 DML', 'Film de test sur le DML',
     2026, 5, 4, 3.99, 95, 19.99, 'PG'),
    ('LAB02 RETURNING', 'Film de test sur RETURNING',
     2026, 5, 5, 4.99, 100, 24.99, 'G')
returning film_id, title, release_year,
          language_id, rental_rate, length, rating;
```

```text
 film_id |      title      | release_year | language_id | rental_rate | length | rating
---------+-----------------+--------------+-------------+-------------+--------+-------
    1001 | LAB02 DDL       |         2026 |           5 |        2.99 |     90 | PG
    1002 | LAB02 DML       |         2026 |           5 |        3.99 |     95 | PG
    1003 | LAB02 RETURNING |         2026 |           5 |        4.99 |    100 | G
(3 lignes)
```

## exercice 2.7 : pratiquer les opérations update

je modifie l'adresse e-mail du premier client de pagila.

```sql
update customer
set email = 'client1.lab02@example.com'
where customer_id = 1
returning customer_id, first_name, last_name,
          email, last_update;
```

```text
 customer_id | first_name | last_name |           email
-------------+------------+-----------+---------------------------
           1 | MARY       | SMITH     | client1.lab02@example.com
(1 ligne)
```

la seconde consigne est ambiguë : rental_rate appartient à film et non à rental. modifier cette colonne accorderait donc la réduction à tous les futurs clients qui louent ces films, pas seulement aux clients du magasin 1.

je teste malgré tout la jointure demandée dans une transaction afin d'observer son effet, puis je fais rollback pour ne pas modifier les tarifs globaux de pagila. la sous-requête distinct évite aussi de réduire plusieurs fois le même film.

```sql
begin;

with films_eligibles as materialized (
    select distinct i.film_id
    from inventory i
    join rental r on r.inventory_id = i.inventory_id
    join customer c on c.customer_id = r.customer_id
    where c.store_id = 1
),
films_modifies as (
    update film f
    set rental_rate = round(f.rental_rate * 0.90, 2)
    from films_eligibles e
    where f.film_id = e.film_id
    returning f.film_id, f.rental_rate
)
select
    count(*) as films_modifies,
    min(rental_rate) as tarif_minimum,
    max(rental_rate) as tarif_maximum
from films_modifies;

rollback;

select count(*) as tarifs_reduits_restants
from film
where rental_rate in (0.89, 2.69, 4.49);
```

```text
 films_modifies | tarif_minimum | tarif_maximum
----------------+---------------+--------------
            957 |          0.89 |          4.49
(1 ligne)

ROLLBACK

 tarifs_reduits_restants
-------------------------
                       0
(1 ligne)
```

le modèle pagila ne possède pas de tarif propre à une location ou à un client. une vraie réduction limitée au magasin 1 demanderait une table de promotions ou un prix enregistré sur chaque location.

## exercice 2.8 : pratiquer les opérations delete

je supprime le client de test créé dans l'exercice 2.6.

```sql
delete from customer
where email = 'lab02.client@example.com'
returning customer_id, first_name, last_name, email;
```

```text
 customer_id | first_name | last_name |          email
-------------+------------+-----------+--------------------------
         600 | LAB        | DEUX      | lab02.client@example.com
(1 ligne)
```

### archivage des anciennes locations

je crée la table d'archive.

```sql
create table rental_archive
(like rental including all);
```

ma première tentative de déplacement est refusée car les paiements des partitions référencent toujours les locations. une seconde tentative qui essaie de mettre rental_id à null est également refusée parce que cette colonne est not null. les contraintes de cette version de pagila empêchent donc bien de supprimer les locations seules.

pour réaliser réellement l'archivage sans détruire la base de référence des prochains laboratoires, je crée une copie de travail.

```bash
sudo -u postgres createdb -O labuser pagila_lab02_archive

sudo -u postgres pg_dump --no-owner --no-privileges pagila |
  sudo -u postgres psql -d pagila_lab02_archive

psql -h 127.0.0.1 -U labuser -d pagila_lab02_archive
```

dans cette copie, j'archive d'abord les paiements liés dans une table permanente, puis les locations. les deux déplacements sont validés dans la même transaction.

```sql
begin;

create table payment_archive_lab02
(like payment including all);

with paiements_deplaces as (
    delete from payment p
    using rental r
    where p.rental_id = r.rental_id
      and r.rental_date < current_date - interval '2 years'
    returning p.*
)
insert into payment_archive_lab02
select * from paiements_deplaces;

with locations_deplacees as (
    delete from rental
    where rental_date < current_date - interval '2 years'
    returning *
)
insert into rental_archive
select * from locations_deplacees;

select
    (select count(*) from payment_archive_lab02)
        as paiements_archives,
    (select count(*) from rental_archive)
        as locations_archivees,
    (select count(*) from payment)
        as paiements_restants,
    (select count(*) from rental)
        as locations_restantes;

commit;
```

```text
 paiements_archives | locations_archivees | paiements_restants | locations_restantes
--------------------+---------------------+--------------------+---------------------
              16049 |               16044 |                  0 |                   0
(1 ligne)

COMMIT
```

l'archivage demandé est donc réellement effectué dans pagila_lab02_archive, tandis que la base pagila principale conserve ses locations et ses paiements.

je vérifie aussi le point sur truncate : sous postgresql, truncate peut être annulé par rollback lorsqu'il est exécuté dans une transaction. je ne le traite donc pas comme une opération non transactionnelle.

https://www.postgresql.org/docs/17/sql-truncate.html

## exercice 2.9 : concevoir le schéma blogapp

je crée le schéma final avec les six entités (utilisateurs, catégories, posts, tags, post_tags et commentaires.

les contraintes primary key et unique créent déjà leurs index btree. je ne rajoute donc pas un second index sur email, username ou les colonnes slug. de la même manière, la clé primaire de post_tags sur (post_id, tag_id) couvre déjà les recherches qui commencent par post_id.

### table)utilisateurs

```sql
create table utilisateurs (
    user_id serial primary key,
    username varchar(50) not null unique,
    email varchar(255) not null unique,
    mot_de_passe_hash varchar(255) not null,
    prenom varchar(50),
    nom varchar(50),
    bio text,
    avatar_url varchar(500),
    est_actif boolean default true,
    est_verifie boolean default false,
    cree_le timestamptz default now(),
    modifie_le timestamptz default now(),
    derniere_connexion timestamptz,
    constraint chk_username_longueur
        check (length(username) >= 3),
    constraint chk_email_format
        check (
            email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'
        )
);

create index idx_utilisateurs_actif
on utilisateurs(est_actif)
where est_actif = true;
```

### table catégories

```sql
create table categories (
    category_id serial primary key,
    nom varchar(100) not null unique,
    slug varchar(100) not null unique,
    description text,
    cree_le timestamptz default now(),
    constraint chk_categorie_nom_longueur
        check (length(nom) >= 2)
);
```

### table posts

```sql
create table posts (
    post_id serial primary key,
    user_id integer not null,
    category_id integer,
    titre varchar(200) not null,
    slug varchar(250) not null unique,
    extrait text,
    contenu text not null,
    image_vedette varchar(500),
    statut varchar(20) default 'brouillon',
    compteur_vues integer default 0,
    publie_le timestamptz,
    cree_le timestamptz default now(),
    modifie_le timestamptz default now(),

    constraint fk_posts_utilisateur
        foreign key (user_id)
        references utilisateurs(user_id)
        on delete cascade,

    constraint fk_posts_categorie
        foreign key (category_id)
        references categories(category_id)
        on delete set null,

    constraint chk_statut
        check (statut in ('brouillon', 'publie', 'archive')),

    constraint chk_date_publication
        check (statut != 'publie' or publie_le is not null),

    constraint chk_compteur_vues_nonnegative
        check (compteur_vues >= 0)
);

create index idx_posts_utilisateur on posts(user_id);
create index idx_posts_categorie on posts(category_id);
create index idx_posts_statut on posts(statut);
create index idx_posts_publie
on posts(publie_le desc)
where statut = 'publie';
create index idx_posts_cree on posts(cree_le desc);
```

### table tags

j'avais renommé cree_le en date_creation pendant l'exercice alter table. je le renomme à nouveau pour respecter le schéma final du sujet, puis j'ajoute les index.

```sql
alter table tags
rename column date_creation to cree_le;

create index idx_tags_compteur
on tags(compteur_utilisation desc);
```

### table de jonction post_tags

```sql
create table post_tags (
    post_id integer not null,
    tag_id integer not null,
    cree_le timestamptz default now(),
    primary key (post_id, tag_id),

    constraint fk_post_tags_post
        foreign key (post_id)
        references posts(post_id)
        on delete cascade,

    constraint fk_post_tags_tag
        foreign key (tag_id)
        references tags(tag_id)
        on delete cascade
);

create index idx_post_tags_tag on post_tags(tag_id);
```

### exercice 2.4 et table commentaires

les tables parentes existent maintenant. je peux créer commentaires avec toutes les contraintes demandées et la relation récursive parent_id. pour l'exercice 2.4, la suppression d'un utilisateur référencé est bien en restrict.

```sql
create table commentaires (
    comment_id serial primary key,
    post_id integer not null,
    user_id integer not null,
    parent_id integer,
    contenu text not null,
    est_approuve boolean default false,
    cree_le timestamptz default now(),
    modifie_le timestamptz default now(),

    constraint fk_commentaires_post
        foreign key (post_id)
        references posts(post_id)
        on delete cascade,

    constraint fk_commentaires_utilisateur
        foreign key (user_id)
        references utilisateurs(user_id)
        on delete restrict,

    constraint fk_commentaires_parent
        foreign key (parent_id)
        references commentaires(comment_id)
        on delete cascade,

    constraint chk_contenu_non_vide
        check (length(trim(contenu)) >= 1)
);

create index idx_commentaires_post
on commentaires(post_id, cree_le);

create index idx_commentaires_utilisateur
on commentaires(user_id, cree_le);

create index idx_commentaires_parent
on commentaires(parent_id);

create index idx_commentaires_approuve
on commentaires(est_approuve)
where est_approuve = true;
```

le schéma final donné plus loin dans le sujet demande ensuite cascade pour cette même relation. je fais donc cette évolution explicitement après avoir vérifié la contrainte restrict de l'exercice 2.4.

```sql
alter table commentaires
drop constraint fk_commentaires_utilisateur;

alter table commentaires
add constraint fk_commentaires_utilisateur
foreign key (user_id)
references utilisateurs(user_id)
on delete cascade;
```

```text
schéma final :
FOREIGN KEY (user_id)
REFERENCES utilisateurs(user_id)
ON DELETE CASCADE
```

je vérifie les tables et leur propriétaire.

```text
             liste des relations
 schéma |     nom      | type  | propriétaire
--------+--------------+-------+-------------
 public | categories   | table | labuser
 public | commentaires | table | labuser
 public | post_tags    | table | labuser
 public | posts        | table | labuser
 public | tags         | table | labuser
 public | utilisateurs | table | labuser
(6 lignes)
```

## données de démonstration

j'ajoute trois utilisateurs, trois catégories, six tags, deux posts initiaux et trois commentaires. j'utilise uniquement des valeurs de test.

```text
utilisateurs : alice, bob, charlie
catégories   : PostgreSQL, Programmation, Bases de donnees
tags         : PostgreSQL, SQL, Tutoriel, Web, Backend, Inutilise
posts initiaux : 2
commentaires   : 3
```

## défi du laboratoire 2

### ajouter dix posts

j'utilise generate_series pour ajouter les dix posts en une seule instruction.

```sql
insert into posts (
    user_id, category_id, titre, slug,
    extrait, contenu, statut, compteur_vues
)
select
    (numero % 3) + 1,
    (numero % 3) + 1,
    'Article Lab02 ' || numero,
    'article-lab02-' || numero,
    'Extrait de l article ' || numero,
    'Contenu pratique SQL et PostgreSQL numero ' || numero,
    'brouillon',
    numero * 10
from generate_series(1, 10) as numero;
```

```text
posts_ajoutes
--------------
           10
```

### publier tous les brouillons

je mets à jour le statut et la date de publication dans la même instruction.

```sql
with posts_publies as (
    update posts
    set statut = 'publie',
        publie_le = coalesce(publie_le, now()),
        modifie_le = now()
    where statut = 'brouillon'
    returning post_id
)
select count(*) as brouillons_publies
from posts_publies;
```

```text
 brouillons_publies
--------------------
                 11
```

### supprimer les tags inutilisés

je recalcule d'abord compteur_utilisation à partir de post_tags et je supprime les tags si le compteur vaut zéro.

```sql
update tags t
set compteur_utilisation = statistiques.nombre
from (
    select tag_id, count(*)::integer as nombre
    from post_tags
    group by tag_id
) as statistiques
where t.tag_id = statistiques.tag_id;

delete from tags
where compteur_utilisation = 0
returning tag_id, nom, slug;
```

```text
 tag_id |    nom    |   slug
--------+-----------+----------
      6 | Inutilise | inutilise
(1 ligne)
```

### pratiquer upsert

j'insère à nouveau le slug postgresql. le on conflict évite le doublon et met à jour la description existante.

```sql
insert into tags (nom, slug, description)
values (
    'PostgreSQL',
    'postgresql',
    'SGBD PostgreSQL - description mise a jour par UPSERT'
)
on conflict (slug)
do update set description = excluded.description
returning tag_id, nom, slug,
          description, compteur_utilisation;
```

```text
 tag_id |    nom     |    slug    | compteur_utilisation
--------+------------+------------+---------------------
      1 | PostgreSQL | postgresql |                   3
(1 ligne)
```

## requêtes d'application web

### profil utilisateur

j'exécute la requête de profil pour alice.

```text
 username | nom_complet  | total_posts | posts_publies | total_vues
----------+--------------+-------------+----------------+-----------
 alice    | Alice Martin |           4 |              4 |        330
(1 ligne)
```

### liste paginée des posts

la requête paginée retourne dix posts publiés avec leur auteur, leur catégorie et le nombre de commentaires.

```text
 post_id |             titre              | auteur  |    categorie
---------+--------------------------------+---------+------------------
      12 | Article Lab02 10               | bob     | Programmation
       2 | Construire une application web | bob     | Programmation
       3 | Article Lab02 1                | bob     | Programmation
       4 | Article Lab02 2                | charlie | Bases de donnees
       5 | Article Lab02 3                | alice   | PostgreSQL
       6 | Article Lab02 4                | bob     | Programmation
       7 | Article Lab02 5                | charlie | Bases de donnees
       8 | Article Lab02 6                | alice   | PostgreSQL
       9 | Article Lab02 7                | bob     | Programmation
      10 | Article Lab02 8                | charlie | Bases de donnees
(10 lignes)
```

### détail d'un post

la requête sur getting-started-postgresql retourne l'auteur, la catégorie et les tags ensembles.

```text
 post_id |             titre             | auteur | categorie  |         tags
---------+-------------------------------+--------+------------+----------------------
       1 | Bien demarrer avec PostgreSQL | alice  | PostgreSQL | {PostgreSQL,Tutoriel}
(1 ligne)
```

les deux commentaires approuvés sont également là.

```text
 comment_id |            contenu             | username | nom_commentateur
------------+--------------------------------+----------+-----------------
          1 | Excellent tutoriel PostgreSQL. | bob      | Bob Durand
          2 | Merci pour ce retour.          | alice    | Alice Martin
(2 lignes)
```

### recherche et filtres

la recherche du mot postgresql retourne 11 posts publiés. le filtre sur la catégorie programmation retourne 5 posts et le filtre sur le tag tutoriel retourne 3 posts.

```text
recherche postgresql : 11 résultats
catégorie programmation : 5 résultats
tag tutoriel : 3 résultats
```

## vérifications finales

je vérifie une dernière fois les trois bases.

```text
pagila :
  tables publiques : 23, rental_archive comprise
  films             : 1003
  locations         : 16044
  paiements         : 16049
  tarifs réduits restants : 0
  films de test en français : 3

pagila_lab02_archive :
  locations archivées : 16044
  paiements archivés : 16049
  locations restantes : 0
  paiements restants : 0

blogapp :
  utilisateurs : 3
  catégories   : 3
  posts        : 12
  tags         : 5
  post_tags    : 13
  commentaires : 3
  index redondants retirés : 6
  fk commentaires/utilisateur finale : cascade
```

le laboratoire 2 est terminé
