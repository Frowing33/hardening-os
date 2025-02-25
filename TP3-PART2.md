# Part II. Gotta get chrooty

➜ **`chroot` c'est le vieux de la vieille de l'isolation de processus.** 

Il est là depuis si longtemps le frérot, ça marche toujours aussi bien, mais c'est vrai qu'il paraît limité pour les standards d'isolation d'aujourd'hui.

> *Ca reste un grand classique vous allez pas y couper !*

Le principe de `chroot` c'est de changer l'emplacement de la racine du disque pour un processus donné.

Genre on fait croire à un programme que son dossier `/` c'est genre `/toto/super_chroot/`. Comme ça quand il fait `ls /`, il l'ignore, mais en réalité, il liste le contenu du dossier `/toto/super_chroot/`.

> Pour que ça fonctionne normalement et correctement il n'est pas rare de remettre les dossiers qu'on trouve habituellement à la racine du disque dans `/toto/super_chroot/`

## Sommaire

- [Part II. Gotta get chrooty]
  - [Sommaire]
  - [1. Play manually]
  - [2. SSH old friend]

## 1. Play manually

🌞 **Créez le dossier `/srv/get_chrooted/`**

```console
[jeanc@efrei-xmg4agau1 ~]$ sudo mkdir -p /srv/get_chrooted
```
🌞 **Essayez de `chroot` à l'intérieur en lançant un shell**

- possible que ça fonctionne pas immédiatement car y'a pas de shells dans votre `chroot` :d
- déplacez le nécessaire dans `/srv/get_chrooted/` pour pouvoir lancé un shell `chroot`é à l'intérieur
* Nous devons voir quelles bibliothèques sont nécessaires pour que le shell fonctionne.

```console
[jeanc@efrei-xmg4agau1 ~]$ ldd /bin/bash
        linux-vdso.so.1 (0x00007ffe0a7f5000)
        libtinfo.so.6 => /lib64/libtinfo.so.6 (0x00007f0e93ca9000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f0e93a00000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f0e93e3a000)
```
On va copier les dépendances :
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo cp -v /lib64/libtinfo.so.6 /srv/get_chrooted/lib64/
sudo cp -v /lib64/libc.so.6 /srv/get_chrooted/lib64/
sudo cp -v /lib64/ld-linux-x86-64.so.2 /srv/get_chrooted/lib64/
```
Une fois les lib minimum copiées dans le répertoire.
On peut maintenant essayer de chrooter avec le shell
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo chroot /srv/get_chrooted /bin/bash
bash-5.1#
```
Ca marche nickel :D
## 2. SSH old friend

Keskivien foutr là tu vas me dire. OpenSSH, ce bro, comme d'hab, va nous faire le café.

On peut indiquer dans la conf OpenSSH qu'un nouvel utilisateur doit être automatiquement `chroot`é dans un dossier donné quand il se connecte.

🌞 **Créez un user `imsad`**
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo useradd -m -s /bin/bash imsad
```

🌞 **Modifier la configuration du serveur SSH**
Génération de la clé ssh pour l'accès en SSH de "imsad" sur son environement chrooté.
```console
[jeanc@efrei-xmg4agau1 home]$ sudo ls -l /srv/get_chrooted/home/imsad/.ssh/
```
- uniquement quand le user `imsad` se connecte en SSH :
  - il est `chroot`é dans `/srv/get_chrooted/`
  - son shell doit fonctionner normalement
  - il se connecte avec une clé SSH
  - il ne doit pas pouvoir remarquer qu'il est `chroot`é dans un dossier particulier
- dans le compte-rendu :
  - montrez une connexion SSH fonctionnelle sur le user `imsad`
  - prouvez que le `bash` ouvert est bien `chroot`é
