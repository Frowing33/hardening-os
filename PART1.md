# hardening-os

# Part I : User management

## 1. Existing users

🌞 **Déterminer l'existant :**
- lister tous les utilisateurs créés sur la machine
```console
[jeanc@efrei-xmg4agau1 ~]$ cut -d: -f1 /etc/passwd
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
operator
games
ftp
nobody
tss
systemd-coredump
dbus
polkitd
sssd
libstoragemgmt
setroubleshoot
cockpit-ws
cockpit-wsinstance
sshd
chrony
tcpdump
jeanc
```
- lister tous les groupes d'utilisateur
``` console 
[jeanc@efrei-xmg4agau1 ~]$ cut -d: -f1 /etc/group
root
bin
daemon
sys
adm
tty
disk
etc ...
```
Il existe plusieurs façon d'afficher les groupes par exemple on peut également executer : 
```
less /etc/group
```

Dans notre cas, j'ai utilisé cut afin de pouvoir afficher uniquement le nom du groupe et par connaissance de cette commande.

- déterminer la liste des groupes dans lesquels se trouvent votre utilisateur
``` console
[jeanc@efrei-xmg4agau1 ~]$ groups $(whoami)
jeanc : jeanc wheel
```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par `root`**
```console
[jeanc@efrei-xmg4agau1 ~]$ ps -u root
    PID TTY          TIME CMD
      1 ?        00:00:02 systemd
      2 ?        00:00:00 kthreadd
      3 ?        00:00:00 rcu_gp
      4 ?        00:00:00 rcu_par_gp
      5 ?        00:00:00 slub_flushwq
      6 ?        00:00:00 netns
      8 ?        00:00:00 kworker/0:0H-events_highpri
      9 ?        00:00:00 kworker/u8:0-events_unbound
```
🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par votre utilisateur**
```console
[jeanc@efrei-xmg4agau1 ~]$ ps aux | grep "^$(whoami)"
jeanc       1057  0.0  0.3  22892 13652 ?        Ss   10:50   0:00 /usr/lib/systemd/systemd --user
jeanc       1059  0.0  0.2  26516  7628 ?        S    10:50   0:00 (sd-pam)
jeanc       1066  0.0  0.1   9088  5632 tty1     Ss+  10:50   0:00 -bash
jeanc       1704  0.0  0.1  19608  7092 ?        S    10:58   0:00 sshd: jeanc@pts/0
jeanc       1705  0.0  0.1   8688  5376 pts/0    Ss   10:58   0:00 -bash
jeanc       1774  0.0  0.0  10084  3328 pts/0    R+   11:13   0:00 ps aux
jeanc       1775  0.0  0.0   6408  2176 pts/0    S+   11:13   0:00 grep --color=auto ^jeanc
```

🌞 **Déterminer le hash du mot de passe de `root`**
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo cat /etc/shadow | grep root
root:$6$y9hqf9f82TAGoyiz$xuFwonhyIRZVkyIs4d6K0/p7Y71ASgGXAGzkwQLmSfQCFHpK0HT.IJqHnX6ZYcRs3.WlFwTAcIlz2OyO71aBs.:20122:0:99999:7:::
```
🌞 **Déterminer le hash du mot de passe de votre utilisateur**
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo cat /etc/shadow | grep jeanc
jeanc:$6$Kkt2xTtdxEt3DjFS$6sx0d4h07lLnSklkQ8lb6nPksD3hSEZoeCOJEDTURW2FCWc05aRbPJ0cCLzJAGljK/TNzWy4JnNpJFWuDDYu90::0:99999:7:::
```
🌞 **Déterminer la fonction de hachage qui a été utilisée**
La fonction de hachage utilisée est le SHA-512.
Nous pouvons le déterminé en regardant les 3 premiers caractères du hash : $6 $ 
🌞 **Déterminer, pour l'utilisateur `root`** :
- son shell par défaut :
```console 
[jeanc@efrei-xmg4agau1 ~]$ getent passwd root
root:x:0:0:root:/root:/bin/bash
```
- le chemin vers son répertoire personnel
```console
[jeanc@efrei-xmg4agau1 ~]$ getent passwd root | cut -d: -f6
/root
```
Pour information voici certaines explications pour la commande cut : 
`cut -d: -f6` extrait le 6ᵉ champ (`/root`) de la ligne obtenue avec `getent passwd root`, en utilisant `:` comme séparateur.
🌞 **Déterminer, pour votre utilisateur** :
- son shell par défaut
```console
[jeanc@efrei-xmg4agau1 ~]$ getent passwd $USER
jeanc:x:1000:1000:Jean COMBERTON:/home/jeanc:/bin/bash
```
- le chemin vers son répertoire personnel
```console 
[jeanc@efrei-xmg4agau1 ~]$ getent passwd $USER | cut -d: -f6
/home/jeanc
```
🌞 **Afficher la ligne de configuration du fichier `sudoers` qui permet à votre utilisateur d'utiliser `sudo`**
```console
[jeanc@efrei-xmg4agau1 etc]$ sudo visudo
## Fichier /etc/sudoers

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
```



## 2. User creation and configuration

🌞 **Créer un utilisateur :**

- doit s'appeler `meow`
- ne doit appartenir QUE à un groupe nommé `admins`
- ne doit pas avoir de répertoire personnel utilisable
- ne doit pas avoir un shell utilisable
1.  Création du groupe "admins"
```console
[jeanc@efrei-xmg4agau1 etc]$ sudo groupadd admins
```
2. Création  du user : "meow" avec les contraintes données :
```console
[jeanc@efrei-xmg4agau1 etc]$ sudo useradd -M -N -g 
admins -s /sbin/nologin meow
```
3. Vérifications du user meow 
```console
[jeanc@efrei-xmg4agau1 etc]$ id meow
uid=1001(meow) gid=1001(admins) groups=1001(admins) 
```
```console
[jeanc@efrei-xmg4agau1 home]$ ls -l /home
total 0
drwx------. 3 jeanc jeanc 111 Feb  3 11:11 jeanc
```
Ici on n'a pas le répertoire meow
```console
[jeanc@efrei-xmg4agau1 home]$ grep "^meow:" /etc/passwd
meow:x:1001:1001::/home/meow:/sbin/nologin
```
On constate bien que meow n'a pas de shell défini.

🌞 **Configuration `sudoers`**

- ajouter une configuration `sudoers` pour que l'utilisateur `meow` puisse exécuter seulement et uniquement les commandes `ls`, `cat`, `less` et `more` en tant que votre utilisateur
```console 
[jeanc@efrei-xmg4agau1 ~]$ sudo su -s /bin/bash - meow
su: warning: cannot change directory to /home/meow: No such file or directory
[meow@efrei-xmg4agau1 jeanc]$
```

- Ajouter une configuration `sudoers` pour que les membres du groupe `admins` puisse exécuter seulement et uniquement la commande `dnf` en tant que `root`
Voici la ligne de configuration à intégrer : 
```console
%admins ALL=(ALL) NOPASSWD: /usr/bin/dnf
```
Pour tester : 
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo su -s /bin/bash - meow
su: warning: cannot change directory to /home/meow: No such file or directory
[meow@efrei-xmg4agau1 jeanc]$ sudo dnf update
Last metadata expiration check: 1:04:45 ago on Mon 03 Feb 2025 11:56:20 AM CET.
Dependencies resolved.
============================================================================
 Package                    Arch   Version                  Repo       Size
============================================================================
Installing:
 kernel                     x86_64 5.14.0-503.22.1.el9_5    baseos    2.0 M
 etc ......
```
- ajouter une configuration `sudoers` pour que votre utilisateur puisse exécuter n'importe quel commande en tant `root`, sans avoir besoin de saisir un mot de passe

Afin d'effectuer cette configuration nous devons modifier le fichier de configuration :
```console
sudo visudo
```
```console
jeanc ALL=(ALL) NOPASSWD: ALL
```
- prouvez que ces 3 configurations ont pris effet (vous devez vous authentifier avec le bon utilisateur, et faire une commande `sudo` qui doit fonctioner correctement)

1. Démonstration Meow (ls,cat etc...)
```console 
[meow@efrei-xmg4agau1 jeanc]$ sudo -u jeanc cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin

[meow@efrei-xmg4agau1 jeanc]$ sudo -u jeanc echo "test"
-bash: ton_utilisateur: Permission denied
```
2. Démonstration admins
Ici on lance la commande "dnf", et ça fonctionne bien.
```console
sudo dnf update
Last metadata expiration check: 2:29:45 ago on Mon 03 Feb 2025 11:56:20 AM CET.
Dependencies resolved.
```
Alors que mkdir ne fonctionne pas car il n'a pas les autorisations :
```console
[meow@efrei-xmg4agau1 jeanc]$ sudo mkdir test
[sudo] password for meow:
Sorry, user meow is not allowed to execute '/bin/mkdir test' as root on efrei-xmg4agau1.campus.villejuif.
```
3. Démonstration root pour notre user :
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo whoami
root
```
On constate qu'il est bien en root, et **nous demande pas de mot de passe !**
## 3. Hackers gonna hack

🌞 **Déjà une configuration faible ?**

- l'utilisateur `meow` est en réalité complètement `root` sur la machine hein là. Prouvez-le.
- proposez une configuration similaire, sans présenter cette faiblesse de configuration
  - vous pouvez ajouter de la configuration
  - ou supprimer de la configuration
  - du moment qu'on garde des fonctionnalités à peu près équivalentes !

1. Pourquoi la configuration de l'utilisateur 'meow' est en réalité 'root'  ?

L'utilisateur `meow` peut exécuter `cat`, `less` ou `more` pour **lire n'importe quel fichier accessible par `jeanc`**.

Cependant, `less` et `more` permettent **d'exécuter des commandes shell** avec `!sh`. Par exemple, si `meow` exécute :
```console
sudo -u jeanc less /etc/passwd
```
et qu'il exécute la commande : ```console !sh```, il accède au shell...
Pour faire simple, la configuration actuelle **permet à `meow` de devenir root** via `jeanc` !

Pour éviter ça, nous allons supprimer more et less.
```console
meow ALL=(ALL) /bin/ls, /bin/cat
```
```console
[meow@efrei-xmg4agau1 jeanc]$ sudo -u jeanc ls /
afs  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[meow@efrei-xmg4agau1 jeanc]$ sudo -u jeanc more /etc/passwd
[sudo] password for meow:
Sorry, user meow is not allowed to execute '/bin/more /etc/passwd' as jeanc on efrei-xmg4agau1.campus.villejuif.
```
