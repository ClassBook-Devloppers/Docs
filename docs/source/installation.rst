=====
Installation
=====

ClassBook à forcément besoin d'un 2ème disque dur pour stocker les fichiers utilisateurs, les datas, ...

.. _installation:

Installation Facile
===================

 Pour installer ClassBook Facilement, La première chose à faire est de télécharger le shell script avec cette commande:

.. code-block:: console

   $ wget https://raw.githubusercontent.com/classbook-devloppers/linux-server/main/preinstall.sh


et ensuite venir faire un
  
.. code-block:: console
  
  $ sudo chmod 313 preinstall.sh
  
et 
  
.. code-block:: console
  
  $ ./preinstall.sh

.. note::

  Pour pouvoir utiliser classbook, il faut que le serveur soit sous Ubuntu Server 22.04, Debian 10 ou WSL

Installer ClassBook en Ligne de Commande
========================================
                                                                     
Installer ClassBook en ligne de commande est un peu plus compliqué car c'est l'utilisateur qui exécute les commandes à la place du script et une mauvaise commande peut être un risque pour l'OS ou la machine

Pour installer Classbook en Ligne de Commande nous aurons besoin de :

- Git
- Nginx
- MariaDB
- SSH
- Samba
- PHP
- Parted

1ère Partie : Installations des Paquets
------------

Effectuer les mises à jour du serveur avec 
                                                                     
.. code-block:: console        

  $ sudo apt-get update -y

.. code-block:: console 
                                                                     
  $ sudo apt update -y

et installer les services Nginx, MariaDB, SSH, Samba, PHP et Git avec cette commande :
                                                                     
.. code-block:: console
                                                                     
 $ sudo apt install nginx mariadb-server openssh-server samba parted git php -y

vous pourrez verifier si voutre serveur web est fonctionnel en entrant son IP ( si vous ne savez pas vous pouvez utiliser la commande 

.. code-block:: console
 
 $ ip address

la carte réseau devrait commençer par : "ens" ou "enp"                                                               
                                                                               
2ème Partie : Configuration des Applications
------------
Une fois l'étape 1 réussi
                                                                     
confgurer mariadb avec cette commande : 

.. code-block:: console

  $ sudo mariadb-secure-installation

                    
et répondre aux questions par 
  
    Le mot de passe de l'utilisateur,
    N,
    N,
    Y,
    N,
    Y,
    Y,

Une fois ça fait, nous allons configurer le disque dur avec parted :

..note:: 

    si vous êtes plus à l'aise avec un autre logiciel de partitionnage que parted vous pouvez l'utiliser en utilisant la même configuration des systemes de fichiers

Nous aurons besoin de :

- Un disque dur avec au moins 100Go d'espace libre
- Parted
- un accès Super-utilisateur
- Nano

> Partie 1 : Configuration des partitions 

Sélectionner un disque (souvent /dev/sdb comme deuxième disque)
.. note::
    Faites attention à bien mettre le nom de votre disque à la place de 'votre_disque'


.. code-block:: console

    $ lsblk -d -o NAME,SIZE 

Une fois le disque choisi, le partitionner avec ces commandes :

.. code-block:: console

   $ sudo parted 'votre_disque' mklabel gpt

.. code-block:: console

   $ sudo parted -a opt 'votre_disque' mkpart primary ext4 10G
   $ sudo parted -a opt 'votre_disque' mkpart primary ext4 30G
   $ sudo parted -a opt 'votre_disque' mkpart primary ext4 40G

.. code-block:: console
     
    $ parted $selected_disk align-check optimal 1

.. code-block:: console

   $ sudo mkfs.ext4 'votre_disque' 1
   $ sudo mkfs.ext4 'votre_disque' 2
   $ sudo mkfs.ext4 'votre_disque' 3

.. code-block:: console

    $ sudo e2label 'votre_disque' 1 /classbook/web
    $ sudo e2label 'votre_disque' 2 /classbook/smb
    $ sudo e2label 'votre_disque' 3 /classbook/datas

Une fois que toutes ces étapes ont été faites, il faut entrer les noms des volumes dans /etc/fstab avec nano :
.. code-block:: console

    $ sudo nano /etc/fstab

.. code-block:: console

    'votre_disque' /classbook/web ext4 defaults 0 0
    'votre_disque' /classbook/smb ext4 defaults 0 0
    'votre_disque' /classbook/datas ext4 defaults 0 0

Configuration de Samba et Nginx :

Grace aux partitions précédentes, nous pouvons faire la configuration de nginx et samba :

Nous aurons besoin de : 

- Nano
- Nginx
- Samba
- Un accès Super-utilisateur

> Partie 1 : Configuration de Nginx

Tout d'abord, taper la commande : 

.. code-block:: console

    $ sudo nano /etc/nginx/sites-availables/classbook 

Et dans nano, mettre ce morceau de code :

.. code-block:: 

    server {
    listen 80;
    server_name classbook;

    root /classbook/web;
    index index.html index.htm;


    location /datas {
        alias /classbook/datas;
    }
}

Et activer le site avec cette commande : 

.. code-block:: console

    $ ln -s /etc/nginx/sites-available/classbook /etc/nginx/sites-enabled/

> Partie 2 : Configuration de Samba

Avant de configurer les partages samba, il faut créer un nouvel utilisateur : 

.. code-block:: console

    $ sudo smbpasswd classbook

et rentrer un mot de passe 

Pour configurer samba ouvrir nano en super-utilisateur avec cette commande : 

.. code-block:: console

    $ sudo nano /etc/samba/smb.conf

Entrer ce code : 

.. code-block::

    [datas]
    path = /classbook/datas
    valid users = @admin, classbook
    writable = yes
    guest ok = no
    create mode = 0770
    directory mode = 0770
    force group = admin

[shared]
    path = /classbook/smb
    valid users = @admin, classbook
    writable = yes
    guest ok = no
    create mode = 0770
    directory mode = 0770
    force group = admin

3ème Partie : Configuration de Classbook
------------

Pour pouvoir utiliser classbook, Il nous faut : 

- Le code source de classbook
- Un accès Super-utilisateur

> Étape 1 :

Pour avoir le code source dans le répertoire /classbook/web, il faut aller dans ce répertoire : 

.. code-block:: console

    $ cd /classbook/web

puis faire : 

.. code-block:: console

    $ git clone https://github.com/classbook-devloppers/source-code.git

3ème Partie : Post-Installation
------------

pour la Post-Installation, redémarrer tout les services, enlever les fichier inutiles et effacer le cache :

.. code-block:: console 

    $ sudo apt autoremove -y

.. code-block:: console 

    sudo nginx -s reload && sudo systemctl restart mariadb &&  sudo systemctl restart smbd && sudo systemctl restart nginx

Une fois ces commandes éxécutés, redémarrer le serveur ( de préference ) avec la commande :

.. code--block:: console

    $ sudo reboot 

FIN : 
------------

Voilà, vous avez réussi à installer classbook sur votre serveur