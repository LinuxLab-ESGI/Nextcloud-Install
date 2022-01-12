# Nextcloud-Install

- [Nextcloud-Install](#nextcloud-install)
  - [Prérequis](#prérequis)
  - [Mettre une adresse IP fixe](#mettre-une-adresse-ip-fixe)
    - [Sur Debian/Debian-like](#sur-debiandebian-like)
  - [Installer les paquets nécessaires](#installer-les-paquets-nécessaires)
    - [1. Installation du serveur apache2](#1-installation-du-serveur-apache2)
    - [2. Installer PHP et ses dépendances](#2-installer-php-et-ses-dépendances)
    - [3. Installer le SGBD](#3-installer-le-sgbd)
  - [Création du dossier d'installation](#création-du-dossier-dinstallation)
  - [Configurer sa base de données](#configurer-sa-base-de-données)
    - [PostgreSQL](#postgresql)
    - [MariaDB](#mariadb)
  - [Configurer son serveur web](#configurer-son-serveur-web)
    - [Cas 1](#cas-1)
    - [Cas 2](#cas-2)
  - [Installation en interface Web](#installation-en-interface-web)
  - [Configurer un cache](#configurer-un-cache)
    - [Configuration de APCu](#configuration-de-apcu)
    - [Configuration de Redis](#configuration-de-redis)

## Prérequis

- Une machine Unix:
  - Raspbian
  - Debian
  - Ubuntu
  - ...

- Un accès internet

- Un accès CLI

## Mettre une adresse IP fixe

### Sur Debian/Debian-like

Dans `/etc/network/interfaces` ajouter la configuration réseau en fonction du nom de l'interface réseau :

``` text
auto eth1
iface eth1 inet static
  address 192.168.0.1
  netmask 255.255.255.0
  gateway 192.168.56.1
  dns-nameservers 1.1.1.1
```

> `ip a` pour lister les interfaces réseau de la machine

## Installer les paquets nécessaires

On commence par mettre à jour notre machine.

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

![Warning](https://api.iconify.design/ion/warning-outline.svg) Ici l'installation se fera avec un serveur apache2, dernière version de PHP et un SGBD (PostgreSQL ou Apache2).

### 1. Installation du serveur apache2

```bash
sudo apt install apache2
```

### 2. Installer PHP et ses dépendances

```bash
sudo apt install php php-common php-gd php-igbinary php-imagick php php-bcmath php-cli php-curl php-gd php-gmp php-igbinary php-intl php-json php-mbstring php-readline php-xml php-zip
```

### 3. Installer le SGBD

- PostgreSQL

```bash
sudo apt install postgresql php-pgsql
```

- MariaDB

```bash
sudo apt install mariadb-server libapache2-mod-php php-mysql
```

Nous pouvons verifier la bonne installation du serveur Apache en allant en tapant l'adresse IP de la machine dans un navigateur web :

## Création du dossier d'installation

Nextcloud peut être installé n'importe où sur votre machine, dans notre cas nous allons l'installer au sein du dossier `/var/www/` mais il pourrait très bien être localisé dans `/home/user`

```bash
sudo mkdir -p /var/www/nextcloud
```

On récupère le fichier php qui va nous permettre d'installer Nextcloud:

```bash
wget https://download.nextcloud.com/server/installer/setup-nextcloud.php
```

Maintenant il est nécessaire de donner les droits à l'utilisateur `www-data` :

```bash
sudo chown -R /var/www/nextcloud
```

## Configurer sa base de données

### PostgreSQL

Normalement, arrivé à cette étape, vous avez installé *PostgreSQL*.
Maintenant, il faut configurer votre base de données et votre utilisateur pour qu'ils soient par la suite accessible par le *php*.

Pour cela, on commence par accéder à sa base de données en ligne de commande:

```bash
sudo -u postgres psql
```

Ensuite, vous devriez obtenir un bash avec comme prompt: `postgres=#`.
Cela signifie que vous êtes bien connecté.

On va donc créer notre base de données et notre utilisateur:

```sql
CREATE USER myuser CREATEDB;
CREATE DATABASE mydatabase OWNER myuser;
```

On quitte l'interface:

```sql
\q
```

### MariaDB

Terminer la configuration de mysql :  
`mysql_secure_installation`

``` text
Set root password? [Y/n] : Y
Remove anonymous users? [Y/n] : Y
Disallow root login remotely? [Y/n] : Y
Remove test database and access to it? [Y/n] : Y
Reload privilege tables now? [Y/n] : Y
```

Connexion à mariadb :  
`sudo mysql`

Creation d'un utilisateur pour le serveur Nextcloud :

``` sql
CREATE USER 'nextcloud_user'@'localhost' IDENTIFIED BY 'password';
```

``` SQL
Création de la base de donnée :
CREATE DATABASE IF NOT EXISTS nextcloud_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

Droits à notre utilisateur Nextcloud :

``` SQL
GRANT ALL PRIVILEGES ON nextcloud_db.* TO 'nextcloud_user'@'localhost';
```

Application les changements :

``` SQL
FLUSH PRIVILEGES;
```

## Configurer son serveur web

Maintenant que notre base de données est configurée, il ne nous reste plus qu'à installer le serveur en lui-même.

Pour cela, il faut déjà créer le site virtuel pour apache afin de terminer l'installation en interface web.

On crée donc un site virtuel:

```bash
# on copie le fichier de conf par défaut pour créer le notre
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/nextcloud.conf
# on modifie notre fichier de configuration
sudo nano /etc/apache2/sites-available/nextcloud.conf
```

Une fois le fichier ouvert, on lui ajoute les lignes suivantes:

```bash
ServerAdmin nextcloud@mydomain.tld

DocumentRoot /var/www/html/nextcloud

<Directory /var/www/html/nextcloud>

  Options Indexes FollowSymLinks
  AllowOverride All
  Require all granted

  <IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
  </IfModule>

  <IfModule mod_dav.c>
    Dav off
  </IfModule>

</Directory>

```

Maintenant que notre site est créé, on l'active:

```bash
# on active le site virtuel
sudo a2ensite nextcloud.conf
# on redémarre le serveur web
sudo systemctl restart apache2.service
```

Afin de terminer l'installation de votre serveur, il faut maintenant vous rendre sur l'interface web de celui-ci. On accède donc à `127.0.0.1:80`.

Si vous souhaitez avoir des URL sans extensions de fichiers, il suffit d'activer deux modules et de rajouter de la configuration.

### Cas 1

Si vous hébergez votre nextcloud sur derrière un nom de dossier comme `https://mydomain.tld/nextcloud/`, alors vous devez rajouter les lignes suivantes dans votre fichier `config.php`:

```bash
sudo nano /var/www/html/nextcloud/config/config.php
```

```php
'overwrite.cli.url' => 'https://mydomain.tld/nextcloud',
'htaccess.RewriteBase' => '/nextcloud',
```

### Cas 2

Si vous hébergez votre nextcloud sur directement sur votre domaine comme `https://mydomain.tld`, alors vous devez rajouter les lignes suivantes dans votre fichier `config.php`:

```bash
sudo nano /var/www/html/nextcloud/config/config.php
```

```php
'overwrite.cli.url' => 'https://mydomain.tld/',
'htaccess.RewriteBase' => '/',
```

Dans les 2 cas, vous devez ensuite effectuer la commande suivante pour mettre à jour votre fichier `.htaccess`:

```bash
sudo -u www-data php /var/www/html/nextcloud/occ maintenance:update:htaccess
```

## Installation en interface Web

Maintenant que vous avez fini de configurer votre serveur dans son ensemble, vous devez aller sur son interface web, accessible par `ip_serveur:port`, dans notre cas, `127.0.0.1:80`.

![Nextcloud installation interface](images/install.png)

Il va falloir spécifier la localisation du dossier des données utilisateur, il peut très bien être en dehors du dossier nextcloud mais il faudra bien penser à donner les droits à l'utilisateur `www-data`. Pour nous c'est: `/var/www/html/nextcloud/data`.

Vous aller choisir votre couple *user/password* d'administration, puis sélectionner la base de données que vous allez utiliser. Dans notre cas, nous choisirons *postgresql*.

On va ensuite remplir les champs indiquant les informations sur notre base de données. Pour nous, il s'agit des étapes effectuées lors de l'étape **Configurer sa base de données**, à savoir le nom de la base et le couple *user/password*.

On active également l'option d'installation des applications par défaut.

Si on valide l'installation, on devrait arriver sur l'interface d'accueil du serveur ce qui signifie qu'il est prêt à être utilisé.

## Configurer un cache

Maintenant que notre serveur nextcloud est configuré. On peut observer que des ralentissements peuvent survenir lors du chargement de certaines pages.

Il pourrait donc être intéressant d'installer un système de cache afin de ne pas avoir de problèmes de chargement.

Deux options s'offrent à nous, le serveur de cache directement sur notre serveur nextcloud ou alors sur un autre serveur en version distribuée.

Nous allons aborder l'installation sur le même serveur et utiliser les deux outils suivants:

- APCu: Gère un cache local du système.
- Redis: Gère le cache en mode local ou distribué, prend également en charge les transactions sur les fichiers.


### Configuration de APCu

### Configuration de Redis
