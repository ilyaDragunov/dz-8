					Домашнее задание №8

	Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова.
 	Файл и слово должны задаваться в /etc/sysconfig


1. Для начала создаём файл с конфигурацией для сервиса в директории /etc/sysconfig - из неё сервис будет брать необходимые переменные.

```

[root@test-systemd ~]# cd /etc/sysconfig/
[root@test-systemd sysconfig]# ls
anaconda         firewalld         man-db           rsyslog
chronyd          grub              modules          run-parts
cloud-info       ip6tables-config  network          samba
console          iptables-config   network-scripts  selinux
cpupower         irqbalance        nftables.conf    sshd
crond            kdump             qemu-ga          sshd-permitrootlogin
ebtables-config  kernel            rpcbind
[root@test-systemd sysconfig]# touch watchlog
[root@test-systemd sysconfig]# ls
anaconda         firewalld         man-db           rsyslog
chronyd          grub              modules          run-parts
cloud-info       ip6tables-config  network          samba
console          iptables-config   network-scripts  selinux
cpupower         irqbalance        nftables.conf    sshd
crond            kdump             qemu-ga          sshd-permitrootlogin
ebtables-config  kernel            rpcbind          watchlog

```

2. Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение, плюс ключевое слово ‘AHTUNG’

```

[root@test-systemd etc]# cd /etc/sysconfig/
[root@test-systemd sysconfig]# nano watchlog

 GNU nano 2.9.8 watchlog

# Configuration file for my watchlog service
# Place it to /etc/sysconfig

# File and word in that file that we will be monit
WORD="ACHTUNG"
LOG=/var/log/watchlog.log

```

3. Создадим скрипт, отправляющий лог в системный журнал

```

[root@test-systemd sysconfig]# cd /opt/
[root@test-systemd opt]# touch watchlog.sh
[root@test-systemd opt]# nano watchlog.sh
  GNU nano 2.9.8 watchlog.sh

#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

```

4. Добавим права на запуск файла

```

[root@test-systemd opt]# chmod +x /opt/watchlog.sh

```

5. Создадим юнит для сервиса

```

[root@test-systemd /]# cd /etc/systemd/system/
[root@test-systemd system]# touch wathclog.service
[root@test-systemd system]# nano watchlog.service

[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

[root@test-systemd system]# touch watchlog.timer
[root@test-systemd system]# nano watchlog.timer

[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target

```

6. Стартуем timer, проверяем что статус активен, есть запись в логах

```

[root@test-systemd ~]# systemctl start watchlog.timer
[root@test-systemd ~]# systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
   Loaded: loaded (/etc/systemd/system/watchlog.timer; disabled; vendor preset:>
   Active: active (elapsed) since Wed 2023-05-31 20:49:00 UTC; 20s ago
  Trigger: n/a

May 31 20:49:00 test-systemd systemd[1]: Started Run watchlog script every 30 s>
lines 1-6/6 (END)...skipping...
● watchlog.timer - Run watchlog script every 30 second
   Loaded: loaded (/etc/systemd/system/watchlog.timer; disabled; vendor preset: disabled)
   Active: active (elapsed) since Wed 2023-05-31 20:49:00 UTC; 20s ago
  Trigger: n/a

May 31 20:49:00 test-systemd systemd[1]: Started Run watchlog script every 30 second.

[root@test-systemd ~]# tail -f /var/log/messages
May 31 20:48:39 localhost systemd[24138]: Started Mark boot as successful after the user session has run 2 minutes.
May 31 20:48:39 localhost systemd[24138]: Reached target Timers.
May 31 20:48:39 localhost systemd[24138]: Starting D-Bus User Message Bus Socket.
May 31 20:48:39 localhost systemd[24138]: Listening on D-Bus User Message Bus Socket.
May 31 20:48:39 localhost systemd[24138]: Reached target Sockets.
May 31 20:48:39 localhost systemd[24138]: Reached target Basic System.
May 31 20:48:39 localhost systemd[24138]: Reached target Default.
May 31 20:48:39 localhost systemd[24138]: Startup finished in 45ms.
May 31 20:48:39 localhost systemd[1]: Started User Manager for UID 1000.
May 31 20:49:00 localhost systemd[1]: Started Run watchlog script every 30 second.

```

7. Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса
   должно также называться. Устанавливаем spawn-fcgi и необходимые для него пакеты

```

[root@test-systemd ~]# yum install epel-release -y && yum install spawn-fcgi php php-cli
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 2:48:58 ago on Wed May 31 18:04:04 2023.
Dependencies resolved.
==========================================================================================================
 Package                     Architecture          Version                    Repository             Size
==========================================================================================================
Installing:
 epel-release                noarch                8-11.el8                   extras                 24 k

Transaction Summary
==========================================================================================================
Install  1 Package

Total download size: 24 k
Installed size: 35 k
Downloading Packages:
epel-release-8-11.el8.noarch.rpm                                           58 kB/s |  24 kB     00:00
----------------------------------------------------------------------------------------------------------
Total                                                                      57 kB/s |  24 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                  1/1
  Installing       : epel-release-8-11.el8.noarch                                                     1/1
  Running scriptlet: epel-release-8-11.el8.noarch                                                     1/1
  Verifying        : epel-release-8-11.el8.noarch                                                     1/1

Installed:
  epel-release-8-11.el8.noarch

Complete!
Failed to set locale, defaulting to C.UTF-8
Extra Packages for Enterprise Linux Modular 8 - x86_64                    1.1 MB/s | 733 kB     00:00
Extra Packages for Enterprise Linux 8 - x86_64                            4.6 MB/s |  14 MB     00:02
Last metadata expiration check: 0:00:01 ago on Wed May 31 20:53:07 2023.
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Module yaml error: Unexpected key in data: static_context [line 9 col 3]
Dependencies resolved.
==========================================================================================================
 Package                  Architecture Version                                      Repository       Size
==========================================================================================================
Installing:
 php                      x86_64       7.2.24-1.module_el8.2.0+313+b04d0a66         appstream       1.5 M
 php-cli                  x86_64       7.2.24-1.module_el8.2.0+313+b04d0a66         appstream       3.1 M
 spawn-fcgi               x86_64       1.6.3-17.el8                                 epel             24 k
Installing dependencies:
 apr                      x86_64       1.6.3-12.el8                                 appstream       129 k
 apr-util                 x86_64       1.6.1-6.el8                                  appstream       105 k
 centos-logos-httpd       noarch       85.8-2.el8                                   baseos           75 k
 httpd                    x86_64       2.4.37-43.module_el8.5.0+1022+b541f3b1       appstream       1.4 M
 httpd-filesystem         noarch       2.4.37-43.module_el8.5.0+1022+b541f3b1       appstream        39 k
 httpd-tools              x86_64       2.4.37-43.module_el8.5.0+1022+b541f3b1       appstream       107 k
 mailcap                  noarch       2.1.48-3.el8                                 baseos           39 k
 mod_http2                x86_64       1.15.7-3.module_el8.4.0+778+c970deab         appstream       154 k
 nginx-filesystem         noarch       1:1.14.1-9.module_el8.0.0+184+e34fea82       appstream        24 k
 php-common               x86_64       7.2.24-1.module_el8.2.0+313+b04d0a66         appstream       661 k
Installing weak dependencies:
 apr-util-bdb             x86_64       1.6.1-6.el8                                  appstream        25 k
 apr-util-openssl         x86_64       1.6.1-6.el8                                  appstream        27 k
 php-fpm                  x86_64       7.2.24-1.module_el8.2.0+313+b04d0a66         appstream       1.6 M
Enabling module streams:
 httpd                                 2.4
 nginx                                 1.14
 php                                   7.2

Transaction Summary
==========================================================================================================
Install  16 Packages

Total download size: 9.0 M
Installed size: 31 M
Is this ok [y/N]: y
Is this ok [y/N]: y
Downloading Packages:
(1/16): apr-util-bdb-1.6.1-6.el8.x86_64.rpm                                77 kB/s |  25 kB     00:00
(2/16): apr-util-1.6.1-6.el8.x86_64.rpm                                   241 kB/s | 105 kB     00:00
(3/16): apr-1.6.3-12.el8.x86_64.rpm                                       277 kB/s | 129 kB     00:00
(4/16): apr-util-openssl-1.6.1-6.el8.x86_64.rpm                           155 kB/s |  27 kB     00:00
(5/16): httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch.rp 316 kB/s |  39 kB     00:00
(6/16): httpd-tools-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64.rpm     479 kB/s | 107 kB     00:00
(7/16): mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64.rpm         807 kB/s | 154 kB     00:00
(8/16): nginx-filesystem-1.14.1-9.module_el8.0.0+184+e34fea82.noarch.rpm  206 kB/s |  24 kB     00:00
(9/16): php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm               3.0 MB/s | 1.5 MB     00:00
(10/16): httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64.rpm          1.2 MB/s | 1.4 MB     00:01
(11/16): php-common-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm       1.3 MB/s | 661 kB     00:00
(12/16): php-cli-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm          3.0 MB/s | 3.1 MB     00:01
(13/16): centos-logos-httpd-85.8-2.el8.noarch.rpm                         464 kB/s |  75 kB     00:00
(14/16): php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm          3.4 MB/s | 1.6 MB     00:00
(15/16): spawn-fcgi-1.6.3-17.el8.x86_64.rpm                               143 kB/s |  24 kB     00:00
(16/16): mailcap-2.1.48-3.el8.noarch.rpm                                   71 kB/s |  39 kB     00:00
----------------------------------------------------------------------------------------------------------
Total                                                                     3.0 MB/s | 9.0 MB     00:02
warning: /var/cache/dnf/epel-6c12381928511f32/packages/spawn-fcgi-1.6.3-17.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 2f86d6a1: NOKEY
Extra Packages for Enterprise Linux 8 - x86_64                            1.6 MB/s | 1.6 kB     00:00
Importing GPG key 0x2F86D6A1:
 Userid     : "Fedora EPEL (8) <epel@fedoraproject.org>"
 Fingerprint: 94E2 79EB 8D8F 25B2 1810 ADF1 21EA 45AB 2F86 D6A1
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                  1/1
  Installing       : php-common-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                          1/16
  Running scriptlet: httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch                  2/16
  Installing       : httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch                  2/16
  Installing       : apr-1.6.3-12.el8.x86_64                                                         3/16
  Running scriptlet: apr-1.6.3-12.el8.x86_64                                                         3/16
  Installing       : apr-util-bdb-1.6.1-6.el8.x86_64                                                 4/16
  Installing       : apr-util-openssl-1.6.1-6.el8.x86_64                                             5/16
  Installing       : apr-util-1.6.1-6.el8.x86_64                                                     6/16
  Running scriptlet: apr-util-1.6.1-6.el8.x86_64                                                     6/16
  Installing       : httpd-tools-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64                       7/16
  Installing       : php-cli-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                             8/16
  Installing       : mailcap-2.1.48-3.el8.noarch                                                     9/16
  Installing       : centos-logos-httpd-85.8-2.el8.noarch                                           10/16
  Installing       : mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64                          11/16
  Installing       : httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64                            12/16
  Running scriptlet: httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64                            12/16
  Running scriptlet: nginx-filesystem-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch                 13/16
  Installing       : nginx-filesystem-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch                 13/16
  Installing       : php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                            14/16
  Running scriptlet: php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                            14/16
  Installing       : php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                                15/16
  Installing       : spawn-fcgi-1.6.3-17.el8.x86_64                                                 16/16
  Running scriptlet: spawn-fcgi-1.6.3-17.el8.x86_64                                                 16/16
  Running scriptlet: httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64                            16/16
  Running scriptlet: spawn-fcgi-1.6.3-17.el8.x86_64                                                 16/16
  Running scriptlet: php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                            16/16
  Verifying        : apr-1.6.3-12.el8.x86_64                                                         1/16
  Verifying        : apr-util-1.6.1-6.el8.x86_64                                                     2/16
  Verifying        : apr-util-bdb-1.6.1-6.el8.x86_64                                                 3/16
  Verifying        : apr-util-openssl-1.6.1-6.el8.x86_64                                             4/16
  Verifying        : httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64                             5/16
  Verifying        : httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch                  6/16
  Verifying        : httpd-tools-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64                       7/16
  Verifying        : mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64                           8/16
  Verifying        : nginx-filesystem-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch                  9/16
  Verifying        : php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                                10/16
  Verifying        : php-cli-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                            11/16
  Verifying        : php-common-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                         12/16
  Verifying        : php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                            13/16
  Verifying        : centos-logos-httpd-85.8-2.el8.noarch                                           14/16
  Verifying        : mailcap-2.1.48-3.el8.noarch                                                    15/16
  Verifying        : spawn-fcgi-1.6.3-17.el8.x86_64                                                 16/16

Installed:
  apr-1.6.3-12.el8.x86_64
  apr-util-1.6.1-6.el8.x86_64
  apr-util-bdb-1.6.1-6.el8.x86_64
  apr-util-openssl-1.6.1-6.el8.x86_64
  centos-logos-httpd-85.8-2.el8.noarch
  httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64
  httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch
  httpd-tools-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64
  mailcap-2.1.48-3.el8.noarch
  mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64
  nginx-filesystem-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch
  php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-cli-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-common-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  spawn-fcgi-1.6.3-17.el8.x86_64

Complete!

```

8. /etc/rc.d/init.d/spawn-fcgi - cам Init скрипт, который будем переписывать Но перед этим необходимо
   раскомментировать строки с переменными в /etc/sysconfig/spawn-fcgi

```

До

[root@test-systemd ~]# cat /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
#SOCKET=/var/run/php-fcgi.sock
#OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"

После

[root@test-systemd ~]# cat /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"

```

9. Создаем юнит файл

```

[root@test-systemd ~]# nano /etc/systemd/system/spawn-fcgi.service


 GNU nano 2.9.8                     /etc/systemd/system/spawn-fcgi.service

[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target

```

10. Убеждаемся, что все успешно работает

```

[root@test-systemd ~]# systemctl start spawn-fcgi
[root@test-systemd ~]# systemctl status spawn-fcgi.service
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-05-31 21:05:12 UTC; 32s ago
 Main PID: 24817 (php-cgi)
    Tasks: 33 (limit: 12421)
   Memory: 18.4M
   CGroup: /system.slice/spawn-fcgi.service
           ├─24817 /usr/bin/php-cgi
           ├─24818 /usr/bin/php-cgi
           ├─24819 /usr/bin/php-cgi
           ├─24820 /usr/bin/php-cgi
           ├─24821 /usr/bin/php-cgi
           ├─24822 /usr/bin/php-cgi
           ├─24823 /usr/bin/php-cgi
           ├─24824 /usr/bin/php-cgi
           ├─24825 /usr/bin/php-cgi
           ├─24826 /usr/bin/php-cgi
           ├─24827 /usr/bin/php-cgi
           ├─24828 /usr/bin/php-cgi
           ├─24829 /usr/bin/php-cgi
           ├─24830 /usr/bin/php-cgi
           ├─24831 /usr/bin/php-cgi
           ├─24832 /usr/bin/php-cgi
           ├─24833 /usr/bin/php-cgi
           ├─24834 /usr/bin/php-cgi
           ├─24835 /usr/bin/php-cgi
           ├─24836 /usr/bin/php-cgi
           ├─24837 /usr/bin/php-cgi
           ├─24838 /usr/bin/php-cgi
           ├─24839 /usr/bin/php-cgi
           ├─24840 /usr/bin/php-cgi
           ├─24841 /usr/bin/php-cgi
           ├─24842 /usr/bin/php-cgi
           ├─24843 /usr/bin/php-cgi
           ├─24844 /usr/bin/php-cgi
           ├─24845 /usr/bin/php-cgi
           ├─24846 /usr/bin/php-cgi
           ├─24847 /usr/bin/php-cgi
           ├─24848 /usr/bin/php-cgi
           └─24849 /usr/bin/php-cgi

May 31 21:05:12 test-systemd systemd[1]: Started Spawn-fcgi startup service by Otus.

```

11. Дополнить юнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигами
   Для запуска нескольких экземпляров сервиса будем использовать шаблон в конфигурации файла окружения
   (/usr/lib/systemd/system/httpd.service ) Скопируем файл httpd.service из usr/lib/systemd/system/ в /etc/systemd/system
   и переименуем его в httpd@.service

 ```

[root@test-systemd /]# cp /usr/lib/systemd/system/httpd.service /etc/systemd/system
[root@test-systemd /]# cd /etc/systemd/system/
[root@test-systemd system]# ll
total 20
lrwxrwxrwx. 1 root root   57 Dec  4  2020 dbus-org.freedesktop.nm-dispatcher.service -> /usr/lib/systemd/system/NetworkManager-dispatcher.service
lrwxrwxrwx. 1 root root   41 Dec  4  2020 dbus-org.freedesktop.timedate1.service -> /usr/lib/systemd/system/timedatex.service
lrwxrwxrwx. 1 root root   41 Dec  4  2020 default.target -> /usr/lib/systemd/system/multi-user.target
drwxr-xr-x. 2 root root   32 Dec  4  2020 getty.target.wants
-rw-r--r--. 1 root root  944 Jun  1 11:43 httpd.service
drwxr-xr-x. 2 root root 4096 Dec  4  2020 multi-user.target.wants
drwxr-xr-x. 2 root root   48 Dec  4  2020 network-online.target.wants
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-blkmap.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-idmapd.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-mountd.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-server.service.requires
drwxr-xr-x. 2 root root    6 Oct  7  2019 nginx.service.d
drwxr-xr-x. 2 root root    6 May  7  2020 php-fpm.service.d
drwxr-xr-x. 2 root root   31 Dec  4  2020 remote-fs.target.wants
drwxr-xr-x. 2 root root   33 Dec  4  2020 rpc-gssd.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 rpc-statd-notify.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 rpc-statd.service.requires
drwxr-xr-x. 2 root root   51 Dec  4  2020 sockets.target.wants
-rw-r--r--. 1 root root  270 May 31 21:01 spawn-fcgi.service
drwxr-xr-x. 2 root root  151 Dec  4  2020 sysinit.target.wants
lrwxrwxrwx. 1 root root   39 Dec  4  2020 syslog.service -> /usr/lib/systemd/system/rsyslog.service
lrwxrwxrwx. 1 root root    9 May 11  2019 systemd-timedated.service -> /dev/null
drwxr-xr-x. 2 root root   61 Dec  4  2020 timers.target.wants
drwxr-xr-x. 2 root root   29 Dec  4  2020 vmtoolsd.service.requires
-rw-r--r--. 1 root root  142 May 31 15:30 watchlog.service
-rw-r--r--. 1 root root  166 May 31 15:31 watchlog.timer
[root@test-systemd system]# mv httpd.service httpd@.service
[root@test-systemd system]# ll
total 20
lrwxrwxrwx. 1 root root   57 Dec  4  2020 dbus-org.freedesktop.nm-dispatcher.service -> /usr/lib/systemd/system/NetworkManager-dispatcher.service
lrwxrwxrwx. 1 root root   41 Dec  4  2020 dbus-org.freedesktop.timedate1.service -> /usr/lib/systemd/system/timedatex.service
lrwxrwxrwx. 1 root root   41 Dec  4  2020 default.target -> /usr/lib/systemd/system/multi-user.target
drwxr-xr-x. 2 root root   32 Dec  4  2020 getty.target.wants
-rw-r--r--. 1 root root  944 Jun  1 11:43 httpd@.service
drwxr-xr-x. 2 root root 4096 Dec  4  2020 multi-user.target.wants
drwxr-xr-x. 2 root root   48 Dec  4  2020 network-online.target.wants
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-blkmap.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-idmapd.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-mountd.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 nfs-server.service.requires
drwxr-xr-x. 2 root root    6 Oct  7  2019 nginx.service.d
drwxr-xr-x. 2 root root    6 May  7  2020 php-fpm.service.d
drwxr-xr-x. 2 root root   31 Dec  4  2020 remote-fs.target.wants
drwxr-xr-x. 2 root root   33 Dec  4  2020 rpc-gssd.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 rpc-statd-notify.service.requires
drwxr-xr-x. 2 root root   33 Dec  4  2020 rpc-statd.service.requires
drwxr-xr-x. 2 root root   51 Dec  4  2020 sockets.target.wants
-rw-r--r--. 1 root root  270 May 31 21:01 spawn-fcgi.service
drwxr-xr-x. 2 root root  151 Dec  4  2020 sysinit.target.wants
lrwxrwxrwx. 1 root root   39 Dec  4  2020 syslog.service -> /usr/lib/systemd/system/rsyslog.service
lrwxrwxrwx. 1 root root    9 May 11  2019 systemd-timedated.service -> /dev/null
drwxr-xr-x. 2 root root   61 Dec  4  2020 timers.target.wants
drwxr-xr-x. 2 root root   29 Dec  4  2020 vmtoolsd.service.requires
-rw-r--r--. 1 root root  142 May 31 15:30 watchlog.service
-rw-r--r--. 1 root root  166 May 31 15:31 watchlog.timer
[root@test-systemd system]# nano httpd@.service

# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

# [Service]
# Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C
EnvironmentFile=/etc/sysconfig/httpd-%I
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true

[Install]
WantedBy=multi-user.target


```

12. В самом файле окружения (которых будет два) задается опция для запуска веб-сервера с необходимым конфигурационным файлом
   Создаем 2 файла в каталоге /etc/sysconfig/  и добавляем в них заданные строки 
	   # /etc/sysconfig/httpd-first
	OPTIONS=-f conf/first.conf

	# /etc/sysconfig/httpd-second
	OPTIONS=-f conf/second.conf
		
```

[root@test-systemd /]# cd /etc/sysconfig/
[root@test-systemd sysconfig]# nano httpd-first
[root@test-systemd sysconfig]# cat httpd-first
OPTIONS=-f conf/first.conf
[root@test-systemd sysconfig]#
[root@test-systemd sysconfig]# nano httpd-second
[root@test-systemd sysconfig]# cat httpd-second
OPTIONS=-f conf/second.conf
[root@test-systemd sysconfig]#

```

13. В каталоге /etc/httpd/conf находим файл httpd.conf, копируем его в этот же каталог и переименовываем в first.conf

```

[root@test-systemd conf]# cd /etc/httpd/conf
[root@test-systemd conf]# cp httpd.conf first.conf
[root@test-systemd conf]# ll
total 40
-rw-r--r--. 1 root root 11899 Jun  1 15:00 first.conf
-rw-r--r--. 1 root root 11899 Nov 12  2021 httpd.conf
-rw-r--r--. 1 root root 13064 Nov 12  2021 magic

```

14. Еще раз в каталоге /etc/httpd/conf находим файл httpd.conf, копируем его в этот же каталог и переименовываем в second.conf,
   и согласно заданию добавляем PidFile /var/run/httpd-second.pid - т.е. должен быть указан файл пида и    Listen 8080 - указан порт,
   который будет отличаться от другого инстанса

```

[root@test-systemd conf]#  cd /etc/httpd/conf
[root@test-systemd conf]# cp httpd.conf second.conf
[root@test-systemd conf]# ll
total 180
-rw-r--r--. 1 root root  11899 Jun  1 15:15  first.conf
-rw-r--r--. 1 root root  11899 Nov 12  2021  httpd.conf
-rw-r--r--. 1 root root  13064 Nov 12  2021  magic
-rw-r--r--. 1 root root  11934 Jun  1 15:36  second.conf

# Do not add a slash at the end of the directory path.  If you point
# ServerRoot at a non-local disk, be sure to specify a local disk on the
# Mutex directive, if file-based mutexes are used.  If you wish to share the
# same ServerRoot for multiple httpd daemons, you will need to change at
# least PidFile.
#
ServerRoot "/etc/httpd"
PidFile /var/run/httpd-second.pid
#
# Listen: Allows you to bind Apache to specific IP addresses and/or
# ports, instead of the default. See also the <VirtualHost>
# directive.
#
# Change this to Listen on specific IP addresses as shown below to
# prevent Apache from glomming onto all bound IP addresses.
#
#Listen 12.34.56.78:80
Listen 8080

```

15. Запускаем наши сервисы

```

[root@test-systemd conf]# systemctl enable  httpd@first
Created symlink /etc/systemd/system/multi-user.target.wants/httpd@first.service → /etc/systemd/system/httpd@.service.
[root@test-systemd conf]# systemctl enable  httpd@second
Created symlink /etc/systemd/system/multi-user.target.wants/httpd@second.service → /etc/systemd/system/httpd@.service.

[root@test-systemd conf]# systemctl start  httpd@first
[root@test-systemd conf]# systemctl start  httpd@second

```

16. Проверяем запущенные сервисы и порты

```

[root@test-systemd ~]# ss -tnulp | grep httpd
tcp     LISTEN   0        128                    *:80                  *:*       users:(("httpd",pid=720,fd=4),("httpd",pid=719,fd=4),("httpd",pid=718,fd=4),("httpd",pid=711,fd=4))
tcp     LISTEN   0        128                    *:8080                *:*       users:(("httpd",pid=935,fd=4),("httpd",pid=934,fd=4),("httpd",pid=933,fd=4),("httpd",pid=713,fd=4))


[root@test-systemd system]# systemctl status ht
htcacheclean.service  httpd@first.service   httpd-init.service    httpd@second.service  httpd.service         httpd.socket
[root@test-systemd system]# systemctl status ht
htcacheclean.service  httpd@first.service   httpd-init.service    httpd@second.service  httpd.service         httpd.socket
[root@test-systemd system]# systemctl status httpd@first.service
● httpd@first.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-06-01 15:38:45 UTC; 24min ago
     Docs: man:httpd.service(8)
 Main PID: 711 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 12421)
   Memory: 26.7M
   CGroup: /system.slice/system-httpd.slice/httpd@first.service
           ├─711 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─717 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─718 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─719 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           └─720 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND

Jun 01 15:38:44 test-systemd systemd[1]: Starting The Apache HTTP Server...
Jun 01 15:38:45 test-systemd httpd[711]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress t>
Jun 01 15:38:45 test-systemd systemd[1]: Started The Apache HTTP Server.
Jun 01 15:38:45 test-systemd httpd[711]: Server configured, listening on: port 80


[root@test-systemd system]# systemctl status httpd@second.service
● httpd@second.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-06-01 15:38:45 UTC; 24min ago
     Docs: man:httpd.service(8)
 Main PID: 713 (httpd)
   Status: "Running, listening on: port 8080"
    Tasks: 213 (limit: 12421)
   Memory: 35.2M
   CGroup: /system.slice/system-httpd.slice/httpd@second.service
           ├─713 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─932 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─933 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─934 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           └─935 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND

Jun 01 15:38:44 test-systemd systemd[1]: Starting The Apache HTTP Server...
Jun 01 15:38:45 test-systemd httpd[713]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress t>
Jun 01 15:38:45 test-systemd systemd[1]: Started The Apache HTTP Server.
Jun 01 15:38:45 test-systemd httpd[713]: Server configured, listening on: port 8080





# dz-8
