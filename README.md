# Installation-de-GlusterFS
# Dans le cadre de la mise en place d'un environnement de stockage distribué: GlusterFS

# Installation de GlusterFS sur les deux serveurs (Srv1-D12 et Srv2-D12):

    apt-get update
    apt-get install glusterfs-server -y

# Configuration de la partition:

# Identification des partitions disponibles sur chaque serveur avec la commande :

    lsblk

# Supposons que nous j'ai décidé d’utiliser la partition /dev/nvme1n1 comme volume GlusterFS. il est necessaire de Formater cette partition en ext4 et la monter:

    mkfs.ext4 /dev/nvme1n1
    mkdir /mnt/brick1
    mount /dev/nvme1n1 /mnt/brick1

# Configuration de GlusterFS:
# Sur chaque serveur, j'autorise le trafic TCP entrant dans le port 24007 (GlusterFS):

    iptables -A INPUT -p tcp --dport 24007 -j ACCEPT
    service iptables-persistent save
    service iptables-persistent reload
    
# Initialisation des volumes distants sur chaque serveur:

    gluster peer probe Srv2-D12
    gluster peer probe Srv1-D12

# Cela ajoute l’autre serveur en tant que pair. Vérification des pairs existants:

    gluster peer status

# Installation du «point de réplication»:

# Création d'un volume répliqué appelé gvol1 composé de deux bricks, un sur chaque serveur:

    gluster volume create gvol1 transport tcp Srv1-D12:/mnt/brick1 Srv2-D12:/mnt/brick1

# Activez le volume:

    gluster volume start gvol1

# Afficher les informations sur le volume:

    gluster volume info

# Test de la réplication inter-serveurs:

# Créez un répertoire sur le premier serveur et affichez sa structure sous forme arborescente:

    ssh Srv1-D12 mkdir /mnt/brick1/rep-test
    ssh Srv1-D12 tree /mnt/brick1

# Création d'un fichier sur le deuxième serveur et l'observation de la synchronisation avec le premier:

    touch /mnt/brick1/rep-test/file1
    sleep 5
    ssh Srv1-D12 tree /mnt/brick1

# Tests complémentaires:

# j'ai Écrit des données sur un brick et check depuis l'autre brick. Par exemple, sur Srv1-D12:

    echo 'Hello World!' > /mnt/brick1/rep-test/hello.txt
    cat /mnt/brick1/rep-test/hello.txt
    
# Sur Srv2-D12, on doit obtenir le même résultat:

    cat /mnt/brick1/rep-test/hello.txt

# Création d'un répertoire sur l'un des bricks, et vérification qu'il est répliqué sur l'autre brick:

    ssh Srv1-D12 mkdir /mnt/brick1/rep-test/newdir
    ssh Srv2-D12 ls -l /mnt/brick1/rep-test/ | grep newdir

# Le nouveau répertoire devrait apparaitre sur le deuxième serveur.
# Déconnectez un des serveurs de la paire GlusterFS:

    gluster peer detach Srv2-D12



