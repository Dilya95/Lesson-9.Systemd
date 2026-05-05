# Домашнее задание 9: Systemd — создание unit-файла

## Задания
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.


## Структура
ansible-playbooks - выполнение заданий с помощью ansible плейбуков
README.md - ход выполнения команд по методичке с результатами


## Выполнение

### Задание 1
```
root@otus-homework:~# nano /etc/default/watchlog

root@otus-homework:~# cat /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log


root@otus-homework:~# nano /var/log/watchlog.log

root@otus-homework:~# cat /var/log/watchlog.log
INFO: System started
ALERT: High CPU usage
DEBUG: Processing request
ALERT: Memory limit exceeded
INFO: Normal operation


root@otus-homework:~# nano /opt/watchlog.sh

root@otus-homework:~# cat  /opt/watchlog.sh
#!/bin/bash

WORD="$1"
LOG="$2"
DATE=$(date)

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

root@otus-homework:~# chmod +x /opt/watchlog.sh

root@otus-homework:~# ls -la /opt/watchlog.sh
-rwxr-xr-x 1 root root 131 May  5 10:03 /opt/watchlog.sh


root@otus-homework:~# nano /etc/systemd/system/watchlog.service

root@otus-homework:~# cat /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG


root@otus-homework:~# systemctl start watchlog.service


root@otus-homework:~# nano /etc/systemd/system/watchlog.timer

root@otus-homework:~# cat  /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target


root@otus-homework:~# systemctl start watchlog.timer


root@otus-homework:~# tail -n 1000 /var/log/syslog  | grep word
May  5 10:00:19 otus-homework kernel: [    4.865163] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
May  5 10:33:49 otus-homework root: Tue May  5 10:33:49 AM UTC 2026: I found word, Master!
May  5 10:34:37 otus-homework root: Tue May  5 10:34:37 AM UTC 2026: I found word, Master!
May  5 10:35:20 otus-homework root: Tue May  5 10:35:20 AM UTC 2026: I found word, Master!
May  5 10:36:37 otus-homework root: Tue May  5 10:36:37 AM UTC 2026: I found word, Master!

```

### Задание 2
```
root@otus-homework:/# apt install spawn-fcgi php php-cgi php-cli  apache2 libapache2-mod-fcgid -y


root@otus-homework:~# mkdir /etc/spawn-fcgi/

root@otus-homework:~# nano /etc/spawn-fcgi/fcgi.conf

root@otus-homework:~# cat /etc/spawn-fcgi/fcgi.conf
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"


root@otus-homework:~# nano /etc/systemd/system/spawn-fcgi.service

root@otus-homework:~# cat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target


root@otus-homework:~# systemctl start spawn-fcgi

root@otus-homework:~# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2026-05-05 08:54:13 UTC; 3s ago
   Main PID: 1229 (php-cgi)
      Tasks: 33 (limit: 9385)
     Memory: 18.9M
        CPU: 31ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─1229 /usr/bin/php-cgi
             ├─1230 /usr/bin/php-cgi
             ├─1231 /usr/bin/php-cgi
             ├─1232 /usr/bin/php-cgi
             ├─1233 /usr/bin/php-cgi
             ├─1234 /usr/bin/php-cgi
             ├─1235 /usr/bin/php-cgi
             ├─1236 /usr/bin/php-cgi
             ├─1237 /usr/bin/php-cgi
             ├─1238 /usr/bin/php-cgi
             ├─1239 /usr/bin/php-cgi
             ├─1240 /usr/bin/php-cgi
             ├─1241 /usr/bin/php-cgi
             ├─1242 /usr/bin/php-cgi
             ├─1243 /usr/bin/php-cgi
             ├─1244 /usr/bin/php-cgi
             ├─1245 /usr/bin/php-cgi
             ├─1246 /usr/bin/php-cgi
             ├─1247 /usr/bin/php-cgi
             ├─1248 /usr/bin/php-cgi
             ├─1249 /usr/bin/php-cgi
             ├─1250 /usr/bin/php-cgi
             ├─1251 /usr/bin/php-cgi
             ├─1252 /usr/bin/php-cgi
             ├─1253 /usr/bin/php-cgi
             ├─1254 /usr/bin/php-cgi
             ├─1255 /usr/bin/php-cgi
             ├─1256 /usr/bin/php-cgi
             ├─1257 /usr/bin/php-cgi
             ├─1258 /usr/bin/php-cgi
             ├─1259 /usr/bin/php-cgi
             ├─1260 /usr/bin/php-cgi
             └─1261 /usr/bin/php-cgi

May 05 08:54:13 otus-homework systemd[1]: Started Spawn-fcgi startup service by Otus.
```


### Задание 3
```
root@otus-homework:~# apt install nginx -y

root@otus-homework:~# nano /etc/systemd/system/nginx@.service

root@otus-homework:~# cat /etc/systemd/system/nginx@.service
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target


root@otus-homework:~# cp /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf

root@otus-homework:~# cp /etc/nginx/nginx.conf /etc/nginx/nginx-second.conf

root@otus-homework:~# nano /etc/nginx/nginx-first.conf

root@otus-homework:~# nano /etc/nginx/nginx-second.conf


root@otus-homework:~# cat /etc/nginx/nginx-first.conf
user www-data;
worker_processes auto;
pid /run/nginx-first.pid;
#include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##
        server {
        listen 9001;
        server_name localhost;
        
        root /var/www/html;
        index index.html index.htm;
        
        location / {
            try_files $uri $uri/ =404;
        }
    }
        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        #include /etc/nginx/conf.d/*.conf;
        #include /etc/nginx/sites-enabled/*;

}


root@otus-homework:~# nginx -t -c /etc/nginx/nginx-first.conf
nginx: the configuration file /etc/nginx/nginx-first.conf syntax is ok
nginx: configuration file /etc/nginx/nginx-first.conf test is successful


root@otus-homework:~# nginx -t -c /etc/nginx/nginx-second.conf
nginx: the configuration file /etc/nginx/nginx-second.conf syntax is ok
nginx: configuration file /etc/nginx/nginx-second.conf test is successful



root@otus-homework:~# systemctl start nginx@first

root@otus-homework:~# systemctl start nginx@second

root@otus-homework:~# systemctl status nginx@first
● nginx@first.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2026-05-05 09:17:46 UTC; 33s ago
       Docs: man:nginx(8)
   Main PID: 1966 (nginx)
      Tasks: 3 (limit: 9385)
     Memory: 2.2M
        CPU: 11ms
     CGroup: /system.slice/system-nginx.slice/nginx@first.service
             ├─1966 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on;"
             ├─1967 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             └─1968 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""

May 05 09:17:46 otus-homework systemd[1]: Starting A high performance web server and a reverse proxy server...
May 05 09:17:46 otus-homework systemd[1]: Started A high performance web server and a reverse proxy server.


root@otus-homework:~# systemctl status nginx@second
● nginx@second.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2026-05-05 09:18:08 UTC; 14s ago
       Docs: man:nginx(8)
    Process: 2004 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-second.conf -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2005 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 2006 (nginx)
      Tasks: 3 (limit: 9385)
     Memory: 2.3M
        CPU: 11ms
     CGroup: /system.slice/system-nginx.slice/nginx@second.service
             ├─2006 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;"
             ├─2007 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             └─2008 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""

May 05 09:18:08 otus-homework systemd[1]: Starting A high performance web server and a reverse proxy server...
May 05 09:18:08 otus-homework systemd[1]: Started A high performance web server and a reverse proxy server.
 

root@otus-homework:~# ss -tnulp | grep nginx
tcp   LISTEN 0      511                             0.0.0.0:9002      0.0.0.0:*    users:(("nginx",pid=2008,fd=6),("nginx",pid=2007,fd=6),("nginx",pid=2006,fd=6))                                                                                                                                 
tcp   LISTEN 0      511                             0.0.0.0:9001      0.0.0.0:*    users:(("nginx",pid=1968,fd=6),("nginx",pid=1967,fd=6),("nginx",pid=1966,fd=6))                                                                                                                                 

root@otus-homework:~# ps afx | grep nginx
   2153 pts/0    S+     0:00          \_ grep --color=auto nginx
   1966 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on;
   1967 ?        S      0:00  \_ nginx: worker process
   1968 ?        S      0:00  \_ nginx: worker process
   2006 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;
   2007 ?        S      0:00  \_ nginx: worker process
   2008 ?        S      0:00  \_ nginx: worker process
```
