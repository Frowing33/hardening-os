
# Part IV : Linux Namespaces

## 1. Explore

🌞 **Utiliser /proc**

- déterminer les *namespaces* de votre `bash` actuel
```console
[jeanc@Host-006 ~]$ ls -l /proc/self/ns
total 0
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 net -> 'net:[4026531840]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 jeanc jeanc 0  9 mars  11:12 uts -> 'uts:[4026531838]'
```
- déterminer les *namespaces* du processuis qui porte l'identifiant PID 1
```console
jeanc@Host-006 ~]$ sudo ls -l /proc/1/ns
total 0
lrwxrwxrwx. 1 root root 0  9 mars  11:14 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 net -> 'net:[4026531840]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0  9 mars  11:14 uts -> 'uts:[4026531838]'

```
- ils devraient être identiques

🌞 **Lister tous les *namespaces* en cours d'utilisation**

- avec une simple commande `lsns`
```console
[jeanc@Host-006 ~]$ lsns
        NS TYPE   NPROCS   PID USER  COMMAND
4026531834 time       67  5200 jeanc /usr/lib/systemd/systemd --user
4026531835 cgroup     67  5200 jeanc /usr/lib/systemd/systemd --user
4026531836 pid        67  5200 jeanc /usr/lib/systemd/systemd --user
4026531837 user       67  5200 jeanc /usr/lib/systemd/systemd --user
4026531838 uts        67  5200 jeanc /usr/lib/systemd/systemd --user
4026531839 ipc        67  5200 jeanc /usr/lib/systemd/systemd --user
4026531840 net        67  5200 jeanc /usr/lib/systemd/systemd --user
4026531841 mnt        67  5200 jeanc /usr/lib/systemd/systemd --user
```

## 2. Create

### A. net

🌞 **Créer un nouveau *namespace* `network`**

- avec une commande `unshare`
```console
[jeanc@Host-006 ~]$ sudo unshare --net bash
```
- lancez un `bash` à l'intérieur

> `unshare` est aussi le nom de l'appel système (syscall) qui permet de demander au kernel de créer un *namespace*. Ainsi, cette commande, c'est vraiment juste appeler ce syscall depuis la ligne de commande.

🌞 **Prouvez que votre nouveau *namespace* est bien là**

- déjà un `ip a` devrait montrer aucune des cartes réseau depuis l'intérieur du *namespace*
```console
[root@Host-006 jeanc]# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
- et on peut `lsns` pour voir ce nouveau *namespace*
```console
[root@Host-006 jeanc]# lsns
        NS TYPE   NPROCS   PID USER   COMMAND
4026531834 time      276     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531835 cgroup    276     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531836 pid       276     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531837 user      274     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531838 uts       271     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531839 ipc       276     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531840 net       272     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531841 mnt       258     1 root   /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
4026531862 mnt         1    47 root   kdevtmpfs
4026532498 mnt         1   680 root   /usr/lib/systemd/systemd-udevd
4026532499 uts         1   680 root   /usr/lib/systemd/systemd-udevd
4026532747 mnt         2   793 root   /sbin/auditd
4026532748 mnt         2   821 dbus   /usr/bin/dbus-broker-launch --scope system --audit
4026532758 net         1   827 root   /usr/sbin/irqbalance
4026532818 mnt         1   827 root   /usr/sbin/irqbalance
4026532819 mnt         1   867 chrony /usr/sbin/chronyd -F 2
4026532820 net         1   832 rtkit  /usr/libexec/rtkit-daemon
4026532857 mnt         1   831 root   /usr/libexec/power-profiles-daemon
4026532881 mnt         1   832 rtkit  /usr/libexec/rtkit-daemon
4026532882 mnt         1   834 root   /usr/libexec/switcheroo-control
4026532883 mnt         1   835 root   /usr/lib/systemd/systemd-logind
4026532884 mnt         1   837 root   /usr/libexec/upowerd
4026532885 uts         1   827 root   /usr/sbin/irqbalance
4026532886 user        1   827 root   /usr/sbin/irqbalance
4026532887 uts         1   867 chrony /usr/sbin/chronyd -F 2
4026532888 uts         1   831 root   /usr/libexec/power-profiles-daemon
4026532889 user        1   837 root   /usr/libexec/upowerd
4026532890 uts         1   835 root   /usr/lib/systemd/systemd-logind
4026532891 mnt         1   884 root   /usr/sbin/ModemManager
4026532892 mnt         1   952 root   /usr/sbin/NetworkManager --no-daemon
4026532893 net         2  6295 root   bash
4026533056 mnt         1  5921 root   /usr/libexec/fwupd/fwupd
4026533057 mnt         1  1150 root   /usr/sbin/rsyslogd -n
4026533188 mnt         1  1812 colord /usr/libexec/colord

```
- et on peut voir avec un `ls -al` dans `/proc` que ce nouveau terminal est dans un autre *namespace* `network`
```console
[root@Host-006 jeanc]# ls -al /proc/self/ns/net
lrwxrwxrwx. 1 root root 0  9 mars  11:19 /proc/self/ns/net -> 'net:[4026532893]'
[root@Host-006 jeanc]# ls -al /proc/1/ns/net
lrwxrwxrwx. 1 root root 0  9 mars  11:14 /proc/1/ns/net -> 'net:[4026531840]'
```
C'est nickel ça fonctionne bien

### B. pid

🌞 **Créer un nouveau *namespace* `pid`**

- avec une commande `unshare`
- lancez un `bash` à l'intérieur
```console
[jeanc@Host-006 ~]$ sudo unshare --pid --mount-proc --fork bash
[root@Host-006 jeanc]# 
```
> Attention, pour avoir un process qui ne voit pas **du tout** les autres processus, il faut lui créer un namespace PID, mais il faut aussi lui donner un `/proc` qui est différent de la machine hôte. Avec la commande `unshare`, je vous recommande donc : `unshare --pid --mount-proc --fork` pour complètement isoler un processus dans un namespace PID, dans lequel il ne verra que lui-même.

🌞 **Prouvez que votre nouveau *namespace* est bien là**

- déjà un `ps -ef` devrait pas montrer grand chose
```console
root@Host-006 jeanc]# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 11:20 pts/0    00:00:00 bash
root          24       1  0 11:22 pts/0    00:00:00 ps -ef

```
- et on peut `lsns` pour voir ce nouveau *namespace*
```console
root@Host-006 jeanc]# lsns
        NS TYPE   NPROCS PID USER COMMAND
4026531834 time        2   1 root bash
4026531835 cgroup      2   1 root bash
4026531837 user        2   1 root bash
4026531838 uts         2   1 root bash
4026531839 ipc         2   1 root bash
4026531840 net         2   1 root bash
4026532893 mnt         2   1 root bash
4026532894 pid         2   1 root bash
```
- et on peut voir avec un `ls -al` dans `/proc` que ce nouveau terminal est dans un autre *namespace* `pid`
```console
[root@Host-006 jeanc]# ls -al /proc/self/ns/pid
lrwxrwxrwx. 1 root root 0  9 mars  11:23 /proc/self/ns/pid -> 'pid:[4026532894]'
```
## 3. AND MY CONTAINERS

Oèoè les conteneurs les conteneurs, ils arrivent ils arrivent.

Un outil comme Docker repose **complètement** sur les *namespaces* et les *CGroups* pour lancer des processus de façon isolée sur la machine.

> Maintenant vous comprenez pourquoi j'me vénère quand on me dit que c'est des ptites VMs (genre non, mais alors pas du tout). Aussi pourquoi le conteneur est "isolé" du reste du système. Aussiiiiiiii pourquoi Docker est une techno fondamentalement Linux : Linux *namespaces* + Linux *CGroups* bois&gyals.

### A. Quick install

🌞 **Installer Docker sur la machine**

- suivez les instructions de la doc officielle spécifique pour notre OS !
- une fois installé, ajoutez votre user au groupe docker : `sudo usermod -aG docker $(whoami)`
- et démarrez le service docker ! `sudo systemctl enable --now docker`
```console
[jeanc@Host-006 ~]$ sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
[sudo] Mot de passe de jeanc : 
Ajout du dépôt depuis : https://download.docker.com/linux/rhel/docker-ce.repo
[jeanc@Host-006 ~]$ sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
Docker CE Stable - x86_64                                                                                                                                                            56 kB/s | 3.5 kB     00:00    
Le paquet docker-ce-3:28.0.1-1.el9.x86_64 est déjà installé.
Le paquet docker-ce-cli-1:28.0.1-1.el9.x86_64 est déjà installé.
Le paquet containerd.io-1.7.25-3.1.el9.x86_64 est déjà installé.
Le paquet docker-compose-plugin-2.33.1-1.el9.x86_64 est déjà installé.
Dépendances résolues.
Rien à faire.
Terminé !
[jeanc@Host-006 ~]$ sudo systemctl --now enable docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
[jeanc@Host-006 ~]$ sudo usermod -aG docker $(whoami)
[jeanc@Host-006 ~]$ sudo systemctl enable --now docker

```
### B. A simple container

🌞 **Lancer un simple conteneur qui sleep**

- utilisez la commande suivante :

```bash
docker run -d debian sleep 9999
```

🌞 **Avez les commandes de votre choix, avec le plus de détails possible, prouvez-que :**

- ce processus `sleep` est bien lancé sur votre machine, par votre utilisateur courant
```console
jeanc@Host-006 ~]$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED         STATUS         PORTS     NAMES
6c3d45d70082   debian    "sleep 9999"   2 minutes ago   Up 2 minutes             silly_rhodes
```
- ce processus `sleep` est isolé à l'aide de *namespaces*
- un *CGroup* a été attribué à ce conteneur
- tout nouveau processus "dans" le conteneur est lui aussi isolé (voir ci-dessous pour lancer d'autres process dans le conteneur)
```console
root@6c3d45d70082:/# apt update -y
apt install -y procps iproute2 iputils-ping
```
```bash
# pour obtenir un shell dans un conteneur existant
docker exec -it <conteneur> bash
# par exemple, si le conteneur est nommé "toto"
docker exec -it toto bash

# depuis là, vous pouvez lancez des nouveaux programmes
apt update -y
## procps contient la commande ps
## iproute2 contient la commande ip
## iputils-ping contient la commande ping
apt install -y procps iproute2 iputils-ping
```
```console
root@6c3d45d70082:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 0e:f8:2b:f3:be:cf brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

```
### C. CGroup

Et les CGroups dans tout ça ?

🌞 **Lancer un conteneur restreint**

- avec des options du `docker run`
- limiter l'accès RAM de ce conteneur à 456M
```console
jeanc@Host-006 ~]$ sudo docker run -d --memory="456m" debian sleep 9999
d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f
```

🌞 **CGroup ?**

- prouvez que cette limite a été mise en place avec une conf CGroup
- un *CGroup* est automatiquement créé à chaque fois que vous lancez un conteneur (un `scope` systemd) !
```console
[jeanc@Host-006 ~]$ sudo docker inspect --format='{{.HostConfig.Memory}}' d83b1f9c4d1d
478150656
```
478150656 -> Correspond bien à 456Mo *1024*1024
Et pour les scopes systemd : 
```console
[jeanc@Host-006 ~]$ sudo systemctl status 'docker-*.scope'
Warning: The unit file, source configuration file or drop-ins of docker-6c3d45d70082ea9136385858951cabd053fac4020b0ff61ce6aac2195ffaf2b9.scope changed on disk. Run 'systemctl daemon-reload' to reload units.
Warning: The unit file, source configuration file or drop-ins of docker-d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f.scope changed on disk. Run 'systemctl daemon-reload' to reload units.
● docker-6c3d45d70082ea9136385858951cabd053fac4020b0ff61ce6aac2195ffaf2b9.scope - libcontainer container 6c3d45d70082ea9136385858951cabd053fac4020b0ff61ce6aac2195ffaf2b9
     Loaded: loaded (/run/systemd/transient/docker-6c3d45d70082ea9136385858951cabd053fac4020b0ff61ce6aac2195ffaf2b9.scope; transient)
  Transient: yes
    Drop-In: /run/systemd/transient/docker-6c3d45d70082ea9136385858951cabd053fac4020b0ff61ce6aac2195ffaf2b9.scope.d
             └─50-DevicePolicy.conf, 50-DeviceAllow.conf
     Active: active (running) since Sun 2025-03-09 11:30:57 CET; 12min ago
         IO: 3.7M read, 55.2M written
      Tasks: 1 (limit: 10741)
     Memory: 44.1M
        CPU: 3.576s
     CGroup: /system.slice/docker-6c3d45d70082ea9136385858951cabd053fac4020b0ff61ce6aac2195ffaf2b9.scope
             └─7134 sleep 9999

mars 09 11:30:57 Host-006 systemd[1]: Started libcontainer container 6c3d45d70082ea9136385858951cabd053fac4020b0ff61ce6aac2195ffaf2b9.

● docker-d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f.scope - libcontainer container d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f
     Loaded: loaded (/run/systemd/transient/docker-d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f.scope; transient)
  Transient: yes
    Drop-In: /run/systemd/transient/docker-d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f.scope.d
             └─50-DevicePolicy.conf, 50-DeviceAllow.conf, 50-MemoryMax.conf, 50-MemorySwapMax.conf
     Active: active (running) since Sun 2025-03-09 11:41:03 CET; 2min 30s ago
         IO: 16.0K read, 0B written
      Tasks: 1 (limit: 10741)
     Memory: 928.0K (max: 456.0M swap max: 456.0M available: 455.0M)
        CPU: 23ms
     CGroup: /system.slice/docker-d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f.scope
             └─7870 sleep 9999

mars 09 11:41:03 Host-006 systemd[1]: Started libcontainer container d83b1f9c4d1d7866c935228070cf839005f0fd0b3a5f5f603afe63fad3f32f8f.

```
![not afraid](./img/nowask.png)
