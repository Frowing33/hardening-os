
# Part I : Learn

## 1. Anatomy of a program

### A. `file`

`file` est une commande uqi permet de déterminer le type d'un fichier.

Ceci ne repose pas du tout sur l'extension du fichier. `file` regarde directement les bits qui composent le fichier pour en déterminer le type. Il se concentre sur les premiers octets du fichiers qui contient généralement des métadonnées suffisantes pour déterminer le type.

🌞 **Utiliser `file` pour déterminer le type de :**

- la commande `ls` :
```console
jeanc@srv-ubuntu:~$ file /usr/bin/ls
/usr/bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3eca7e3905b37d48cf0a88b576faa7b95cc3097b, for GNU/Linux 3.2.0, stripped
```
- la commande `ip` :
```console
jeanc@srv-ubuntu:~$ file /usr/sbin/ip
/usr/sbin/ip: symbolic link to /bin/ip
```
Ici on constate un lien symoblique, on va donc refaire la commande avec le chemin d'origine : 
```console
jeanc@srv-ubuntu:~$ file /usr/bin/ip
/usr/bin/ip: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=1ae3de4ab46efbf676158c3cf6a41908da57689d, for GNU/Linux 3.2.0, stripped
```
- un fichier `.mp3` que vous aurez téléchargé sur le disque de la VM
```console
jeanc@srv-ubuntu:~$ file Sound.mp3
Sound.mp3: Audio file with ID3 version 2.3.0, contains: MPEG ADTS, layer III, v1, 320 kbps, 44.1 kHz, JntStereo
```
### B. `readelf`

`readelf` permet d'obtenir des informations sur un fichier ELF : un exécutable Linux.
`readelf` permet notamment de voir de quelle adresse à quelle adresse se trouve  tell ou telle section.

🌞 **Utiliser `readelf` sur le programme `ls`**
Il faut au préalable installer la dépendance : 
```console 
jeanc@srv-ubuntu:~$ sudo apt install binutils
```
- afficher le *header* du programme
  - il contient toutes les métadonnées principales du programme
  - c'est l'option `readelf -h`
 ```console
jeanc@srv-ubuntu:~$ readelf -h /usr/bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x6d30
  Start of program headers:          64 (bytes into file)
  Start of section headers:          140328 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```
- afficher la liste des sections du programme
  - c'est l'option `readelf -S`
```console 
jeanc@srv-ubuntu:~$ readelf -S /usr/bin/ls
There are 31 section headers, starting at offset 0x22428:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000000338  00000338
       0000000000000030  0000000000000000   A       0     0     8
etc ...
```
- déterminer à quel adresse commence le code du programme
  - pour rappel, le code est dans la section `.text`
```console 
jeanc@srv-ubuntu:~$ readelf -S /bin/ls | grep .text
  [16] .text             PROGBITS         0000000000004d70  00004d70
```
  - vous devriez voir cette adresse dans la sortie de `readelf -S`

### C. `ldd`

`ldd` est un outil qui permet de manipuler le *dynamic linker* de Linux. Le *dynamic linker* c'est un programme qui s'occupe de trouver les librairies nécessaires quand un autre programme se lance.

**On peut utiliser `ldd` notamment pour visualiser de quelle librairie a besoin un programme donné.**
```console
jeanc@srv-ubuntu:~$ ldd /bin/ls
        linux-vdso.so.1 (0x00007ffcadfe3000)
        libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x000070e53ada1000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x000070e53aa00000)
        libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x000070e53ad07000)
        /lib64/ld-linux-x86-64.so.2 (0x000070e53adfb000)
```
🌞 **Utiliser `ldd` sur le programme `ls`**

- afficher la liste des librairies que va utiliser `ls` pendant son fonctionnement
```console
jeanc@srv-ubuntu:~$ ldd /bin/ls
        linux-vdso.so.1 (0x00007ffcadfe3000)
        libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x000070e53ada1000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x000070e53aa00000)
        libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x000070e53ad07000)
        /lib64/ld-linux-x86-64.so.2 (0x000070e53adfb000)
```
- déterminer, parmi ces librairies, laquelle est la Glibc
```console
jeanc@srv-ubuntu:~$ ldd /bin/ls | grep libc
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ddf42e00000)
```
> La Glibc est une des librairies les plus importantes au sein d'un système Linux, car elle contient notamment tout le nécessaire pour passer des *syscalls* élémentaires. Si un programme souhaite lire ou écrire dans un fichier par exemple, il aura besoin d'inclure la Glibc.

## 2. Syscalls basics

### A. Syscall list

> Vous pourrez trouver une [liste des syscalls Linux sur un système x86_64 iciiii](https://filippo.io/linux-syscall-table/).

🌞 **Donner le nom ET l'identifiant unique d'un syscall qui permet à un processus de...**

- lire un fichier stocké sur disque
**Nom du syscall** : `read`
**Identifiant unique** : `0`
- écrire dans un fichier stocké sur disque
**Nom du syscall** : `write`
**Identifiant unique** : `1`
- lancer un nouveau processus
**Nom du syscall** : `fork` ou `execve`
**Identifiant unique (x86_64)** :
`fork` : `57`
`execve` : `59`
**Description** :
`fork` crée un **nouveau processus** en clonant l'appelant.
`execve` **remplace** le processus courant par un nouveau programme.

> Pour la suite du TP, gardez-vous sous le coude les réponses apportées à cette question. Juste après vous allez regarder le langage machine contenu dans des exécutables à la recherche de l'appel à un *syscall*. Il faudra le repérer grâce à son identifiant !

![Fork exec](./img/forkexec.png)

### B. `objdump`

`objdump` permet de désassembler un programme, c'est à dire d'afficher le code contenu par un exécutable, sous forme de langage assembleur compréhensible par les humains (un peu, beaucoup plus qu'une purée d'octets en tout cas !)

🌞 **Utiliser `objdump`** sur la commande `ls`

- afficher le contenu de la section `.text`
  - je vous laisse trouver la commande sur l'internet :D
```console
jeanc@srv-ubuntu:~$ objdump -d -j .text /bin/ls
```
- mettez en évidence quelques lignes qui contiennent l'instruction `call`
  - il devrait y en avoir plusieurs
  - chaque `call` est un appel à une fonction, potentiellement importée *via* une librairie
```console
jeanc@srv-ubuntu:~$ objdump -d -j .text /bin/ls | grep 'call'
    4d70:       e8 db f9 ff ff          call   4750 <abort@plt>
    4d75:       e8 d6 f9 ff ff          call   4750 <abort@plt>
    4d7a:       e8 d1 f9 ff ff          call   4750 <abort@plt>
    4d7f:       e8 cc f9 ff ff          call   4750 <abort@plt>
    4d84:       e8 c7 f9 ff ff          call   4750 <abort@plt>
    4d89:       e8 c2 f9 ff ff          call   4750 <abort@plt>
    4d8e:       e8 bd f9 ff ff          call   4750 <abort@plt>
    4de8:       e8 83 fb ff ff          call   4970 <strrchr@plt>
    4e16:       e8 55 f9 ff ff          call   4770 <strncmp@plt>
    4e5c:       e8 5f fd ff ff          call   4bc0 <setlocale@plt>
```
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
  - il y en a aucune normalement : `ls` ne contient pas directement de syscalls
  - car il importe la Glibc, qui contient des syscalls, et les appelle avec `call`
```console
jeanc@srv-ubuntu:~$ objdump -d -j .text /bin/ls | grep syscall
jeanc@srv-ubuntu:~$
```

🌞 **Utiliser `objdump`** sur la librairie Glibc

- vous avez repéré son chemin exact au point d'avant avec `ldd`
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
  - il devrait y en avoir pas mal
  - chaque ligne qui contient l'instruction `syscall` est la dernière d'un bloc de code qui est le syscall lui-même
- trouvez l'instrution `syscall` qui exécute le syscall `open()`
```console
1af87d:       b8 02 00 00 00          mov    $0x2,%eax
  1af882:       48 89 04 24             mov    %rax,(%rsp)
  1af886:       e8 85 03 00 00          call   1afc10 <__nss_database_lookup@GLIBC_2.2.5+0x27c30>
```

> Pour exécuter un `syscall`, le programme met dans le registre `eax` l'identifiant du syscall (avec l'instruction `mov`) puis exécute l'instruction `syscall`. Vous cherchez donc une instruction `syscall` précédé d'un `mov` qui met l'identifiant de `open()` dans `eax`.
