# Part I : A bit of exploration

Commençons par un peu d'exploration manuelle des pseudo-filesystems que sont `/proc` et `/sys`.

⚠️⚠️⚠️ **Vous n'utiliserez que les commandes `cat`, `ls` et `cd` (ou commandes similaires comme du `grep` bien sûr) pour réaliser cette partie.**

![cat /proc](./img/cat_proc.png)

## Sommaire

- [Part I : A bit of exploration](#part-i--a-bit-of-exploration)
  - [Sommaire](#sommaire)
  - [1. /proc](#1-proc)
  - [2. /sys](#2-sys)

## 1. /proc

🌞 **Afficher...** :

- l'état complet de la mémoire (RAM)
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo cat /proc/meminfo
MemTotal:        3747020 kB
MemFree:         3316836 kB
MemAvailable:    3301844 kB
Buffers:            2708 kB
Cached:           172288 kB
...
```

- le nombre de coeurs que votre CPU a (uniquement ce nombre)
```console
[jeanc@efrei-xmg4agau1 ~]$ grep -c ^processor /proc/cpuinfo
4
```
- le nombre de processus lancés (uniquement ce nombre)
```console
[jeanc@efrei-xmg4agau1 ~]$ cat /proc/loadavg | awk '{print $4}' | cut -d'/' -f2
155
```
- la ligne de commande utilisée pour lancer le kernel actuel
```console
[jeanc@efrei-xmg4agau1 ~]$ cat /proc/cmdline
BOOT_IMAGE=(hd0,msdos1)/vmlinuz-5.14.0-427.13.1.el9_4.x86_64 root=/dev/mapper/rl_efrei--xmg4agau1-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rl_efrei--xmg4agau1-swap rd.lvm.lv=rl_efrei-xmg4agau1/root rd.lvm.lv=rl_efrei-xmg4agau1/swap rhgb quiet
```
- la liste des connexions TCP actuelles (même si c'est un peu imbuvable avec nos p'tits yeux)
```console
[jeanc@efrei-xmg4agau1 ~]$ cat /proc/net/tcp
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
   0: 0A99A8C0:0016 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 23049 1 0000000000000000 100 0 0 10 0
   1: 0A99A8C0:0016 0299A8C0:8A8D 01 00000034:00000000 01:00000017 00000000     0        0 23052 4 0000000000000000 23 6 21 10 -1
```
- la valeur actuelle de la *swappiness* (cf le tip ci-dessous)
```console
[jeanc@efrei-xmg4agau1 ~]$ cat /proc/sys/vm/swappiness
30
```

> La `swap` est une partition sur le disque qui va être utilisée automatiquement par l'OS si la mémoire (RAM) s'apprête à être pleine : l'OS va décharger une partie des machins en RAM pour les mettre sur la partition de `swap`. Ca rame de fou dukoo hein, mais ça continue de fonctionner. La *swappiness* détermine le pourcentage de remplissage de la mémoire à partir duquel l'OS va commencer à utiliser la `swap` (à "swapper" comme on dit :d).

## 2. /sys

> N'oubliez jamais les pages du `man`, c'est un très bonne doc souvent. [Là encore pour **sysfs** (`/sys`)](https://man7.org/linux/man-pages/man5/sysfs.5.html).

🌞 **Afficher...** :

- la liste des périphériques de types bloc reconnus par l'OS (genre les disques durs par exemple koa)
- la liste des modules kernel qui sont actuellements en cours d'utilisation
- la liste des cartes réseau
