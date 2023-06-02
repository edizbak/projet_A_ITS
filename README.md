# Projet_A_ITS
Ce projet vise à la mise en place d'un environnement dynamique automatisé pour **Mediawiki**

## Environnement cible
Le but est de mettre en place 3 machines qui seront configurées via un script **Vagrant** :
- un load balancer **nginx**
- deux serveurs applicatifs **mediawiki**

***

# Jour 1
## Configuration initiale des VM
En premier lieu, on a récupéré les fichiers *Vagrantfile* et *install_ansible.sh* à partir du repo https://github.com/diranetafen/cursus-devops/tree/master/vagrant/ansible
Ces fichiers vont nous servir de base et on va les éditer afin qu'ils correspondent à nos besoins (puisque nous ne souhaitons pas particulièrement installer **Ansible** sur nos VMs).
- la première action est donc de remplacer *ansible* par *mediawiki* dans le fichier *Vagrantfile*
- on met ensuite en commentaire la plupart des actions du script *install_ansible.sh*, l'installation de **Mediawiki** à proprement parler sera effectuée ultérieurement

On teste l'exécution du *Vagrantfile* (ne pas oublier de se placer dans le répertoire où sont nos fichiers !) :
```
vagrant up
```
Une première erreur apparaît :
![Erreur Vagrant](/images/cap1.png "Première erreur Vagrant")

Une rapide recherche google nous apprend que le problème vient d'un certificat self-signed comme il en existe tant... [Des détails ici](/stories/certif_kaspersky.md)

La solution retenue est d'ajouter une ligne à la config dans *Vagrantfile* :
```
mediawiki.vm.box_download_insecure=true
```
*Probablement à proscrire en contexte réel si on n'est pas certain de l'identité du serveur contacté par vagrant pour télécharger l'image désirée*

On relance l'exécution du *Vagrantfile* pour tomber sur une nouvelle erreur :
![2nde Erreur Vagrant](/images/cap2.png "Invalid state: unknown")

Celle-ci semble provoquée par le délai induit par l'UAC Windows, erreur bénigne donc.
On détruit l'environnement, on relance, et la troisième tentative aboutit enfin :

![Capture VirtualBox](/images/cap3.png "joie et félicité, ça a fini par marcher")

On s'auto-congratule modérément et on commence les recherches sur l'installation de **Mediawiki** pour le lendemain... Ça va être sympa, on va devoir installer PHP, une base de donnée, extraire plein de fichiers, bref il va y avoir de quoi faire !

***

# Jour 2
## Pré-requis d'installation de Mediawiki

Avant d'installer **Mediawiki**, il nous faut installer quelques pré-requis :
- nous avons besoin de **PHP**
- il nous faut également un serveur http, on a choisi **Nginx**
- enfin il faut une base de données, on prendra <s>**PostgreSQL**</s> **MariaDB**

Toutes ces étapes vont être effectuées automatiquement par le script d'installation *install_mediawiki.sh*.
En attendant, on va déjà s'occuper d'installer ces paquets manuellement pour ensuite être en mesure d'automatiser ces étapes.

## Installation PHP  
**Mediawiki** a besoin d'une version récente de **PHP** pour fonctionner, on va donc récupérer une version hébergée dans le repo [remirepo](http://rpms.remirepo.net/enterprise/remi-release-7.rpm) :  
```
yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum-config-manager --enable remi-php81
yum update
yum install -y php
```


  ## Installation PostgreSQL
<s>Il nous faut maintenant une version récente de **PostgreSQL**, et encore une fois celle qui est disponible sur les repos officiels de **CentOS7** est trop ancienne. On va donc installer les repos officiels de **PostrgreSQL**, puis installer leur version :  
```
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y postgresql15-server
postgresql-15-setup initdb
systemctl enable postgresql-15
```
  </s>  
  
 On change d'avis parce que MariaDB est le SGBD recommandé par la documentation de **Mediawiki**, et donc :  
 
 ## Installation MariaDB
 Il nous faut une version de **MariaDB** supérieure à la 10.3, qui n'est pas dans les repos officiels de **CentOS 7**, on va donc la chercher sur [MariaDB.com](https://mariadb.com/resources/blog/installing-mariadb-10-on-centos-7-rhel-7/) :  
 ```
 wget https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
 chmod 744 mariadb_repo_setup
 ./mariadb_repo_setup
 yum install MariaDB-server
 systemctl enable mariadb
 systemctl start mariadb
 ```

 Cette étape effectuée, nous devons désormais mettre en place les tables et l'utilisateur qui seront utilisées par **Mediawiki** :  
 ```
 mysql
 CREATE DATABASE my_wiki;
 CREATE USER 'wikiuser'@'localhost' IDENTIFIED BY 'database_password';
 GRANT ALL PRIVILEGES ON my_wiki.* TO 'wikiuser'@'localhost' WITH GRANT OPTION;
 ```
 ![Résultat création MariaDB](/images/maria1.png)  
 
 Tout semble ok, nous pouvons passer à l'installation de **Mediawiki** à proprement parler.
 
## Installation Mediawiki
