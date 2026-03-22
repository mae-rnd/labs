# Mettre en place un reverse proxy avec des sous domaines

Depuis qu’on m’a offert un Raspberry Pi 5 pour mon anniversaire, j’ai monté un petit serveur domestique avec plusieurs services dessus, en bare metal et avec Docker. Il peut être intéressant et « esthétique » de monter un reverse proxy avec plusieurs sous domaines pour y accéder, ce qui est toujours plus pratique que d’y accéder à travers une adresse IP locale et un port, du type  `192.168.1.10:8090`. Pour cela, nous allons avoir besoin de deux logiciels : nginx (pour le reverse proxy) et BIND (pour gérer les sous domaines).

Tout d’abord, nous installons ce duo grâce à la commande  `sudo apt install nginx bind9 bind9-utils`. Ensuite, on vérifie qu’ils fonctionnent tous les deux correctement :

![capture-decran-2024-11-02-173554](https://github.com/user-attachments/assets/6df3f5cb-e630-47bd-be22-da6adc4dd7e8)


Nous allons commencer la configuration de tout ça. Pour l’exemple, on imagine deux services qui tournent sur notre serveur : Plex, et Portainer. Afin d’utiliser nginx en tant que reverse proxy, on édite le fichier `/etc/nginx/sites-available/default` avec nano (pensez à le faire avec les droits root). A la fin du fichier, ajoutez les lignes suivantes :

```
server {
    listen 80;
    server_name portainer.<votre domaine>;

    location / {
        proxy_pass http://<adresse:port de Portainer>;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/websocket/ {
        proxy_pass http://<adresse:port de l'API, souvent 8000>;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

Pour Portainer, nous avons besoin de définir deux  `location`, car il utilise à la fois une UI et une API, et cela ne fonctionnera pas si l’une des deux seulement est spécifiée. Ce cas quelque peu particulier est la raison pour laquelle j’ai choisi ce logiciel.

En ce qui concerne Plex, les choses sont plus simples :

```
server {
    listen 80;
    server_name plex.<votre domaine>;

    location / {
        proxy_pass http://<IP de Plex:32400>;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

L’IP en question peut être `127.0.0.1`, ou bien alors l’adresse du conteneur Docker, le cas échéant. On peut l’obtenir grâce à `docker inspect <nom ou ID du conteneur>`.

<img width="854" height="578" alt="capture-decran-2024-11-03-153724" src="https://github.com/user-attachments/assets/3e61bd9d-d12a-4b23-a1f4-b8f528cc8ff4" />

Maintenant que nous avons paramétré nginx comme il convient, nous devons nous occuper de BIND. Il y aura plusieurs fichiers de configuration à modifier.

Premièrement, on exécute la commande  `sudo nano /etc/bind/named.conf.local`. À la fin du fichier, on ajoute les lignes suivantes :

```
zone "exemple.com" {
    type master;
    file "/etc/bind/exemple.com";
};
```

Bien évidemment, il convient de modifier  `exemple.com`  avec votre domaine.

Nous venons de définir la zone DNS dans BIND. Dans un second temps, nous devons la configurer à travers un autre fichier de configuration, qu’il nous faut créer. Celui-ci serait dans notre cas  `/etc/bind/exemple.com`.

```
$TTL    604800         ; Durée de vie par défaut
@       IN      SOA     ns1.exemple.com. admin.exemple.com. (
                              2024101901         ; 
                              604800         ; Mise à jour toutes les semaines
                              86400          ; Tentatives de rafraîchissement
                              2419200        ; Expiration après un mois
                              604800 )       ; Cache négatif TTL d'une semaine

; Serveurs de noms
@       IN      NS      ns1.exemple.com.

; Enregistrement A pour les serveurs de noms
ns1     IN      A       127.0.0.1    ; Adresse IP de ns1

; Enregistrement A pour le domaine principal
@       IN      A       <IP locale>        ; IP pour example.com
www     IN      A       <IP locale>        ; IP pour www.example.com

; Sous-domaines pointant vers la même IP
portainer    IN      A       <IP locale>        ; IP pour sub1.example.com
plex    IN      A       <IP locale>
```

Ceci nous permettra d’accéder à Plex à travers `plex.exemple.com`. Enfin, maintenant que nous avons configuré à la fois notre serveur DNS et notre reverse proxy, nous devons redémarrer les deux services afin que la nouvelle configuration soit initialisée :
```
sudo systemctl restart bind9
sudo systemctl restart nginx
```
