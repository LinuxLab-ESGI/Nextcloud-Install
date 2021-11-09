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

`$ sudo apt update`

`$ sudo apt upgrade`

![Warning](https://api.iconify.design/ion/warning-outline.svg) Ici l'installation se fera avec un serveur apache2, php7.3 et postgresql

### 1. Installation du serveur apache2

`$ sudo apt install apache2`

### 2. Installer PHP et ses dépendances

`$ sudo apt install php php-common php-gd php-igbinary php-imagick php7.3 php7.3-bcmath php7.3-cli php7.3-common php7.3-curl php7.3-gd php7.3-gmp php7.3-igbinary php7.3-intl php7.3-json php7.3-mbstring php7.3-pgsql php7.3-readline php7.3-xml php7.3-zip`

### 3. Installer le SGBD (postgresql)

`$ sudo apt install postgresql`

## Création du dossier d'installation

Nextcloud peut être installer n'importe où sur votre machine, dans notre cas nous allons l'installer au sein de dossier `/var/www/html` mais il pourrait très bien être localisé dans `/home/user`

`$ sudo mkdir -p /var/www/html/nextcloud`

Maintenant il est nécessaire de donner les droits à l'utilisateur `www-data`:

`$ sudo chown -R /var/www/html/nextcloud`

## Configurer sa base de données

## Configurer son serveur web

## Configurer un cache
