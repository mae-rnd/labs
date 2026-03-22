# Installer BIND en tant que serveur DNS personnel sur Linux

Dans le contexte d’aujourd’hui, il peut être pertinent de faire fonctionner son propre DNS, et ce pour de multiples raisons. On pourrait citer notamment la nécessité de protéger sa vie privée, de bloquer les publicités et les domaines malveillants en amont, ou bien, à l’inverse, d’éviter le blocage DNS.

Pour cela, nous allons installer et configurer BIND en tant que solveur DNS (serveur DNS récursif) sur une Debian 12. C’est un serveur DNS très complet et une référence (beaucoup de serveurs DNS racine l’utilisent), mais il a la réputation d’être assez abscons dans sa configuration. Nous verrons que pour l’usage que nous prévoyons, il est assez simple à mettre en place.

Tout d’abord, on admet que vous disposez déjà d’un serveur Debian installé et à jour. Pour s’en assurer, la commande suivante est suffisante :

`sudo apt update && sudo apt upgrade`

Ici, tel est bien le cas.

<img width="796" height="594" alt="capture-decran-2024-06-24-161544" src="https://github.com/user-attachments/assets/c32767ef-c853-47e8-9579-62a258257226" />

On commence bien évidemment par installer BIND, dans sa dernière version. Il y a pour cela deux paquets à installer, *bind9* et *bind9-utils*.

`sudo apt install bind9 bind9-utils`

Une fois fait, on vérifie que le daemon est bien en cours d’exécution, ce qui est automatique après l’installation normalement.

`sudo systemctl status bind9`

<img width="794" height="372" alt="capture-decran-2024-06-24-162302" src="https://github.com/user-attachments/assets/b35e2b91-34c2-41d7-a02f-6871e6a2d77f" />

Nous allons maintenant le configurer pour l’usage souhaité, à savoir en tant que résolveur DNS. On va donc éditer le fichier */etc/bind/named.conf.options*.

`sudo nano /etc/bind/named.conf.options`

Nous allons rajouter les lignes suivantes :

```
listen-on port 53 { 127.0.0.1; <adresse IPv4 locale>; };
listen-on-v6 port 53 { ::1; <adresse IPv6 locale>; };

```

Les deux adresses locales peuvent être obtenues grâce à la commande `ip addr`. Ces deux lignes indiquent au serveur sur quelles interfaces et quels ports il doit se placer en écoute. Par défaut, les requêtes DNS se font sur le port 53. Ensuite, on spécifie les adresses IP autorisées à faire des requêtes au serveur :

```
allow-query { localhost; 192.168.1.0/24; };
allow-recursion { localhost; 192.168.1.0/24; };
```

Ici, on autorise donc le serveur Debian lui-même, ainsi que les machines connectées au réseau local. Si vous utilisez le firewall ufw, ce que je vous conseille, il faudra penser à ajouter la règle suivante afin d’autoriser les requêtes DNS : `sudo ufw allow dns`

Pour initialiser les fichiers de configuration modifiés, on relance BIND : `sudo systemctl restart bind9`

Enfin, on vérifie le bon fonctionnement du service en résolvant un domaine grâce à lui, par exemple tout simplement Google : `dig @localhost www.google.fr`. Ce qui donne le résultat suivant :

<img width="793" height="387" alt="capture-decran-2024-06-24-164740-1" src="https://github.com/user-attachments/assets/1638e512-b966-4855-a72d-bd655e95af00" />

Le domaine est bien résolu, et on obtient l’adresse IP associée : *216.58.214.163*


On peut relancer la même commande pour voir que le temps de la requête n’est plus du tout le même. En effet, elle est dorénavant servie depuis le cache DNS.

<img width="596" height="328" alt="capture-decran-2024-06-24-165436" src="https://github.com/user-attachments/assets/0318b8cd-e52c-4f5e-b4aa-66be84feca73" />

Voilà, vous avez votre propre serveur DNS fonctionnel !
