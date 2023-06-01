# projet_A_ITS
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
[Capture VirtualBox](/images/cap3.png "joie et félicité, ça a fini par marcher")
On s'auto-congratule modérément et on commence les recherches sur l'installation de **Mediawiki** pour le lendemain... Ça va être sympa, on va devoir installer PHP, une base de donnée, extraire plein de fichiers, bref il va y avoir de quoi faire !
