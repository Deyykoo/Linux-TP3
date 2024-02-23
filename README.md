# Linux-TP3

## I. Service SSH

### 1. Analyse du service

```
🌞 S'assurer que le service sshd est démarré

[quentin@node1 ~]$ systemctl status
● node1.tp2.b1
    State: running
    Units: 284 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Mon 2024-01-29 10:46:59 CET; 42min ago
  systemd: 252-13.el9_2
   CGroup: /
           ├─init.scope
           │ └─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 31
           ├─system.slice
           │ ├─NetworkManager.service
           │ │ └─1534 /usr/sbin/NetworkManager --no-daemon
           │ ├─auditd.service
           │ │ └─655 /sbin/auditd
           │ ├─chronyd.service
           │ │ └─688 /usr/sbin/chronyd -F 2
           │ ├─crond.service
           │ │ ├─ 709 /usr/sbin/crond -n
           │ │ └─1511 /usr/sbin/anacron -s
           │ ├─dbus-broker.service
           │ │ ├─690 /usr/bin/dbus-broker-launch --scope system --audit
           │ │ └─694 dbus-broker --log 4 --controller 9 --machine-id 4d53ec6adb904256af5de5ea61d2fb5d --max-bytes 53687>
           │ ├─firewalld.service
           │ │ └─680 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid
           │ ├─rsyslog.service
           │ │ └─681 /usr/sbin/rsyslogd -n
           │ ├─sshd.service
           │ │ └─702 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
           │ ├─systemd-journald.service
           │ │ └─563 /usr/lib/systemd/systemd-journald
           │ ├─systemd-logind.service
           │ │ └─682 /usr/lib/systemd/systemd-logind
           │ └─systemd-udevd.service
           │   └─udev
           │     └─576 /usr/lib/systemd/systemd-udevd
           └─user.slice
             └─user-1000.slice
               ├─session-1.scope
               │ ├─ 710 "login -- quentin"
               │ └─1314 -bash
               ├─session-3.scope
               │ ├─1399 "sshd: quentin [priv]"
               │ ├─1403 "sshd: quentin@pts/0"
               │ └─1404 -bash
               ├─session-5.scope
               │ ├─1596 "sshd: quentin [priv]"
               │ ├─1600 "sshd: quentin@pts/1"
               │ ├─1601 -bash
               │ ├─1691 systemctl status
               │ └─1692 less
               └─user@1000.service
                 └─init.scope
                   ├─1305 /usr/lib/systemd/systemd --user
                   └─1307 "(sd-pam)"
lines 27-55/55 (END)


🌞 Analyser les processus liés au service SSH

[quentin@node1 ~]$ ps -ef | grep ssh
root         702       1  0 10:47 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1399     702  0 10:54 ?        00:00:00 sshd: quentin [priv]
quentin     1403    1399  0 10:54 ?        00:00:00 sshd: quentin@pts/0
root        1596     702  0 11:05 ?        00:00:00 sshd: quentin [priv]
quentin     1600    1596  0 11:05 ?        00:00:00 sshd: quentin@pts/1
quentin     1699    1601  0 11:34 pts/1    00:00:00 grep --color=auto ssh


🌞 Déterminer le port sur lequel écoute le service SSH

[quentin@node1 ~]$ sudo ss -alnpt
[sudo] password for quentin:
State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN     0          128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=702,fd=3))
LISTEN     0          128                     [::]:22                   [::]:*        users:(("sshd",pid=702,fd=4))


🌞 Consulter les logs du service SSH

- commande = journalctl -xe -u sshd

- [quentin@node1 /]$ tail -n 10 /var/log/dnf.log
2024-01-29T11:17:38+0100 DEBUG reviving: failed for 'extras', mismatched repomd.
2024-01-29T11:17:38+0100 DEBUG repo: downloading from remote: extras
2024-01-29T11:17:38+0100 DEBUG countme: event triggered for extras: bucket 1
2024-01-29T11:17:53+0100 DEBUG extras: using metadata from Sun 31 Dec 2023 02:20:21 AM CET.
2024-01-29T11:17:53+0100 DEBUG User-Agent: constructed: 'libdnf (Rocky Linux 9.2; generic; Linux.x86_64)'
2024-01-29T11:17:53+0100 DDEBUG timer: sack setup: 79591 ms
2024-01-29T11:17:53+0100 DEBUG Completion plugin: Generating completion cache...
2024-01-29T11:17:54+0100 INFO Metadata cache created.
2024-01-29T11:17:54+0100 DDEBUG Cleaning up.
2024-01-29T11:17:54+0100 DDEBUG Plugins were unloaded.
```

### 2. Modification du service

```	
🌞 Identifier le fichier de configuration du serveur SSH

  - /etc/ssh/sshd_config


🌞 Modifier le fichier de conf

- echo $RANDOM : 16894

- commande : sudo nano /etc/ssh/sshd_config
# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
Port 16894
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

- [quentin@node1 ssh]$ sudo cat sshd_config | grep 16894
Port 16894

- commande : [quentin@node1 /]$ sudo firewall-cmd --list-all | grep port 
  ports: 16894/tcp
  forward-ports:
  source-ports:


🌞 Redémarrer le service

  - [quentin@node1 ~]$ sudo systemctl restart sshd


🌞 Effectuer une connexion SSH sur le nouveau port

- PS C:\Users\qchai> ssh quentin@10.2.1.11 -p 16894
quentin@10.2.1.11's password:
Last login: Tue Jan 30 10:13:30 2024 from 10.2.1.10
```

## II. Service HTTP

### 1. Mise en place

```
🌞 Installer le serveur NGINX

- [quentin@node1 log]$ sudo dnf install nginx
[sudo] password for quentin:
Last metadata expiration check: 0:03:58 ago on Tue 30 Jan 2024 11:03:23 AM CET.
Dependencies resolved.
========================================================================================================================
 Package                         Architecture         Version                             Repository               Size
========================================================================================================================
Installing:
 nginx                           x86_64               1:1.20.1-14.el9_2.1                 appstream                36 k
Installing dependencies:
 nginx-core                      x86_64               1:1.20.1-14.el9_2.1                 appstream               565 k
 nginx-filesystem                noarch               1:1.20.1-14.el9_2.1                 appstream               8.5 k
 rocky-logos-httpd               noarch               90.14-2.el9                         appstream                24 k

Transaction Summary
========================================================================================================================
Install  4 Packages

Total download size: 634 k
Installed size: 1.8 M
Is this ok [y/N]: y
Downloading Packages:
(1/4): nginx-filesystem-1.20.1-14.el9_2.1.noarch.rpm                                    1.7 kB/s | 8.5 kB     00:05
(2/4): nginx-1.20.1-14.el9_2.1.x86_64.rpm                                               7.0 kB/s |  36 kB     00:05
(3/4): rocky-logos-httpd-90.14-2.el9.noarch.rpm                                         4.7 kB/s |  24 kB     00:05
(4/4): nginx-core-1.20.1-14.el9_2.1.x86_64.rpm                                          4.3 MB/s | 565 kB     00:00
------------------------------------------------------------------------------------------------------------------------
Total                                                                                    61 kB/s | 634 kB     00:10
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                1/1
  Running scriptlet: nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                                                    1/4
  Installing       : nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                                                    1/4
  Installing       : nginx-core-1:1.20.1-14.el9_2.1.x86_64                                                          2/4
  Installing       : rocky-logos-httpd-90.14-2.el9.noarch                                                           3/4
  Installing       : nginx-1:1.20.1-14.el9_2.1.x86_64                                                               4/4
  Running scriptlet: nginx-1:1.20.1-14.el9_2.1.x86_64                                                               4/4
  Verifying        : rocky-logos-httpd-90.14-2.el9.noarch                                                           1/4
  Verifying        : nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                                                    2/4
  Verifying        : nginx-1:1.20.1-14.el9_2.1.x86_64                                                               3/4
  Verifying        : nginx-core-1:1.20.1-14.el9_2.1.x86_64                                                          4/4

Installed:
  nginx-1:1.20.1-14.el9_2.1.x86_64                              nginx-core-1:1.20.1-14.el9_2.1.x86_64
  nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                   rocky-logos-httpd-90.14-2.el9.noarch

Complete!


🌞 Démarrer le service NGINX

[quentin@node1 log]$ sudo systemctl start nginx
[sudo] password for quentin:


🌞 Déterminer sur quel port tourne NGINX

[quentin@node1 log]$ sudo ss -alnpt | grep nginx
LISTEN 0      511          0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=11467,fd=6),("nginx",pid=11466,fd=6))
LISTEN 0      511             [::]:80            [::]:*    users:(("nginx",pid=11467,fd=7),("nginx",pid=11466,fd=7))


🌞 Déterminer les processus liés au service NGINX

[quentin@node1 log]$ ps -ef | grep nginx
root       11466       1  0 11:23 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      11467   11466  0 11:23 ?        00:00:00 nginx: worker process
quentin    11476    1335  0 11:27 pts/0    00:00:00 grep --color=auto nginx


🌞 Déterminer le nom de l'utilisateur qui lance NGINX

[quentin@node1 log]$ cat /etc/passwd | grep nginx
nginx:x:991:991:Nginx web server:/var/lib/nginx:/sbin/nologin


🌞 Test !

- commande : [quentin@node1 ~]$ sudo firewall-cmd --list-all

- commande : [quentin@node1 ~]$ sudo firewall-cmd --add-port=80/tcp --permanent

- commande : [quentin@node1 ~]$ sudo firewall-cmd --reload

[quentin@node1 ~]$ curl 10.2.1.11 | head -n 7
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
100  7620  100  7620    0     0   620k      0 --:--:-- --:--:-- --:--:--  676k
```

### 2. Analyser la conf de NGINX

```
🌞 Déterminer le path du fichier de configuration de NGINX

- [quentin@node1 nginx]$ ls -al /etc/nginx/nginx.conf
- rw-r--r--. 1 root root 2334 Oct 16 20:00 /etc/nginx/nginx.conf


🌞 Trouver dans le fichier de conf

- [quentin@node1 ~]$ cat /etc/nginx/nginx.conf | grep server -A X
(standard input):    server {
(standard input):        server_name  _;
(standard input):        # Load configuration files for the default server block.
(standard input):# Settings for a TLS enabled server.
(standard input):#    server {
(standard input):#        server_name  _;
(standard input):#        ssl_certificate "/etc/pki/nginx/server.crt";
(standard input):#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
(standard input):#        ssl_prefer_server_ciphers on;
(standard input):#        # Load configuration files for the default server block.

- [quentin@node1 ~]$ cat /etc/nginx/nginx.conf | grep include -A X
(standard input):include /usr/share/nginx/modules/*.conf;
(standard input):    include             /etc/nginx/mime.types;
(standard input):    # See http://nginx.org/en/docs/ngx_core_module.html#include
(standard input):    include /etc/nginx/conf.d/*.conf;
(standard input):        include /etc/nginx/default.d/*.conf;
(standard input):#        include /etc/nginx/default.d/*.conf;

```

### 3. Déployer un nouveau site web

```
🌞 Créer un site web

- [quentin@node1 var]$ sudo mkdir www
  [sudo] password for quentin:

- [quentin@node1 var]$ cd www
  [quentin@node1 www]$ sudo mkdir tp3_linux
  [quentin@node1 www]$ ls
  tp3_linux

- [quentin@node1 www]$ cd tp3_linux/
  [quentin@node1 tp3_linux]$ sudo touch index.html
  [quentin@node1 tp3_linux]$ ls
  index.html

- [quentin@node1 tp3_linux]$ sudo nano index.html
  [quentin@node1 tp3_linux]$ cat index.html
  <h1>MEOW mon premier serveur web</h1>


🌞 Gérer les permissions

- [quentin@node1 tp3_linux]$ sudo chmod u+w index.html

- [quentin@node1 tp3_linux]$ ls -l
  total 4
- rw-r--r--. 1 root root 39 Feb 14 16:05 index.html

🌞 Adapter la conf NGINX

- [quentin@node1 nginx]$ sudo touch nginx.conf1

- [quentin@node1 nginx]$
      
      server {

      listen 19999;

      root/var/www/tp3_linux;
      }

