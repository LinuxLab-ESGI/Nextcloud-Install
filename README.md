# Nextcloud-Install

## Prérequis

- Une machine Unix:
  
  - Raspbian
  - Debian
  - Ubuntu
  - ...

- Un accès internet

- Un accès CLI

## Installer les paquets nécessaires

On commence par mettre à jour notre machine

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

![Warning](https://api.iconify.design/ion/warning-outline.svg) Ici l'installation se fera avec un serveur apache2, php7.3 et postgresql

### 1. Installation du serveur apache2

```bash
sudo apt install apache2
```

### 2. Installer PHP et ses dépendances

```bash
sudo apt install php php-common php-gd php-igbinary php-imagick php7.3 php7.3-bcmath php7.3-cli php7.3-common php7.3-curl php7.3-gd php7.3-gmp php7.3-igbinary php7.3-intl php7.3-json php7.3-mbstring php7.3-pgsql php7.3-readline php7.3-xml php7.3-zip
```

### 3. Installer le SGBD (postgresql)

```bash
sudo apt install postgresql
```

## Création du dossier d'installation

Nextcloud peut être installer n'importe où sur votre machine, dans notre cas nous allons l'installer au sein de dossier `/var/www/html` mais il pourrait très bien être localisé dans `/home/user`

```bash
sudo mkdir -p /var/www/html/nextcloud
```

On récupère le fichier php qui va nous permettre d'installer Nextcloud:

```bash
wget https://download.nextcloud.com/server/installer/setup-nextcloud.php
```

Maintenant il est nécessaire de donner les droits à l'utilisateur `www-data`:

```bash
sudo chown -R /var/www/html/nextcloud
```

## Configurer sa base de données

Normalement, arrivé à cette étape, vous avez installé *postgresl*.
Maintenant, il faut configurer votre base de données et votre utilisateur pour qu'ils soient pas la suite accessible par le *php*.

Pour cela, on commence par accéder à sa base de données en ligne de commande:

```bash
sudo -u postgres psql
```

Esuite, vous devriez obtenir un bash avec comme prompt: `postgres=#`.
Cela signifie que vous êtes bien connectés.

On va donc créer notre base de données et notre utilisateur:

```sql
CREATE USER myuser CREATEDB;
CREATE DATABASE mydatabase OWNER myuser;
```

On quitte l'interface:

```sql
\q
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

## Configurer un cache
