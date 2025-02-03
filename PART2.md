
# Part II : Files and permissions

## 1. Listing POSIX permissions

🌞 **Déterminer les permissions des fichiers/dossiers...**

- le fichier qui contient la liste des utilisateurs
```console
ls -l /etc/passwd
-rw-r--r--. 1 root root 1411 Feb  3 12:49 /etc/passwd
```
- le fichier qui contient la liste des hashes des mots de passe des utilisateurs
```console
ls -l /etc/shadow
----------. 1 root root 1008 Feb  3 12:44 /etc/shadow
```
- le fichier de configuration du serveur OpenSSH
```console
ls -l /etc/ssh/sshd_config
-rw-------. 1 root root 3716 Feb  3 10:44 /etc/ssh/sshd_config
```
- le répertoire personnel de l'utilisateur `root`
```console
ls -ld /root
dr-xr-x---. 3 root root 142 Feb  3 14:58 /root
```
le répertoire personnel de votre utilisateur
```console
ls -ld /home/jeanc
drwx------. 3 jeanc jeanc 111 Feb  3 14:38 /home/jeanc
```
le programme `ls`
```console
ls -l /bin/ls
-rwxr-xr-x. 1 root root 140872 Apr 20  2024 /bin/ls
```
le programme `systemctl`
```console
ls -l /bin/systemctl
-rwxr-xr-x. 1 root root 305680 Apr  8  2024 /bin/systemctl
```

## 2. Extended attributes

🌞 **Lister tous les programmes qui ont le bit SUID activé**
```console
find / -perm -4000 -type f -exec ls -l {} \; 2>/dev/null
```
- Explications : 
- `/` → Recherche depuis la racine du système.
-   `-perm -4000` → Recherche les fichiers ayant le bit **SUID** activé.
-   `-type f` → Recherche uniquement des fichiers (évite les répertoires).
-   `2>/dev/null` → Supprime les erreurs liées aux permissions (sinon, beaucoup d’erreurs s'afficheront). 

🌞 **Rendre le fichier de configuration du serveur OpenSSH immuable**

- ça se fait avec les attributs étendus
- "immuable" ça veut dire qu'il ne peut plus être modifié DU TOUT : il est donc en read-only
- prouvez que le fichier ne peut plus être modifié
```console
sudo chattr +i /etc/ssh/sshd_config
```
```console
sudo lsattr /etc/ssh/sshd_config
----i----------------- /etc/ssh/sshd_config
```
Le `i` indique que le fichier est **immuable**.

En lancant la commande :
```console
sudo nano /etc/ssh/sshd_config
```
Ca indique bien que le fichier est en read-only :[ File '/etc/ssh/sshd_config' is unwritable ]
## 3. Protect a file using permissions

🌞 **Restreindre l'accès à un fichier personnel**

- créer un fichier nommé `dont_readme.txt` (avec le contenu de votre choix)
- il doit se trouver dans un dossier lisible et écrivable par tout le monde
- faites en sorte que seul votre utilisateur (pas votre groupe) puisse lire ou modifier ce fichier
- personne ne doit pouvoir l'exécuter
- prouvez que :
  - votre utilisateur peut le lire
  - votre utilisateur peut le modifier
  - l'utilisateur `meow` ne peut pas y toucher
  - l'utilisateur `root` peut quand même y toucher
1. Création du répertoire "public_folder"
```console
mkdir /tmp/public_folder
chmod 777 /tmp/public_folder
```
`/tmp/public_folder` → Ce dossier est **lisible et modifiable par tout le monde** (`777`) !

2. Ajout d'un fichier texte avec un contenu.
 ```console
 echo "Ceci est un fichier privé pour Léo :D" > /tmp/public_folder/dont_readme.txt
```
3. Permission pour notre user "jeanc"
```console 
[jeanc@efrei-xmg4agau1 ~]$ sudo chown $USER:$USER /tmp/public_folder/dont_readme.txt
[jeanc@efrei-xmg4agau1 ~]$ chmod 600 /tmp/public_folder/dont_readme.txt
[jeanc@efrei-xmg4agau1 ~]$ ls -l /tmp/public_folder/dont_readme.txt
-rw-------. 1 jeanc jeanc 40 Feb  3 15:56 /tmp/public_folder/dont_readme.txt
```
4. Vérifications :
- Droit d'afficher :
```console
[jeanc@efrei-xmg4agau1 ~]$ cat /tmp/public_folder/dont_readme.txt
Ceci est un fichier privé pour Léo :D
```
- Droit de modifier par mon user : 
```console
[jeanc@efrei-xmg4agau1 ~]$ echo "Ha mince ! Je parlais pas de notre intervanant mais de Léa aha :D" >> /tmp/public_folder/dont_readme.txt
[jeanc@efrei-xmg4agau1 ~]$ cat /tmp/public_folder/dont_readme.txt
Ceci est un fichier privé pour Léo :D
Ha mince ! Je parlais pas de notre intervanant mais de Léa aha :D
```
- Testons avec Meow l'affichage du fichier:
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo -u meow cat /tmp/public_folder/dont_readme.txt
cat: /tmp/public_folder/dont_readme.txt: Permission denied
```
- Et pour la modification ? :
```console
echo "Tentative de modification" >> /tmp/public_folder/dont_readme.txt
-bash: /tmp/public_folder/dont_readme.txt: Permission denied
```
- En root maintenant : 
```console 
[root@localhost public_folder]# cat /tmp/public_folder/dont_readme.txt
Ceci est un fichier privé.
Ha mince ! Je parlais pas de notre intervanant mais de Léa aha :D
```
