
# Part III : Service Hardening

➜ **OK OK OK OK BON. Vous voyez où on va ? Filtrage de *syscalls* !**

Grâce à la partie précédente, vous avez appris à *tracer* un programme : regarder tous les *syscalls* qu'il appelle pendant son fonctionnement.

**Si on sait quels *syscalls* appelle un programme dans son fonctionnement normal, on peut donc dire qu'il n'a pas besoin des autres ! On va voir comment empêcher un programme de passer certains *syscalls*.**

> Genre un appel à `execve("/bin/bash")`, ne cherche pas plus loin, si c'est pas spécifiquement prévu, **c'est un hack**.

Par exemple, on rappelle qu'un appel à un syscall est **rigoureusement nécessaire** si un programme veut :

- lire/écrire dans un fichier
- exécuter un nouveau programme
- utiliser le réseau
- changer des permissions
- et bien d'autres

**Autrement dit, surveiller les sycalls que passe un programme, c'est surveiller ce qu'il demande au système et c'est avoir une vue très fine sur des comportements potentiellements anormaux.**

➜ **Le mécanisme du kernel Linux qui permet de filter les *syscalls*  que fait un programme s'appelle `seccomp`.**

On utilise donc un profil `seccomp` pour filtrer ce qu'a le droit de faire un processus ou non.

Chaque processus lancé peut être lancé avec une whitelist des *syscalls* qu'il a le droit d'appeler.

Tout autre appel sera bloqué.

➜ **On va utiliser le classique serveur Web NGINX dans cette partie comme exemple !**

Un bon cas d'école, et loin d'être inutile tellement NGINX est partout aujourd'hui :)

➜ Avec **systemd** (le gestionnaire de services de Linux), **il est aisé d'appliquer un profil `seccomp` à un service.**

En ajoutant une clause `SystemCallFilter=` à la définition du service, on peut lister les *syscalls* qu'un service aura le droit d'effectuer.


## 1. Install NGINX

➜ **Installer et démarrer le serveur Web NGINX sur la machine**

- le paquet s'appelle `nginx` sous Rocky
- démarrer le service, ouvrez le port firewall, visitez le site web
- assurez-vous que ça marche correctement quoi
- **puis stoppez le service**

➜ **Visualiser la définition du service NGINX**

- chaque service Linux est défini dans un fichier `.service`
- vous pouvez afficher le chemin et le contenu du fichier associé à un service avec `systemctl cat` :

```bash
sudo systemctl cat nginx
```

> Soyez attentif un peu à son contenu, il faudra écrire par vous-mêmes un service à la partie suivante. Il sera bon de s'inspirer de celui-ci !

➜ **La ligne la plus importante du fichier, c'est celle qui commence par `ExecStart=`**.

- c'est la commande qui est lancée quand vous faites un `sudo systemctl start nginx`
- **autrement dit, lancer cette commande à la main, c'est lancer le programme NGINX à la main**, sans passer par le service
- pourquoi faire ça ? Well...

## 2. NGINX Tracing

🌞 **Tracer l'exécution du programme NGINX**

- lancer NGINX à la main, et utilisez `strace` ou `sysdig` pour voir tous les appels systèmes qu'il effectue
- visitez la page web d'accueil pendant que vous tracez l'exécution, pour voir les *syscalls*  nécessaires lors d'un fonctionnement normal
- dans le compte-rendu, listez tous les *syscalls*  passés par NGINX
```console
jeanc@srv-ubuntu:/home$ sudo strace -c /usr/sbin/nginx -g 'daemon off;'
^Cstrace: Process 19390 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 40,19    0,000473         118         4           clone
  7,39    0,000087          14         6           sendmsg
  6,46    0,000076           3        23         1 openat
  6,37    0,000075           6        12           rt_sigaction
  6,12    0,000072           5        14           ioctl
  5,35    0,000063           4        14           fcntl
  5,10    0,000060          15         4           socketpair
  4,42    0,000052          52         1         1 rt_sigsuspend
  2,63    0,000031           6         5         5 mkdir
  2,63    0,000031           3         9           newfstatat
  2,38    0,000028          14         2           bind
  2,29    0,000027           4         6           socket
  1,87    0,000022           5         4           listen
  1,78    0,000021          21         1           pwrite64
  1,44    0,000017           5         3           setsockopt
  1,27    0,000015           0        25           close
  1,27    0,000015           0        32           mmap
  0,42    0,000005           5         1           dup2
  0,34    0,000004           4         1           rt_sigprocmask
  0,25    0,000003           1         3           prlimit64
  0,00    0,000000           0        18           read
  0,00    0,000000           0        20           fstat
  0,00    0,000000           0         3           lseek
  0,00    0,000000           0         8           mprotect
  0,00    0,000000           0         1           munmap
  0,00    0,000000           0         6           brk
  0,00    0,000000           0         6           pread64
  0,00    0,000000           0         1         1 access
  0,00    0,000000           0         1           getpid
  0,00    0,000000           0         4         4 connect
  0,00    0,000000           0         1           execve
  0,00    0,000000           0         2           uname
  0,00    0,000000           0         1           geteuid
  0,00    0,000000           0         1           getppid
  0,00    0,000000           0         1           arch_prctl
  0,00    0,000000           0        19           futex
  0,00    0,000000           0         1           epoll_create
  0,00    0,000000           0         6           getdents64
  0,00    0,000000           0         1           set_tid_address
  0,00    0,000000           0         1           set_robust_list
  0,00    0,000000           0         1           getrandom
  0,00    0,000000           0         1           rseq
------ ----------- ----------- --------- --------- ----------------
100,00    0,001177           4       274        12 total
```
## 3. NGINX Hardening

🌞 **HARDEN**

- modifier le fichier `nginx.service` pour inclure un filtrage des *syscalls*
- principe du moindre privilège : vous n'autorisez que le strict nécessaire
- vous me remettez le fichier `nginx.service` modifié dans le compte-rendu naturellement !

Voici le fichier du service NGINX modifié, je me suis aidé de : https://linux-audit.com/systemd/systemd-syscall-filtering/
```console
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=mixed
PrivateTmp=true

# Filtrage des syscalls avec seccomp :
# Seuls les appels systèmes listés sont autorisés.
# (La liste proposée est indicative. Ajustez-la selon vos observations avec strace/sysdig.)

#SystemCallFilter=read write open openat close access stat fstat newfstatat lstat pread64 pwrite64 readlink lseek mmap mprotect munmap brk rt_sigaction rt_sigprocmask rt_sigreturn ioctl poll sysinfo clock_gettim>
SystemCallFilter=@system-service @network-io @file-system

[Install]
WantedBy=multi-user.target

```

