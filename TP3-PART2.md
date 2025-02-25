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

## 2. SSH old friend

Keskivien foutr là tu vas me dire. OpenSSH, ce bro, comme d'hab, va nous faire le café.

On peut indiquer dans la conf OpenSSH qu'un nouvel utilisateur doit être automatiquement `chroot`é dans un dossier donné quand il se connecte.

🌞 **Créez un user `imsad`**

🌞 **Modifier la configuration du serveur SSH**

- uniquement quand le user `imsad` se connecte en SSH :
  - il est `chroot`é dans `/srv/get_chrooted/`
  - son shell doit fonctionner normalement
  - il se connecte avec une clé SSH
  - il ne doit pas pouvoir remarquer qu'il est `chroot`é dans un dossier particulier
- dans le compte-rendu :
  - montrez une connexion SSH fonctionnelle sur le user `imsad`
  - prouvez que le `bash` ouvert est bien `chroot`é
