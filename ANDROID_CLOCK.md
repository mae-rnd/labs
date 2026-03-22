# Transformer un vieux téléphone Android en horloge de bureau

Je pense que tout le monde possède un vieux smartphone Android qui traine au fond d’un tiroir et qui prend la poussière. Pourquoi ne pas en faire quelque d’utile ? Je vous propose un petit projet sympa qui va vous permettre d’en faire une horloge de bureau avec un design assez esthétique.

<img width="1020" height="655" alt="Le résultat final (ici sur une VM Android x86)" src="https://github.com/user-attachments/assets/e43fe697-d4cd-4078-8e55-3c767c4f432e" />

La première chose à faire est de faire une réinitialisation usine de votre téléphone. Vous pouvez également en profiter pour installer une ROM custom, si vous le souhaitez. Vous trouverez toutes les ressources nécessaires pour ce faire sur XDA au besoin. Personnellement, j’ai utilisé un vieux smartphone d’entrée de gamme de 2016 qui fonctionnait de base avec Android 7 Nougat, et que j’ai upgradé vers Android 10. Pour ce tuto, j’ai utilisé une VM Android x86.

<img width="1022" height="721" alt="Android x86" src="https://github.com/user-attachments/assets/c557cf1d-bfac-48ca-affe-7e157e037c78" />

La première chose à faire, c’est de télécharger Termux, un émulateur de terminal pour Android. Il est conseillé de télécharger l’APK depuis le [dépôt Github](https://github.com/termux/termux-app) officiel. Une fois installée, lancez l’application comme n’importe quelle application Android. Vous devriez voir cet écran :

<img width="1020" height="662" alt="capture-decran-2024-07-16-163056" src="https://github.com/user-attachments/assets/f35e56b7-4039-4595-bc65-54c862711044" />

Nous allons avoir besoin de divers packages pour notre projet. Pour installer tout cela, saisissez : `pkg install tmux peaclock neofetch ruby`.

Ceci installera tout ce dont nous avons besoin :

-   tmux est un multiplexeur qui nous permettra de diviser l’écran en plusieurs parties ;
-   peaclock est l’horloge ;
-   neofetch permet d’afficher des informations sur le système, c’est la partie gauche du terminal ;
-   ruby est nécessaire pour installer lolcat, qui permet d’afficher le texte de neofetch en couleur, ce que nous allons faire maintenant grâce à la commande `gem install lolcat`.

Maintenant que nous avons tout ce dont nous avons besoin sur le système, nous allons pouvoir passer à la configuration.

Les fichiers de configuration de tmux sont  `~/.tmux.conf`  et  `~/.tmux.conf.local`. Si vous souhaitez dupliquer le style du screenshot, utilisez simplement les fichiers que  [je vous fournis](https://gist.github.com/mae-rnd/ad288cd35cfddd8c887e6075ba4e5e47). Sinon, à titre personnel, je peux vous recommander  [celui-ci.](https://github.com/gpakosz/.tmux)  Mais une petite recherche Google pourra vous donner d’autres idées. N’oubliez pas d’appliquer la configuration grâce à la commande  `tmux source-file ~/.tmux.conf`.

Il en va de même pour peaclock. Le fichier de configuration est localisé dans  `~/.peaclock/`config. Là encore vous trouverez assez facilement des bonnes idées esthétiques sur Internet, mais en suivant le lien ci-dessus vous aurez le fichier de configuration dont je me sers.

Tout étant configuré, nous allons pouvoir démarrer tout cela. Tout d’abord, sur Termux, tapez la commande  `tmux splitw -h -l 80`. Ceci divisera votre écran en deux parties, celle de droite étant plus large que celle de gauche. Notez qu’en fonction de la taille de votre écran il sera peut-être nécessaire d’ajuster la valeur. Ensuite, dans la partie gauche, saisissez  `neofetch --off | lolcat`, et dans la partie droite, entrez simplement  `peaclock`. Et voilà !  
  
Edit du 11/08/2025 : si vous souhaitez utiliser fastfetch au lieu de neofetch, vous pouvez l’installer avec  `pkg install fastfetch`  et taper  `fastfetch -l none | lolcat`  pour afficher la commande !
