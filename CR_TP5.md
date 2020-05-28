# Compte Rendu  - TP 5

180/03/2020
Thibault GILG

## Exercice 1

Pour cet exercice, on doit découper le réseau interne 172.16.0.0 /23 en 7 sous-réseaux pour un total de 254 machines. Ainsi, on ajuste la taille et l'adresse des réseaux en fonction du découpage imposé. Les sous-réseaux comportent tous (sauf SR7) entre 30 et 62 machines (2^5 - 2 < taille < 2^6 -2), donc nous pouvions commencer par quelconque sous-réseau. Voici donc le plan VLSM proposé :

-   _Sous-réseau 1 : 38 machines_

    Adresse de sous réseau : 172.16.0.0 /26
    Adresse de broadcast : 172.16.0.63 /26
    Première machine configuréé : 172.16.0.1 /26  
    Dernière machine configurée : 172.16.0.62 /26
    
-   _Sous-réseau 2 : 33 machines_  
    Adresse de sous réseau : 172.16.0.64 /26
    Adresse de broadcast : 172.16.0.127 /26
    Première machine configuréé : 172.16.0.65 /26
    Dernière machine configurée : 172.16.0.126 /26
    
-   _Sous-réseau 3 : 52 machines_  
    Adresse de sous réseau : 172.16.0.128 /26
    Adresse de broadcast : 172.16.0.191 /26
    Première machine configuréé : 172.16.0.129 /26 
    Dernière machine configurée : 172.16.0.190 /26
    
-   _Sous-réseau 4 : 35 machines_  
    Adresse de sous réseau : 172.16.0.192 /26
    Adresse de broadcast : 172.16.0.255 /26
    Première machine configuréé : 172.16.0.193 /26 
    Dernière machine configurée : 172.16.0.254 /26
    
-   _Sous-réseau 5 : 34 machines_  
    Adresse de sous réseau : 172.16.1.0 /26
    Adresse de broadcast : 172.16.1.63 /26
    Première machine configuréé : 172.16.1.1 /26 
    Dernière machine configurée : 172.16.1.62 /26
    
-   _Sous-réseau 6 : 37 machines_  
    Adresse de sous réseau : 172.16.1.64 /26
    Adresse de broadcast : 172.16.1.127 /26
    Première machine configuréé : 172.16.1.65 /26  
    Dernière machine configurée :172.16.1.126 /26

Pour le dernier sous-réseau, j'ai considéré un masque /27 comme on nécessitait de moins de 30 machines.
  
-   _Sous-réseau 7 : 25 machines_  
    Adresse de sous réseau : 172.16.1.128 /27 
    Adresse de broadcast : 172.16.1.159 /27
    Première machine configuréé :172.16.1.129 /27 
    Dernière machine configurée :172.16.1.158 /27

## Exercice 2

Pour préparer ainsi l'environnement, j'ai effectué les tâches suivantes :
* pour le serveur, configurer un NAT pour avoir Internet et un réseau intern tpadmin.local pour communiquer avec le client
* pour le client, configurer un réseau internet tpadmin.local pour communiquer avec le serveur

Ensuite, pour vérifier la bonne communication entre le client et le serveur j'attribue une adresse IP à une interface de chaque machine avec les commande suivantes :

* serveur :
	```
	sudo ip link set enp0s8 up //activation de la carte 8
	sudo ip addr add 192.168.100.1/24 dev enp0s8 //attribution d'adresse IP
	```
* client :
	```
	sudo ip link set enp0s3 up //activation de l'interface 8
	sudo ip addr add 192.168.100.1/24 dev enp0s3
	 //attribution d'adresse IP
	```	
Lorsque l'on ping entre serveur et client, la communication fonctionne correctement. Cependant, cette configuration n'est que provisoire donc il faut procéder autrement par la suite.

2. L'interface ```lo``` est l'interface de loopback. Lorsque le serveur contacte l'adresse associée, il "boucle" sur lui-même, c'est-à-dire que le signal lui est renvoyé.

## Exercice 3 : Installation du serveur DHCP

2. Pour vérifier que la configuration est correcte, il faut taper la commande ```ìp a``` qui affiche l'état des ports du serveur et leur adresse IP si elles sont été configurées. La configuration est bonne. 

3. Le "lease time" correspond au temps pendant lequel le serveur alloue une adresse IP à un client, dans le cadre du protocole DHCP. Ici, le temps précisé est en secondes. 

4. On modifie donc le fichier ```/etc/default/isc-dhcp-server``` en précisant que le serveur doit écouter sur l'interface ```enp0s8```.

5. Après avoir validé la configuration du serveur, on vérifie que le serveur est actif par la commande ```systemctl status isc-dhcp-server```.

7. J'exécute la commande ```tail -f /var/log/syslog``` pour afficher les dernières lignes du fichier de log du système. Puis, j'active la carte réseau du client et je le lance. Trois lignes apparaissent en plus, commençant chacune respectivement par ```DHCPDISCOVER```, ```DHCPOFFER``` et ```DHCPREQUEST``` qui correspondent à :
* ```DHCPDISCOVER``` : le client envoie par diffusion un message afin de trouver un serveur DHCP. S'il n'y a aucune réponse d'un serveur DHCP, alors le client s'attribue lui-même une adresse IP.
* ```DHCPOFFER``` : réponse d'un serveur DHCP lorsqu'il reçoit un message ```DHCPDISCOVER``` contenant  une adresse IPV4 libre qui est alloué au client
* ```DHCPREQUEST``` : message de diffusion envoyé par le client pour signaler l'acceptation de la première adresse IPV4 envoyée par un serveur DHCP
* ```DHCPACK``` : message de reconnaissance du serveur DHCP envoyé au client. Ce message n'apparaît pas sur mon serveur pour une raison inconnue.

Avec la commande ```ip a``` dans la console du client, on peut voir l'adresse IP de l'interface réseau ```enp0s3```, qui est : *192.168.100.100*, pendant un certain temps. Elle appartient bien à la plage mentionné dans le fichier de configuration : [192.168.100.100 - 192.168.100.240].

8. Le fichier ```/var/lib/dhcp/dhcpd.leases``` contient la base de données des adresses IP allouées au client, en spécifiant pour chaque adresse IP, l'horaire d'attribution, l'horaire de fin de bail, l'adresse MAC et le nom du client. La commande ```dhcp-lease-list``` affiche le contenu du fichier ```dhcpd.leases``` sous la forme d'un tableau avec un titre pour chaque colonne.  Dans mon cas, l'adresse IP 192.168.100.100 y est. 

9. Lorsque chez le serveur (resp. client), on ping l'adresse IP du client (resp. serveur), les requêtes HTTP reviennent bien.

10.  Lorsque l'on modifie le fichier de configuration du serveur ```etc/dhcp/dhcpd.conf```, il faut toujours valider les modifications par la suite avec la commande ```dhcpd -t```. Cela permet de vérifier s'il y a des erreurs de syntaxe dans le code du fichier. Ensuite, on redémarre le service DHCP avec la commande ```systemctl restart isc-dhcp-service``` côté serveur et on force le renouvellement du bail avec la commande ```dhclient -v``` côté client.

## Exercice 4 : Donner un accès Internet au client

1. Tout d'abord, j'autorise *l'IP forwarding* en décommentant une ligne de code du fichier ```/etc/sysctl.conf``` .

2. Ensuite, il faut autoriser la traduction d’adresse source (masquerading) avec ```sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE```. Dorénavant, le client a accès à Internet. L'échange de paquets avec l'adresse IP 1.1.1.1 se fait correctement.

## Exercice 5 : Installation du serveur DNS

1. Pour installer Bind 9, il suffit d'entrer la commande ```sudo apt install bind9```. On le démarre avec la commande ```sudo service bind9 start``` et on vérifie qu'il est actif avec la commande ```systemctl status bind9.service```.

2. Ainsi, pour configurer le serveur DNS, il faut lui renseigner des adresses privées. Pour cela, nous allons modifier le fichier ```/etc/bind/named.conf.options``` et insérer dans la partie "forwarders" les adresses IP 1.1.1.1 et 8.8.8.8. Ensuite, je redémarre le serveur Bind 9 avec ```sudo service bind9 restart``` et je vérifie qu'il est actif avec la commande ```systemctl status bind9.service```.

## Exercice 6 : Configuration du serveur DNS pour la zone *tpadmin.local*

1. Je modifie le fichier ```/etc/bind/named.conf.local``` en ajoutant le code suivant :
```
zone "tpadmin.local" {
type master; // c'est un serveur maître
file "/etc/bind/db.tpadmin.local"; // lien vers le fichier de définition de zone
};
```

2. Ensuite, il faut créer une copie du fichier ```/etc/bind/db.local``` que l'on nomme ```/etc/bind/db.tpadmin.local``` et on remplace toutes les occurrences de ```localhost``` par ```tpadmin.local``` et on lui renseigne l'adresse IP du serveur : 192.168.100.1. Cette étape est la configuration du fichier de zone.

3. Puis, on configure le fichier de zone inverse. On modifie donc le fichier ```/etc/bind/named.conf.local``` avec le code suivant :
```
zone "100.168.192.in-addr.arpa" {
type master;
file "/etc/bind/db.192.168.100";
};
```
Enfin, on crée le fichier ```/etc/bind/db.192.168.100``` sur le modèle de ```/etc/bind/db.127``` et on le modifie de la même façon que pour le fichier de zone.

4. Je valide les fichiers de configuration avec :
```
named-checkconf named.conf.local
named-checkzone tpadmin.local /etc/bind/db.tpadmin.local
named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100
```
Aucune erreur n'est détectée, la configuration paraît correcte.

5. Je redémarre le serveur Bind 9 avec ```sudo systemctl restart bind9.service```. Les "ping" fonctionnent très bien.
