
# Part III : CGroup

➜ **Les *CGroup* c'est un mécanisme du noyau Linux pour faire des groupes de processus et restreindre leur accès aux ressources de la machine.**

On parle ici notamment de restreindre l'accès à :

- la mémoire (par exemple : on définit une quantité de mémoire maximum que le groupe de processus aura le droit d'utiliser)
- le CPU (par exemple : on définit un poids pour qu'un groupe de processus soit prioritaire)
- écriture/lecture (par exemple : limitation des écritures sur le disque)
- d'autres trucs, c'pour vous donner une idée

➜ **Les *CGroup* permettent aussi de monitorer en temps réel l'accès aux ressources d'un groupe de processus.**

Vous pouvez constater ça avec les commandes `systemd-cgls` et `systemd-cgtop` notamment.

## Sommaire

- [Part III : CGroup]
  - [Sommaire]
  - [1. Explore]
  - [2. Do it]
  - [3. systemd]
    - [A. One-shot]
    - [B. Real service]

## 1. Explore

Pour rappel : la configuration actuelle des *CGroups* est dispo dans `/sys/fs/cgroup`.

🌞 **Afficher...** :

- la liste des controllers *CGroups* dispos sur le système
```console
[jeanc@efrei-xmg4agau1 ~]$ ls /sys/fs/cgroup/
cgroup.controllers      cgroup.threads         dev-mqueue.mount  misc.capacity                  system.slice
cgroup.max.depth        cpuset.cpus.effective  init.scope        misc.current                   user.slice
cgroup.max.descendants  cpuset.cpus.isolated   io.stat           sys-fs-fuse-connections.mount
cgroup.procs            cpuset.mems.effective  memory.numa_stat  sys-kernel-config.mount
cgroup.stat             cpu.stat               memory.reclaim    sys-kernel-debug.mount
cgroup.subtree_control  dev-hugepages.mount    memory.stat       sys-kernel-tracing.mount
```
- la quantité de mémoire max que vous êtes autorisés à utiliser dans votre session utilisateur
  - par défaut, sous Rocky, le *controller* memory n'est pas activé : normal si vous ne voyez aucun fichier `memory.max`
  - si c'est le cas, ça veut dire qu'aucune restriction RAM est en place (vous devez justement constater ça)
  On constate bien qu'aucun fichier memory.max est apparent, donc pas de réstriction.
- les noms de tous les *CGroups* créés
  - ce sont tous les sous-dossiers de `/sys/fs/cgroup`
  - il devrait y avoir au moins les slices et scopes de systemd, on en parle plus bas
 ```console
[jeanc@efrei-xmg4agau1 ~]$ ls -R /sys/fs/cgroup/
/sys/fs/cgroup/:
cgroup.controllers      cgroup.threads         dev-mqueue.mount  misc.capacity                  system.slice
cgroup.max.depth        cpuset.cpus.effective  init.scope        misc.current                   user.slice
cgroup.max.descendants  cpuset.cpus.isolated   io.stat           sys-fs-fuse-connections.mount
cgroup.procs            cpuset.mems.effective  memory.numa_stat  sys-kernel-config.mount
cgroup.stat             cpu.stat               memory.reclaim    sys-kernel-debug.mount
cgroup.subtree_control  dev-hugepages.mount    memory.stat       sys-kernel-tracing.mount

/sys/fs/cgroup/dev-hugepages.mount:
cgroup.controllers      cgroup.stat             memory.events.local  memory.peak          memory.swap.peak
cgroup.events           cgroup.subtree_control  memory.high          memory.reclaim       memory.zswap.current
cgroup.freeze           cgroup.threads          memory.low           memory.stat          memory.zswap.max
cgroup.kill             cgroup.type             memory.max           memory.swap.current  pids.current
cgroup.max.depth        cpu.stat                memory.min           memory.swap.events   pids.events
cgroup.max.descendants  memory.current          memory.numa_stat     memory.swap.high     pids.max
cgroup.procs            memory.events           memory.oom.group     memory.swap.max      pids.peak
```

## 2. Do it

🌞 **Créer un nouveau *CGroup*** :

- appelez-le `meow`
- activez les controllers `cpu` `cpuset` et `memory` s'ils ne le sont pas déjà
```console
[jeanc@efrei-xmg4agau1 ~]$ echo "+memory" > cgroup.subtree_control
[jeanc@efrei-xmg4agau1 ~]$ echo "+memory" | sudo tee  cgroup.subtree_control
[jeanc@efrei-xmg4agau1 ~]$ sudo mkdir meow
[jeanc@efrei-xmg4agau1 ~]$ cd meow/
[jeanc@efrei-xmg4agau1 ~]$ echo "+memory" | sudo tee  cgroup.subtree_control
```
> Vous devez donc créer le dossier `/sys/fs/cgroup/meow/` avec un simple `mkdir`, puis interagir avec les fichiers qui s'y trouvent (le dossier a été automatiquement populé).

🌞 **Créer un nouveau sous-CGroup** :

- appelez-le `task1`
```console
[jeanc@efrei-xmg4agau1 ~]$ sudo mkdir task1
[jeanc@efrei-xmg4agau1 ~]$ cd task1
[jeanc@efrei-xmg4agau1 ~]$ cat cgroup.controllers
[jeanc@efrei-xmg4agau1 ~]$ echo "150M" | sudo tee memory.max
[jeanc@efrei-xmg4agau1 ~]$ echo $$  | sudo tee cgroup.procs
[jeanc@efrei-xmg4agau1 ~]$ cat cgroup.procs
```
- on parle de créer le dossier `/sys/fs/cgroup/meow/task1/`
```console
[root@efrei-xmg4agau1 jeanc]# sudo mkdir /sys/fs/cgroup/meow/task1
```
- prouvez que les controllers activés sur `meow` ont bien été hérités
```console
[root@efrei-xmg4agau1 jeanc]# cat /sys/fs/cgroup/meow/task1/cgroup.controllers
cpuset cpu memory
```

🌞 **Mettez en place une limitation RAM**

- définissez une limite de 150M de RAM pour ce CGroup `task1`
```console
[jeanc@efrei-xmg4agau1 ~]$ echo "150M" | sudo tee memory.max
[jeanc@efrei-xmg4agau1 ~]$ echo $$  | sudo tee cgroup.procs
[jeanc@efrei-xmg4agau1 ~]$ cat cgroup.procs
```

🌞 **Prouvez que la limite est effective**

1. utilisez la commande `stress-ng` pour remplir la mémoire de la machine
2. constatez que la RAM est pleine
3. ajoutez votre shell `bash` actuel au *CGroup* `task1`
4. utilisez de nouveau `stress-ng`
5. constatez que le processus `stress-ng` est tué en boucle dès qu'il remplit la RAM au delà de la limite

On lance d'un coté le stress : 
```console
[jeanc@efrei-xmg4agau1 ~]$ stress-ng --vm 1 --vm-bytes 1G
```
Ensuite, on va monitoré l'état de le RAM, et voir si la limitation fonctionne. Ici, la RAM se régule.
```console
Every 0.1s: ps -eo cmd,pid,rss | grep stress                                            efrei-xmg4agau1.etudiants.campus.villejuif: Thu Feb 27 10:48:12 2025

stress-ng --vm 1 --vm-bytes    7986  4864
stress-ng --vm 1 --vm-bytes    7988  1852
stress-ng --vm 1 --vm-bytes    7989 151756
grep stress                    8195  2176
```

> On rappelle que tout processus lancé par un processus existant se retrouvera par défaut dans le même *CGroup* que son parent. C'est pour ça que vous ajoutez votre shell `bash` au *CGroup* : tout ce que vous exécuterez depuis ce `bash` sera exécuté dans le même *CGroup* que lui. Ha et le truc qui tue votre processus quand il prendre trop de RAM, c'est le [**OOM-killer**](https://en.wikipedia.org/wiki/Out_of_memory).

![OOM killer](./img/oom_killer.png)

🌞 **Créer un nouveau sous-*CGroup*** :

- appelez-le `task2`
```console
mkdir /sys/fs/cgroup/meow/task2
```

🌞 **Appliquer des restrictions CPU** :

- utilisez la mécanique de `cpu.weight` pour définir des priorités différentes à `task1` et `task2`
```console
[jeanc@efrei-xmg4agau1 task1]$ echo 100 | sudo tee /sys/fs/cgroup/meow/task1/cpu.weight
[jeanc@efrei-xmg4agau1 task1]$ echo 1000 | sudo tee /sys/fs/cgroup/meow/task2/cpu.weight
[jeanc@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/meow/task1/cpu.weight
100
[jeanc@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/meow/task2/cpu.weight
1000
```
- utilisez `stress-ng` ou un bon vieux `cat /dev/random` pour lancer un processus CPU-intensive, et pour prouver que la restriction est en place
- pour tester, vous devez :
  - lancer deux shells en même temps
  - ajouter le premier au *CGroup* `task1`
  - ajouter le deuxième au *CGroup* `task2`
Shell 1:
 ```console
[jeanc@efrei-xmg4agau1 task1]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task1/cgroup.procs
[jeanc@efrei-xmg4agau1 task1]$ sudo stress-ng --timeout 55555 --vm 4 -c 4 -l 100 --cpu-method=div16
```
Shell 2:
```console
[jeanc@efrei-xmg4agau1 task2]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task2/cgroup.procs
[jeanc@efrei-xmg4agau1 task2]$ sudo stress-ng --timeout 55555 --vm 4 -c 4 -l 100 --cpu-method=div16
```
  - dans les deux shells, lancer un processus CPU-intensive
  - constatez avec un `htop` par exemple que les deux processus ne se répartissent pas équitablement la puissance du CPU
 Resultats : 
```console
[jeanc@efrei-xmg4agau1 ~]$ watch -n0.1 "ps -eo cmd,pid,%cpu | grep stress"
sudo stress-ng --timeout 55   64555  0.0
stress-ng --timeout 55555 -   64557  0.0
stress-ng --timeout 55555 -   64558  0.0
stress-ng --timeout 55555 -   64559  0.0
stress-ng --timeout 55555 -   64560  0.0
stress-ng --timeout 55555 -   64561  0.0
stress-ng --timeout 55555 -   64562 30.6
stress-ng --timeout 55555 -   64563 31.1
stress-ng --timeout 55555 -   64564 31.0
stress-ng --timeout 55555 -   64565 31.0
stress-ng --timeout 55555 -   64566 30.6
stress-ng --timeout 55555 -   64567 30.9
stress-ng --timeout 55555 -   64568 31.3
stress-ng --timeout 55555 -   64569 31.3
sudo stress-ng --vm 4 --tim   64570  0.0
stress-ng --vm 4 --timeout    64572  0.0
stress-ng --vm 4 --timeout    64573  0.0
stress-ng --vm 4 --timeout    64574  0.0
stress-ng --vm 4 --timeout    64575  0.0
stress-ng --vm 4 --timeout    64576  0.0
stress-ng --vm 4 --timeout    64577 24.0
stress-ng --vm 4 --timeout    64578 23.9
stress-ng --vm 4 --timeout    64579 24.0
stress-ng --vm 4 --timeout    64580 23.5
stress-ng --vm 4 --timeout    64581 23.9
stress-ng --vm 4 --timeout    64582 23.9
stress-ng --vm 4 --timeout    64583 24.4
stress-ng --vm 4 --timeout    64584 24.3
grep stress                   65896  0.0
```

## 3. systemd

Par défaut, sous Linux, y'a systemd (ouais encore lui). Il utilise pas mal les *CGroup* nativement. Il fait naturellement deux trucs :

- **il met les `service` dans des `slice`s**
  - créer un `slice` systemd, c'est juste créer un cgroup (dont le nom se terminera par `.slice`)
  - tous les `service`s sont forcément dans un `slice`
- **il met des programmes lancés à l'extérieur (genre des trucs qui sont pas des `services`) dans des `scope`**
  - c'est pareil : un `scope` c'est juste un CGroup créé automatiquement par systemd, dont le nom se termine par `.scope`
  - ça permet à systemd de gérer certains processus même si c'est pas lui qui les lance
  - par exemple quand t'ouvres une session SSH n_n

Bref, l'un des rôles principaux de systemd c'est lancer et gérer des processus (services). Il est donc tout naturel qu'il utilise les *CGroups* Linux pour mener à bien ce job.

### A. One-shot

Y'a une commande rigolote et parfois pratique qui permet de jouer avec tout ça. Une commande qui permet de créer un service système temporaire à la volée en une seule ligne de commande : `systemd-run`.

> Pour plusieurs tests dans le TP, on se servira du serveur Web embarqué par Python. Il se lance en une seule commande, fait des trucs sur le système (serveur web, donc il lit des fichiers, fait des connexions réseau), c'est parfait pour faire des tests ! Pour le lancer : `python -m http.server 8888`.

🌞 **Lancer un serveur Web Python sous forme de service temporaire**

- avec la commande `python -m http.server <PORT>`
- n'oubliez pas d'ouvrir ce port dans le firewall pour tester
- il faudra le lancer avec la commande `systemd-run` pour en faire un service temporaire
- affichez le `status` du service pour prouver qu'il run
```console
[jeanc@efrei-xmg4agau1 task1]$ sudo systemd-run -u my_http_service --description="Python HTTP Server" python3 -m http.server 8888
[sudo] password for jeanc:
Running as unit: my_http_service.service
[jeanc@efrei-xmg4agau1 task1]$ systemctl status my_http_service
● my_http_service.service - Python HTTP Server
     Loaded: loaded (/run/systemd/transient/my_http_service.service; transient)
  Transient: yes
     Active: active (running) since Thu 2025-02-27 13:12:55 CET; 3s ago
   Main PID: 67805 (python3)
      Tasks: 1 (limit: 23148)
     Memory: 9.4M
        CPU: 53ms
     CGroup: /system.slice/my_http_service.service
             └─67805 /bin/python3 -m http.server 8888

Feb 27 13:12:55 efrei-xmg4agau1.campus.villejuif systemd[1]: Started Python HTTP Server.
[jeanc@efrei-xmg4agau1 task1]$ sudo firewall-cmd --add-port=8888/tcp --permanent
sudo firewall-cmd --reload
success
success
```

```bash

# lancer un sleep sous la forme d'un service nommé meow_test.service
sudo systemd-run -u meow_test sleep 9999
```

🌞 **Appliquer à la volée des restrictions**

- avec l'option `-p` de `systemd-run` vous pouvez préciser des paramètres pour le service
- utilisez le paramètre `MemoryMax` pour mettre en place une limite à 234M
Configuration :
```console
[jeanc@efrei-xmg4agau1 task1] sudo systemd-run -u meow_test -p MemoryMax=234M sleep 9999
[jeanc@efrei-xmg4agau1 task1]$ systemctl status meow_test
● meow_test.service - /bin/sleep 9999
     Loaded: loaded (/run/systemd/transient/meow_test.service; transient)
  Transient: yes
     Active: active (running) since Thu 2025-02-27 13:19:52 CET; 4min 18s ago
   Main PID: 68021 (sleep)
      Tasks: 1 (limit: 23148)
     Memory: 192.0K (max: 234.0M available: 233.8M)
        CPU: 1ms
     CGroup: /system.slice/meow_test.service
             └─68021 /bin/sleep 9999

Feb 27 13:19:52 efrei-xmg4agau1.campus.villejuif systemd[1]: Started /bin/sleep 9999.
```
```console
[jeanc@efrei-xmg4agau1 task1]$ sudo grep -nri $(( 234 * 1024 * 1024 )) /sys/fs/cgroup/
/sys/fs/cgroup/system.slice/meow_test.service/memory.max:1:245366784
```
> En vrai, `systemd-run` est un tool vraiment pratique pour limiter l'accès aux ressources d'un process qu'on lance oneshot.

🌞 **Restrictions *CGroup* ?**

- prouvez que la restriction à 234M de `systemd-run` est mise en place avec les *CGroups* Linux

> Les montants de RAM chelous c'pour vous permettre de faire des `grep` pour trouver facilement. Attention par contre, les restrictions automatiques appliquées par systemd sont exprimées en octets (pas KB ni MB) donc faut faire une ptite multiplication pour le trouver facilement. En l'occurence, un `grep -nri $(( 234 * 1024 * 1024 ))` depuis `/sys/fs/cgroup` fera le taff ;)

### B. Real service

🌞 **Créez un service `web.service`**

- habitués nan ? Faut créer un fichier dans `/etc/systemd/system/`
- n'oubliez pas de `sudo systemctl daemon-reload` à chaque modification
- ce service doit lancer un serveur web python sur le port 9999/tcp

🌞 **Restriction *CGroup***

- modifier le fichier `web.service` pour inclure des limitations mises en place par des CGroups :
  - mémoire max : 321M
  - limitation d'écriture disque : 1M
  - limitation de lecture disque : 1M
  - limitation CPU : 50% d'utilisation
 ```console
sudo nano /etc/systemd/system/web.service
[Unit]
Description=Python HTTP Server on port 9999
After=network.target

[Service]
ExecStart=/usr/bin/python3 -m http.server 9999
Restart=always
User=nobody
Group=nogroup
WorkingDirectory=/tmp

# Limitations via CGroup
MemoryMax=321M
IOWriteBandwidthMax=/ 1M
IOReadBandwidthMax=/ 1M
CPUQuota=50%

[Install]
WantedBy=multi-user.target

```
```console
sudo systemctl daemon-reload
sudo systemctl enable web.service
sudo systemctl start web.service
systemctl status web.service
[jeanc@efrei-xmg4agau1 task1]$ systemctl status web.service
● web.service - Python HTTP Server on port 9999
     Loaded: loaded (/etc/systemd/system/web.service; enabled; preset: disabled)
     Active: active (running) since Thu 2025-02-27 13:28:07 CET; 4min 28s ago
   Main PID: 68187 (python3)
      Tasks: 1 (limit: 23148)
     Memory: 9.2M (max: 321.0M available: 311.7M)
        CPU: 74ms
     CGroup: /system.slice/web.service
             └─68187 /usr/bin/python3 -m http.server 9999

Feb 27 13:28:07 efrei-xmg4agau1.campus.villejuif systemd[1]: Started Python HTTP Server on port 9999.
Feb 27 13:29:04 efrei-xmg4agau1.campus.villejuif systemd[1]: [🡕] /etc/systemd/system/web.service:8: Special user nobody configured, this is not safe!
```
🌞 **Prouvez que ces restrictions ont été mises en place avec les *CGroups***

- en explorant le dossier `/sys/` toujours !
Vérifications : 
```console
[jeanc@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/system.slice/web.service/memory.max
336592896
[jeanc@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/system.slice/web.service/cpu.max
50000 100000
[jeanc@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/system.slice/web.service/io.max
8:0 rbps=1000000 wbps=1000000 riops=max wiops=max
```
