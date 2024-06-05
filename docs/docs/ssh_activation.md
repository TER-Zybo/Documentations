# Comment activer SSH

## Introduction

Le protocole SSH (Secure Shell) est un protocole de communication sécurisé qui permet d'accéder à distance à un système. Ce guide explique comment activer et configurer SSH sur une carte embarquée sous Debian.

## Configuration d'une IP statique (optionnel)

1. Éditez le fichier de configuration réseau pour définir une IP statique :
   
    ```bash
    sudo vim /etc/network/interfaces.d/ssh_static.conf
    ```

    Ajoutez les lignes suivantes :

    ```plaintext
    auto enp0
    iface enp0 inet static
    	address 192.168.0.50
    ```

2. Redémarrez le service réseau :

    ```bash
    sudo systemctl restart networking
    ```

3. Vérifiez la configuration réseau avec la commande :

    ```bash
    ifconfig
    ```

4. Configurez votre PC avec une adresse IP dans le même sous-réseau, par exemple 192.168.0.x, où x est un nombre différent de 50.

5. Testez la connexion en pingant l'adresse IP :

    ```bash
    ping 192.168.0.50
    ```

## Configuration SSH

1. Éditez le fichier de configuration SSH :

    ```bash
    sudo vim /etc/ssh/sshd_config.d/local.conf
    ```

    Ajoutez les lignes suivantes :

    ```plaintext
    Port 22
    AddressFamily inet
    ListenAddress 192.168.0.50
    ```

2. Redémarrez le service SSH :

    ```bash
    sudo systemctl restart sshd
    ```

3. Vérifiez l'état du service SSH :

    ```bash
    sudo systemctl status sshd
    ```

4. Testez la connexion SSH depuis votre PC :

    ```bash
    ssh debian@192.168.0.50
    ```