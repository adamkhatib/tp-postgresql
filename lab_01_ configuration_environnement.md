# laboratoire 1 : configuration de l'environnement et première base de données

pour ce tp, j'ai choisi de faire l'installation de postgresql via debian.

## installation et configuration

je commence par installer une vm debian 13 via vmware. une fois l'installation de debian 13 effectuée, je teste les commandes fournies dans le tp afin de vérifier la configuration de mon système.

```console
root@debian:~# lsb_release -a
no lsb modules are available.
distributor id: debian
description:    debian gnu/linux 13 (trixie)
release:        13
codename:       trixie
root@debian:~# df -h
sys. de fichiers taille utilisé dispo uti% monté sur
udev               3,9g       0  3,9g   0% /dev
tmpfs              791m    740k  791m   1% /run
/dev/sda1           19g    1,6g   16g   9% /
tmpfs              3,9g    1,1m  3,9g   1% /dev/shm
tmpfs              5,0m       0  5,0m   0% /run/lock
tmpfs              1,0m       0  1,0m   0% /run/credentials/systemd-journald.service
tmpfs              3,9g       0  3,9g   0% /tmp
tmpfs              1,0m       0  1,0m   0% /run/credentials/getty@tty1.service
tmpfs              791m    8,0k  791m   1% /run/user/1000
root@debian:~# free -h
               total       utilisé      libre     partagé tamp/cache   disponible
mem:           7,7gi       464mi       6,8gi        13mi       707mi       7,3gi
échange:       1,1gi          0b       1,1gi
```

### mise à jour des paquets

je mets ensuite à jour tous les paquets.

```console
root@debian:~# apt update
atteint : 1 http://deb.debian.org/debian trixie inrelease
atteint : 2 http://security.debian.org/debian-security trixie-security inrelease
réception de : 3 http://deb.debian.org/debian trixie-updates inrelease [47,3 kb]
47,3 ko réceptionnés en 0s (181 ko/s)
tous les paquets sont à jour.
root@debian:~# apt upgrade
sommaire :
  mise à niveau de : 0. installation de : 0. supprimé : 0. non mis à jour : 0
root@debian:~# apt dist-upgrade
sommaire :
  mise à niveau de : 0. installation de : 0. supprimé : 0. non mis à jour : 0
```

### installation de postgresql

ensuite, j'installe postgresql sur la machine.

```console
root@debian:~# apt install -y postgresql postgresql-contrib
note : sélection de « postgresql » au lieu de « postgresql-contrib »
installation de :
  postgresql

paquets suggérés :
  postgresql-doc

sommaire :
  mise à niveau de : 0. installation de : 1. supprimé : 0. non mis à jour : 0
taille du téléchargement : 16,7 kb
  espace nécessaire : 30,7 kb / 17,2 gb disponible

réception de : 1 http://deb.debian.org/debian trixie/main amd64 postgresql all 17+278 [16,7 kb]
16,7 ko réceptionnés en 1s (13,0 ko/s)
préconfiguration des paquets...
sélection du paquet postgresql précédemment désélectionné.
(lecture de la base de données... 38737 fichiers et répertoires déjà installés.)
préparation du dépaquetage de .../postgresql_17+278_all.deb ...
dépaquetage de postgresql (17+278) ...
paramétrage de postgresql (17+278) ...
```

je vérifie la version avec :

```console
root@debian:~# psql --version
psql (postgresql) 17.11 (debian 17.11-0+deb13u1)
```

### vérification et activation du service

je vérifie le statut du service postgresql, puis je le lance et je l'active afin qu'il se lance au démarrage.

```console
root@debian:~# systemctl status postgresql
● postgresql.service - postgresql rdbms
     loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: enabled)
     active: active (exited) since mon 2026-08-24 10:09:56 cest; 1h 1min ago
 invocation: cdfc5c00198349d393e75867ac8790bd
   main pid: 2196 (code=exited, status=0/success)
   mem peak: 1.8m
        cpu: 16ms

août 24 10:09:56 debian systemd[1]: starting postgresql.service - postgresql rdbms...
août 24 10:09:56 debian systemd[1]: finished postgresql.service - postgresql rdbms.
root@debian:~# systemctl start postgresql
root@debian:~# systemctl enable postgresql
synchronizing state of postgresql.service with sysv service script with /usr/lib/systemd/systemd-sysv-install.
executing: /usr/lib/systemd/systemd-sysv-install enable postgresql
```

### première connexion avec psql

je bascule ensuite vers l'user 'postgres' et je lance le cli.

```console
root@debian:~# sudo -u postgres psql
psql (17.11 (debian 17.11-0+deb13u1))
saisissez « help » pour l'aide.

postgres=#
```

### installation et configuration de pgadmin

j'installe ensuite pgadmin sur mon hôte windows. pour cela, je dois d'abord modifier la configuration de postgresql sur mon debian afin d'accepter les connexions à la base depuis l'extérieur.

je dois modifier le fichier 'postgresql.conf' situé dans '/etc/postgresql/17/main' afin de changer la ligne :

```ini
listen_addresses = '*'
```

je fais ensuite un restart de postgresql via :

```bash
systemctl restart postgresql
```

j'ajoute ensuite la ligne suivante à la toute fin du fichier '/etc/postgresql/17/main/pg_hba.conf' afin d'autoriser ma machine hôte sur le service postgresql :

```conf
host    postgres        postgres        192.168.183.1/32        scram-sha-256
```

je restart ensuite à nouveau le service postgresql.

je switch ensuite d'user vers 'postgres' afin de set un password :

```console
root@debian:/etc/postgresql/17/main# sudo -i -u postgres
postgres@debian:~$ psql
psql (17.11 (debian 17.11-0+deb13u1))
saisissez « help » pour l'aide.

postgres=# \password postgres
saisir le nouveau mot de passe de l'utilisateur « postgres » :
saisir le mot de passe à nouveau :
```

je fais ensuite un 'ip a' afin de récupérer l'ip de l'interface réseau de la vm, puis je configure pgadmin pour me connecter sur le serveur hébergé sur la vm depuis l'hôte.

je suis bien connecté à mon serveur postgresql depuis mon hôte. tout fonctionne.

## exercice 1.1 : créer une base de données et un utilisateur

je profite d'être toujours dans la console postgresql afin de créer de nouvelles bases de données.

je commence d'abord par créer un utilisateur.

```sql
postgres=# create user labuser with password 'motdepassesecurise';
create role
```

je crée une base de données attribuée à cet user.

```sql
postgres=# create database blogapp_lab owner labuser;
create database
```

je donne tous les droits à l'user dans la base.

```sql
postgres=# grant all privileges on database blogapp_lab to labuser;
grant
```

je liste les bases avec la commande \l :

```text
                                                           liste des bases de données
     nom     | propriétaire | encodage | fournisseur de locale | collationnement | type caract. | locale | règles icu : |    droits d'accès
-------------+--------------+----------+-----------------------+-----------------+--------------+--------+--------------+-----------------------
 blogapp_lab | labuser      | utf8     | libc                  | fr_fr.utf-8     | fr_fr.utf-8  |        |              | =tc/labuser          +
             |              |          |                       |                 |              |        |              | labuser=ctc/labuser
 postgres    | postgres     | utf8     | libc                  | fr_fr.utf-8     | fr_fr.utf-8  |        |              |
 template0   | postgres     | utf8     | libc                  | fr_fr.utf-8     | fr_fr.utf-8  |        |              | =c/postgres          +
             |              |          |                       |                 |              |        |              | postgres=ctc/postgres
 template1   | postgres     | utf8     | libc                  | fr_fr.utf-8     | fr_fr.utf-8  |        |              | =c/postgres          +
             |              |          |                       |                 |              |        |              | postgres=ctc/postgres
```

je liste ensuite les rôles :

```text
postgres=# \du
                                        liste des rôles
 nom du rôle |                                    attributs
-------------+---------------------------------------------------------------------------------
 labuser     |
 postgres    | superutilisateur, créer un rôle, créer une base, réplication, contournement rls
```

### activation de la journalisation des requêtes

je décide ensuite d'activer la journalisation des requêtes.

pour cela, je dois d'abord quitter le mode console postgresql via '\q', puis j'édite le fichier '/etc/postgresql/17/main/postgresql.conf'.

j'ajoute ensuite deux nouvelles lignes 'log_statement = 'all'' et 'log_duration = on'. puis je restart le service.

### création de la première table

je retourne ensuite en mode console postgresql puis je crée ma table de données de test via la commande. lors de la reprise du tp, je constate que cette première exécution a été faite par erreur dans la base postgres. je corrige ce point plus bas en me connectant avec labuser à blogapp_lab avant de rejouer les exercices.

```sql
create table test_users (
	user_id serial primary key,
	username varchar(50) not null unique,
	email varchar(255) not null,
	created_at timestamptz default current_timestamp
);
```

```text
create table
```

j'insère ensuite des données dans les tables.

```sql
insert into test_users (username, email)
values ('alice', 'alice@example.com');
```

```text
insert 0 1
```

```sql
insert into test_users (username, email)
values
	('bob', 'bob@example.com'),
	('charlie', 'charlie@example.com');
```

```text
insert 0 2
```

```sql
select * from test_users;
```

```text
 user_id | username |        email        |          created_at
---------+----------+---------------------+-------------------------------
       1 | alice    | alice@example.com   | 2026-08-24 14:28:38.980743+02
       2 | bob      | bob@example.com     | 2026-08-24 14:30:27.033511+02
       3 | charlie  | charlie@example.com | 2026-08-24 14:30:27.033511+02
(3 lignes)
```

## reprise du tp dans la bonne base

je vérifie d'abord que je peux me connecter en tcp à la base attendue avec le rôle créé dans l'exercice 1.1.

```console
$ psql -h 127.0.0.1 -U labuser -d blogapp_lab
    base     | utilisateur
-------------+-------------
 blogapp_lab | labuser
(1 ligne)
```

je recrée ensuite 'test_users' dans 'blogapp_lab'. la table appartient bien à 'labuser'.

```text
            liste des relations
 schéma |    nom     | type  | propriétaire
--------+------------+-------+--------------
 public | test_users | table | labuser
(1 ligne)
```

j'insère à nouveau les trois lignes de départ afin de poursuivre les exercices dans la bonne base.

```text
 user_id | username |        email        |          created_at
---------+----------+---------------------+-------------------------------
       1 | alice    | alice@example.com   | 2026-08-25 11:26:00.884596+02
       2 | bob      | bob@example.com     | 2026-08-25 11:26:00.886521+02
       3 | charlie  | charlie@example.com | 2026-08-25 11:26:00.886521+02
(3 lignes)
```

## exercice 1.6 : mettre à jour et supprimer

je modifie l'adresse e-mail d'alice.

```sql
update test_users
set email = 'alice.new@example.com'
where username = 'alice';
```

```text
UPDATE 1
 user_id | username |         email
---------+----------+-----------------------
       1 | alice    | alice.new@example.com
(1 ligne)
```

je supprime ensuite 'charlie', puis je contrôle le contenu restant.

```sql
delete from test_users where username = 'charlie';
select user_id, username, email from test_users order by user_id;
```

```text
DELETE 1
 user_id | username |         email
---------+----------+-----------------------
       1 | alice    | alice.new@example.com
       2 | bob      | bob@example.com
(2 lignes)
```

## exercice 1.7 : pratiquer les transactions

je commence par tester une transaction annulée.

```sql
begin;
insert into test_users (username, email)
values ('dave', 'dave@example.com');
rollback;

select count(*) as dave_apres_rollback
from test_users
where username = 'dave';
```

le 'rollback' annule correctement l'insertion.

```text
 dave_apres_rollback
---------------------
                   0
(1 ligne)
```

je rejoue ensuite l'insertion et je la valide avec 'commit'.

```sql
begin;
insert into test_users (username, email)
values ('dave', 'dave@example.com');
commit;

select user_id, username, email from test_users order by user_id;
```

```text
 user_id | username |         email
---------+----------+-----------------------
       1 | alice    | alice.new@example.com
       2 | bob      | bob@example.com
       5 | dave     | dave@example.com
(3 lignes)
```

l'identifiant '5' est normal. la séquence a aussi avancé pendant la transaction annulée.

## exercice 1.8 : nettoyage

je supprime la table de test comme demandé dans le sujet.

```sql
drop table test_users;
```

```text
DROP TABLE
Aucune relation n'a été trouvée.
```

je supprime également la première table de test créée par erreur dans la base 'postgres'. une vérification finale confirme qu'il ne reste aucune table utilisateur dans 'postgres' ni dans 'blogapp_lab'.

## défi optionnel : table notes

je réalise aussi le défi optionnel proposé à la fin du laboratoire. je crée une table 'notes' contenant les colonnes 'note_id', 'title', 'content' et 'created_at', puis j'insère cinq notes.

```sql
create table notes (
    note_id serial primary key,
    title varchar(200) not null,
    content text not null,
    created_at timestamptz default current_timestamp
);

insert into notes (title, content) values
    ('installation', 'postgresql 17 est installe'),
    ('connexion', 'la connexion psql fonctionne'),
    ('pgadmin', 'le serveur est visible depuis windows'),
    ('transactions', 'rollback et commit sont valides'),
    ('nettoyage', 'les tables de test seront supprimees');
```

la requête sur la date du jour retourne bien les cinq notes.

```sql
select note_id, title
from notes
where created_at::date = current_date
order by note_id;
```

```text
 note_id |    title
---------+--------------
       1 | installation
       2 | connexion
       3 | pgadmin
       4 | transactions
       5 | nettoyage
(5 lignes)
```

je mets ensuite à jour le titre de la première note.

```sql
update notes
set title = 'installation terminee'
where note_id = 1;
```

```text
UPDATE 1
 note_id |         title
---------+-----------------------
       1 | installation terminee
(1 ligne)
```

je termine le défi en supprimant toutes les notes puis la table.

```sql
delete from notes;
drop table notes;
```

```text
DELETE 5
DROP TABLE
Aucune relation n'a été trouvée.
```

## vérifications finales

la configuration finale de la vm est la suivante :

```text
os                  : debian 13.6 (trixie)
postgresql          : 17.11
cluster             : 17/main, port 5432, online
service             : active et enabled
listen_addresses    : *
max_connections     : 100
shared_buffers      : 128MB
log_statement       : all
log_duration        : on
base du laboratoire : blogapp_lab, propriétaire labuser
outil graphique     : pgAdmin sur l'hôte Windows
```
