
  
# Part II : Observe

**Il est possible d'observer en temps réel ce que fait un programme. On dit qu'on peut *tracer* un programme.**

Plusieurs techniques pour faire ça, suivant ce qu'on veut voir ; dans ce TP on va se concentrer sur les *syscalls*.

L'outil le plus élémentaire à connaître est `strace`. Il s'utilise en terminal et affiche tous les *syscalls*  que réalisent un processus.

On va aussi utiliser `sysdig` plus moderne et plus puissant.

## Sommaire

- [Part II : Observe](#part-ii--observe)
  - [Sommaire](#sommaire)
  - [1. strace](#1-strace)
  - [2. sysdig](#2-sysdig)
    - [A. Intro](#a-intro)
    - [B. Use it](#b-use-it)
  - [3. Bonus : Stratoshark](#3-bonus--stratoshark)

## 1. strace
🌞 **Utiliser `strace` pour tracer l'exécution de la commande `ls`**

- faites `ls` sur un dossier qui contient des trucs
```console
jeanc@srv-ubuntu:/home$ strace cat /home/jeanc/strace_cat
```
- mettez en évidence le *syscall* pour écrire dans le terminal le résultat du `ls`
```console
jeanc@srv-ubuntu:/home$ strace ls /home/jeanc/
execve("/usr/bin/ls", ["ls", "/home/jeanc/"], 0x7fffb4c6b688 /* 23 vars */) = 0
brk(NULL)                               = 0x5d31df872000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7ca565a52000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
...
write(1, "Sound.mp3\n", 10Sound.mp3)             = 10
```
Voici le syscall pour écrire dans le terminal le résultat du ls **write(1, "Sound.mp3\n", 10Sound.mp3)             = 10**

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `cat`**

- faites `cat` sur un fichier qui contient des trucs
- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
Voici le syscall qui demande l'ouverture du fichier en lecture:
```console
openat(AT_FDCWD, "/etc/ld.so.cache",O_RDONLY|O_CLOEXEC)= 3
```
**Opencat ouvre un fichier en utilisant le descripteur de fichier du répertoire spécifié comme emplacement de départ pour la recherche de chemin**
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal
```console
write(1, "Je suis fatigu\303\251 mais c'est int\303"..., 42Je suis fatigué mais c'est intéressant.
```
🌞 **Utiliser `strace` pour tracer l'exécution de `curl example.org`**

- vous devez utiliser une option de `strace`
- elle affiche juste un tableau qui liste tous les *syscalls*  appelés par la commande tracée, et combien de fois ils ont été appelé
```console
jeanc@srv-ubuntu:/home$ strace -c curl example.org
```

## 2. sysdig

### A. Intro


### B. Use it

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `ls`**

- faites `ls` sur un dossier qui contient des trucs (pas un dossier vide)
- mettez en évidence le *syscall* pour écrire dans le terminal le résultat du `ls`
```console
jeanc@srv-ubuntu:~$ sudo sysdig proc.name=ls
...
...
2733 15:32:51.801418520 3 ls (4577.4577) < write res=38 data=.[0m.[00;36mSound.mp3.[0m  strace_cat.
...
```
> Vous pouvez isoler à la main les lignes intéressantes : copier/coller de la commande, et des seule(s) ligne(s) que je vous demande de repérer.

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `cat`**

- faites `cat` sur un fichier qui contient des trucs
- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal
```console
jeanc@srv-ubuntu:~$ sudo sysdig proc.name=cat
...
6095 15:35:11.565579957 2 cat (4593.4593) > openat dirfd=-100(AT_FDCWD) name=strace_cat(/home/jeanc/strace_cat) flags=1(O_RDONLY) mode=0
...
6106 15:35:11.565814253 2 cat (4593.4593) < write res=42 data=Je suis fatigu.. mais c'est int..ressant..
```
🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par votre utilisateur**

- ça va bourriner sec, vu que vous êtes connectés en SSH étou
- juste pour vous éduquer un peu + à ce que fait le kernel à chaque seconde qui passe
- donner la commande pour ça, pas besoin de me mettre le résultat :d
```console
jeanc@srv-ubuntu:~$ sudo sysdig user.name=$USER
```
🌞 **Livrez le fichier `curl.scap` dans le dépôt git de rendu**

- `sysdig` permet d'enregistrer ce qu'il capture dans un fichier pour analyse ultérieure
- l'extension c'est `.scap` par convention
- **capturez les *syscalls*  effectués par un `curl example.org`**

> `sysdig` est un outil moderne qui sert de base à toute la suite d'outils de la boîte du même nom. On pense par exemple à Falco qui permet de tracer, monitorer, lever des alertes sur des *syscalls* , au sein d'un cluster Kubernetes.

## 3. Bonus : Stratoshark

Un tout nouveau tool bien stylé : [Stratoshark](https://wiki.wireshark.org/Stratoshark). L'interface de Wireshark (et ses fonctionnalités de fou) mais pour visualiser des captures de *syscalls*  (et autres).

Vous prenez pas trop la tête avec ça, mais si vous voulez vous amuser avec une interface stylée, il est là !

Vous pouvez exporter une capture `sysdig` avec `sysdig -w meo.scap proc.name=echo` par exemple, et la lire dans Stratoshark.
