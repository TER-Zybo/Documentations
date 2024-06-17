# Comment activer SSH

## Introduction

Le protocole SSH est un protocole de communication sécurisé permettant d'accéder à distance à un système. Ce guide vous explique comment activer et configurer SSH sur une carte embarquée sous Debian.

## Configuration d'une IP statique (optionnel)

Pour faciliter la connexion SSH, il peut être utile de configurer une IP statique. Suivez les étapes ci-dessous pour définir une IP statique sur votre carte.

1. **Éditez le fichier de configuration réseau :**

    ```
    sudo vim /etc/network/interfaces.d/ssh_static.conf
    ```

    Ajoutez les lignes suivantes :

    ```plaintext
    auto enp0
    iface enp0 inet static
        address 192.168.0.50
        netmask 255.255.255.0
        gateway 192.168.0.1
    ```

    Remplacez `enp0` par le nom de votre interface réseau si nécessaire.

2. **Redémarrez le service réseau :**

    ```
    sudo systemctl restart networking
    ```

3. **Vérifiez la configuration réseau :**

    ```
    ifconfig
    ```

    Assurez-vous que l'adresse IP statique est correctement assignée.

4. **Configurez votre PC avec une adresse IP dans le même sous-réseau :**

    Assignez à votre PC une adresse IP comme `192.168.0.x`, où `x` est un nombre différent de 50.

5. **Testez la connexion en pingant l'adresse IP :**

    ```
    ping 192.168.0.50
    ```

    Si vous recevez une réponse, cela signifie que la configuration réseau est correcte.

## Configuration SSH

1. **Éditez le fichier de configuration SSH :**

    ```
    sudo vim /etc/ssh/sshd_config.d/local.conf
    ```

    Ajoutez les lignes suivantes :

    ```plaintext
    Port 22
    AddressFamily inet
    ListenAddress 192.168.0.50
    ```

    Cela configure le service SSH pour écouter sur le port 22 et l'adresse IP 192.168.0.50.

2. **Redémarrez le service SSH :**

    ```
    sudo systemctl restart sshd
    ```

3. **Vérifiez l'état du service SSH :**

    ```
    sudo systemctl status sshd
    ```

    Assurez-vous que le service SSH est actif et en cours d'exécution.

4. **Testez la connexion SSH depuis votre PC :**

    ```
    ssh debian@192.168.0.50
    ```

    Remplacez `debian` par votre nom d'utilisateur sur la carte embarquée. Si la connexion est réussie, vous avez activé SSH avec succès.
