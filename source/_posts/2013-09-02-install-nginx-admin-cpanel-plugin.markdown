---
layout: post
title: "Install Nginx admin cPanel plugin"
date: 2013-09-02 15:34
comments: true
categories: SysAdmin Nginx NginxAdmin cPanel WHM Plugin
---

There has been a lot of those guides but here’s another one to the collection with minor improvements. 
In this tutorial we will install step by step the Nginx admin cPanel plugin.

###Installation for 32Bit/64Bit

Step 1 Install Nginx Admin

        cd /usr/local/src
        wget http://nginxcp.com/latest/nginxadmin.tar
        tar xf nginxadmin.tar
        cd publicnginx
        ./nginxinstaller install

<!-- more -->

step 2 restart Apache
    
        /etc/init.d/httpd restart

or

        service httpd restart

step 3 Automated */tmp* cleanup by add a cron job

        crontab -e
        0 */1 * * * /usr/sbin/tmpwatch -am 1 /tmp/nginx_client

###Configuration

For VPS server with 1 core and 1GB memory, This is usually the default settings in Nginx Admin :

        worker_processes  2;
        worker_connections 5120; # increase for busier servers
        keepalive_timeout  30;
 

For dedicated server with 4 core and up to 12GB memory:

        worker_processes  4;
        worker_connections 10240; # increase for busier servers
        keepalive_timeout  60;
 

For suPHP add the lines:

        fastcgi_send_timeout 60;
        fastcgi_read_timeout 60;
        fastcgi_connect_timeout 60;


### Troubleshooting

#### If you received this error in installation:

        Traceback (most recent call last):
        File “/usr/local/src/publicnginx/nginxinstaller2″, line 9, in ?import createvhosts
        File “/usr/local/src/publicnginx/createvhosts.py”, line 2, in ?import yaml
        File “/usr/lib/python2.4/site-packages/PyYAML-3.10-py2.4-linux-i686.egg/yaml/__init__.py”, line 26
        SyntaxError: ‘yield’ not allowed in a ‘try’ block with a ‘finally’ clause
 

Try to run pythonfix script and reinstall:

        ./pythonfix
 

If for any reason “pythonfix” not working try to downgrade python manually.

        wget http://www.python.org/ftp/python/2.5.2/Python-2.5.2.tgz
        tar fxz Python-2.5.2.tgz
        cd Python-2.5.2
        ./configure
        make
        make install
 

#### If you received this error in httpd restart.

        nginx: [emerg] bind() to xxx.xxx.47.162:80 failed (98: Address already in use)
        nginx: [emerg] bind() to xxx.xxx.47.164:80 failed (98: Address already in use)
        nginx: [emerg] still could not bind()
        already running.

Possibly that Apache is still running on port 80.

The solution go to:

1. Main >> Server Configuration >> Tweak Settings in WHM the Apache non-SSL IP/port and make sure that is set to 8081.
2. Main >> Service Configuration >> Apache Configuration >> Global Configuration save and “Rebuilt Configuration and restart Apache”.
3. Main >> Plugins >> Nginx Admin >> Rebuild Vhosts

#### If when restart nginx it show this error:

        # service nginx restart
        Restarting nginx daemon: nginxRemaining processes: 1406 11903
        Remaining processes: 1406
        cat: /var/run/nginx.pid: No such file or directory
        kill: usage: kill [-s sigspec | -n signum | -sigspec] pid | jobspec ... or      kill -l [sigspec]
        not runningRemaining processes: 1406
 

Clear all processes under nginx run:

        killall -15 nginx
        /etc/init.d/httpd stop
        ipcs -s | grep nobody | perl -e 'while () { @a=split(/\s+/); print `ipcrm sem $a[1]`}'
        /etc/init.d/httpd start
 

#### If you received this error in Nginx restart.

        Restarting nginx daemon: nginxRemaining processes: 24323.

Run EasyApache

        /scripts/easyapache

