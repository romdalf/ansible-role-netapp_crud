# Mise-à-jour de Spacewwalk 2.5 à 2.6

### Table des matières
* Contexte
* Prérequis
* Configuration 
* Sauvegarde de la configration
* Mise-à-jour système
* Mise-à-jour de spacewalk et de ces services

### Contexte
Lors de la montée de version pour des raisons de fonctionnalités ou maintenance du produit spacewalk (outil de gestion des systèmes linux), il est 
impérative de réaliser ces étapes une séquence précise. 

### Prérequis
Avoir accès à l'internet pour la mise à jour des packages.
Avoir de l'espace disque suffisant pour réaliser le backup de la DB.
Prendre un snapshot de la VM avant action.
Mettre à jour le fichier repo spacewalk pour passer de la version 2.x vers 2.y

### Sauvegarde de la configration
Prendre un snapshot de la machine virutelle en cas de gros dérapage.

Il est impératif de réaliser une sauvegarde de l'environnement et de ces fichiers de configurations.
A cette fin, créez un répertoire de type "sauvegarde-201xyyzz" avec la copie des répertoire ci-dessous:
```
[root@si5148v ~]# ls -al spacewalk-backup-20180613/
total 28
drwxr-xr-x   7 root root 4096 13 jun 10:34 .
dr-xr-x---. 29 root root 4096 13 jun 10:46 ..
drwxr-xr-x   3 root root 4096 12 jun 16:26 etc-jabberd
drwxr-xr-x   4 root root 4096 12 jun 16:25 etc-rhn
drwxr-xr-x   8 root root 4096 12 jun 16:24 etc-sysconfig-rhn
drwxr-xr-x   2 root root 4096 13 jun 10:39 pgbackup
drwxr-xr-x   4 root root 4096 12 jun 16:26 root-ssl-build
```
Les noms de répertoires correspondent à:
etc-jabberd -> /etc/jabberd 
etc-rhn -> /etc/rhn
etc-sysconfig-rhn -> /etc/sysconfig/rhn
root-ssl-build -> /root/ssl-build/

et donc sont les répertoires à les fichiers de configuration à sauvegarder via une commande du genre:
```
[root@si5148v ~]# cp -R /etc/jabberd /root/spacewalk-backup-20180613/etc-jabberd/
```

Le répertoire pgbackup correspond à l'emplacement pour le dump de la base de données. 
Attention, pour cette dernière action, il faut impérativement s'assurer que l'emplacement est sur un disque
disposant d'espace en suffisance.

Pour réaliser le backup de la base de données, il suffit de réaliser les commandes ci-dessous:
```
[root@si5148v ~]# grep db_ /etc/rhn/rhn.conf
db_backend = postgresql
db_user = xxxxx
db_password = yyyyy
db_name = zzzzz
db_host =
db_port =
db_ssl_enabled =
[root@si5148v ~]# pg_dump -U xxxxx zzzzz > rhnschema-20180613
Mot de passe : yyyyy
```

### Mise-à-jour système 
Afin d'assurer une mise-à-jour transversale, il est important de mettre à jour la partie système avant spacewalk.
Il est à noter qu'il est conseillé par RedHat et le groupe de développers spacwalk de fixer la verison de certain
paquet afin qu'il ne soit pas mis à jour. Cela peut se faire pour peut que le paquet yum-plugin-versionlock soit
installé.
Pour ce guide, la référence est: https://github.com/spacewalkproject/spacewalk/wiki/HowToUpgrade26

Pour ce faire réaliser:
```
[root@si5148v ~]# yum install yum-plugin-versionlock
[root@si5148v ~]# yum versionlock cglib
Modules complémentaires chargés : fastestmirror, versionlock
Adding versionlock on: 0:cglib-2.1.3-4.jpp5
versionlock added: 1
[root@si5148v ~]# yum upgrade 
```
Les commandes ci-dessous vous s'assurer mettre à jour le système et les packages de spacewalk par la même occasion.
Suivant l'ampleur de la mise-à-jour, il est intéressant de réaliser une vérification des fichiers de configuration 
qui aurait pu être modifié suite à cette mise-à-jour, cela peut se faire via:

```
[root@si5148v ~]# rpmconf -a
Configuration file `/etc/nsswitch.conf'
-rw-r--r-- 1 root root 1728 12 sep  2016 /etc/nsswitch.conf
-rw-r--r-- 1 root root 1735 10 avr 09:54 /etc/nsswitch.conf.rpmnew
 ==> Package distributor has shipped an updated version.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      M     : merge configuration files
      Z     : background this process to examine the situation
      S     : skip this file
 The default action is to keep your current version.
*** aliases (Y/I/N/O/D/Z/S) [default=N] ?
Your choice: N
```

### Mise-à-jour de spacewalk et de ces services

Vérifier l'état des services et les arrêtés:

```
[root@si5148v ~]# /usr/sbin/spacewalk-service status
Redirecting to /bin/systemctl status postgresql.service
● postgresql.service - PostgreSQL database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
   Active: active (running) since mer 2018-06-13 11:08:47 CEST; 16min ago
 Main PID: 22903 (postgres)
   CGroup: /system.slice/postgresql.service
           ├─22903 /usr/bin/postgres -D /var/lib/pgsql/data -p 5432
           ├─22904 postgres: logger process
           ├─22906 postgres: checkpointer process
           ├─22907 postgres: writer process
           ├─22908 postgres: wal writer process
           ├─22909 postgres: autovacuum launcher process
           ├─22910 postgres: stats collector process
           ├─23243 postgres: rhnuser rhnschema 127.0.0.1(38729) idle
           ├─23244 postgres: rhnuser rhnschema 127.0.0.1(38728) idle
           ├─23245 postgres: rhnuser rhnschema 127.0.0.1(38732) idle
           ├─23258 postgres: rhnuser rhnschema 127.0.0.1(38734) idle
           ├─23259 postgres: rhnuser rhnschema 127.0.0.1(38736) idle
           ├─23260 postgres: rhnuser rhnschema 127.0.0.1(38738) idle
           ├─23262 postgres: rhnuser rhnschema 127.0.0.1(38740) idle
           ├─34914 postgres: rhnuser rhnschema 127.0.0.1(38746) idle
           ├─35229 postgres: rhnuser rhnschema 127.0.0.1(38748) idle
           ├─35231 postgres: rhnuser rhnschema 127.0.0.1(38752) idle
           ├─35251 postgres: rhnuser rhnschema 127.0.0.1(38766) idle
           ├─41915 postgres: rhnuser rhnschema 127.0.0.1(38780) idle
           ├─46781 postgres: rhnuser rhnschema 127.0.0.1(38878) idle
           ├─46782 postgres: rhnuser rhnschema 127.0.0.1(38880) idle
           ├─46783 postgres: rhnuser rhnschema 127.0.0.1(38882) idle
           ├─46961 postgres: rhnuser rhnschema 127.0.0.1(38966) idle
           ├─46962 postgres: rhnuser rhnschema 127.0.0.1(38968) idle
           ├─47023 postgres: rhnuser rhnschema 127.0.0.1(39022) idle
           ├─47201 postgres: rhnuser rhnschema 127.0.0.1(39088) idle
           ├─47202 postgres: rhnuser rhnschema 127.0.0.1(39090) idle
           ├─47203 postgres: rhnuser rhnschema 127.0.0.1(39092) idle
           └─47206 postgres: rhnuser rhnschema 127.0.0.1(39094) idle

jun 13 11:08:46 si5148v.wallonie.intra systemd[1]: Starting PostgreSQL database server...
jun 13 11:08:47 si5148v.wallonie.intra systemd[1]: Started PostgreSQL database server.
Redirecting to /bin/systemctl status jabberd.service
● jabberd.service - Jabber Server
   Loaded: loaded (/usr/lib/systemd/system/jabberd.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since mer 2018-06-13 10:39:53 CEST; 45min ago
 Main PID: 4151 (code=exited, status=0/SUCCESS)

aoû 25 11:58:29 si5148v.wallonie.intra systemd[1]: Starting Jabber Server...
aoû 25 11:58:29 si5148v.wallonie.intra systemd[1]: Started Jabber Server.
jun 13 10:39:53 si5148v.wallonie.intra systemd[1]: Stopped Jabber Server.
jun 13 10:39:53 si5148v.wallonie.intra systemd[1]: Stopping Jabber Server...
Redirecting to /bin/systemctl status tomcat.service
● tomcat.service - Apache Tomcat Web Application Container
   Loaded: loaded (/usr/lib/systemd/system/tomcat.service; enabled; vendor preset: disabled)
   Active: active (running) since mer 2018-06-13 11:08:09 CEST; 16min ago
 Main PID: 21523 (java)
   CGroup: /system.slice/tomcat.service
           └─21523 /usr/lib/jvm/jre/bin/java -ea -Xms256m -Xmx256m -Djava.awt.headless=true -Dorg.xml.sax.driver=org.apache.xerces.parsers.SAXParser -Dorg.apache.tomcat.util.http.Parameters.MAX_COUNT=1024 ...

jun 13 11:13:23 si5148v.wallonie.intra server[21523]: Exception in thread "Timer-0" java.lang.NoClassDefFoundError: com/mchange/v2/resourcepool/BasicResourcePool$AsyncTestIdleResourceTask
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: at com.mchange.v2.resourcepool.BasicResourcePool.checkIdleResources(BasicResourcePool.java:1481)
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: at com.mchange.v2.resourcepool.BasicResourcePool.access$2000(BasicResourcePool.java:32)
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: at com.mchange.v2.resourcepool.BasicResourcePool$CheckIdleResourcesTask.run(BasicResourcePool.java:1964)
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: at java.util.TimerThread.mainLoop(Timer.java:555)
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: at java.util.TimerThread.run(Timer.java:505)
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: Caused by: java.lang.ClassNotFoundException: com.mchange.v2.resourcepool.BasicResourcePool$AsyncTestIdleResourceTask
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1892)
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1735)
jun 13 11:13:23 si5148v.wallonie.intra server[21523]: ... 5 more
Redirecting to /bin/systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since mer 2018-06-13 11:09:05 CEST; 16min ago
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 23233 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=0/SUCCESS)
 Main PID: 23242 (httpd)
   Status: "Total requests: 44; Current requests/sec: 0.2; Current traffic: 102 B/sec"
   CGroup: /system.slice/httpd.service
           ├─23242 /usr/sbin/httpd -DFOREGROUND
           ├─23246 /usr/sbin/httpd -DFOREGROUND
           ├─23247 /usr/sbin/httpd -DFOREGROUND
           ├─23248 /usr/sbin/httpd -DFOREGROUND
           ├─23249 /usr/sbin/httpd -DFOREGROUND
           ├─23250 /usr/sbin/httpd -DFOREGROUND
           ├─23251 /usr/sbin/httpd -DFOREGROUND
           ├─23252 /usr/sbin/httpd -DFOREGROUND
           └─23253 /usr/sbin/httpd -DFOREGROUND

jun 13 11:09:05 si5148v.wallonie.intra systemd[1]: Starting The Apache HTTP Server...
jun 13 11:09:05 si5148v.wallonie.intra systemd[1]: Started The Apache HTTP Server.
Redirecting to /bin/systemctl status osa-dispatcher.service
● osa-dispatcher.service - OSA Dispatcher daemon
   Loaded: loaded (/usr/lib/systemd/system/osa-dispatcher.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since mer 2018-06-13 10:40:08 CEST; 44min ago
 Main PID: 4360 (code=exited, status=255)

aoû 25 11:59:01 si5148v.wallonie.intra systemd[1]: Starting OSA Dispatcher daemon...
aoû 25 11:59:02 si5148v.wallonie.intra systemd[1]: Started OSA Dispatcher daemon.
jun 13 10:40:08 si5148v.wallonie.intra systemd[1]: osa-dispatcher.service: main process exited, code=exited, status=255/n/a
jun 13 10:40:08 si5148v.wallonie.intra systemd[1]: Unit osa-dispatcher.service entered failed state.
jun 13 10:40:08 si5148v.wallonie.intra systemd[1]: osa-dispatcher.service failed.
Redirecting to /bin/systemctl status rhn-search.service
● rhn-search.service - Spacewalk search engine
   Loaded: loaded (/usr/lib/systemd/system/rhn-search.service; enabled; vendor preset: disabled)
   Active: active (running) since ven 2017-08-25 11:59:02 CEST; 9 months 18 days ago
 Main PID: 4389 (rhnsearchd)
   CGroup: /system.slice/rhn-search.service
           ├─4389 /usr/bin/rhnsearchd /usr/share/rhn/config-defaults/rhn_search_daemon.conf wrapper.pidfile=/var/run/rhn-search.pid wrapper.daemonize=TRUE
           └─4391 /usr/bin/java -Djava.library.path=/usr/lib:/usr/lib64:/usr/lib/oracle/11.2/client/lib:/usr/lib/oracle/11.2/client64/lib:/usr/lib/gcj/postgresql-jdbc:/usr/lib64/gcj/postgresql-jdbc -classp...

aoû 25 11:59:02 si5148v.wallonie.intra systemd[1]: Starting Spacewalk search engine...
aoû 25 11:59:02 si5148v.wallonie.intra rhn-search[4376]: Starting rhn-search...
aoû 25 11:59:02 si5148v.wallonie.intra systemd[1]: Started Spacewalk search engine.
● cobblerd.service - LSB: daemon for libvirt virtualization API
   Loaded: loaded (/etc/rc.d/init.d/cobblerd; bad; vendor preset: disabled)
   Active: active (running) since dim 2018-06-10 03:15:05 CEST; 3 days ago
     Docs: man:systemd-sysv-generator(8)
   CGroup: /system.slice/cobblerd.service
           └─59039 /usr/bin/python -s /bin/cobblerd --daemonize

jun 13 11:08:49 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:11] Failed to add dependency on network,.service, ignoring: Invalid argument
jun 13 11:08:49 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:12] Failed to add dependency on xinetd,.service, ignoring: Invalid argument
jun 13 11:08:49 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:11] Failed to add dependency on network,.service, ignoring: Invalid argument
jun 13 11:08:49 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:12] Failed to add dependency on xinetd,.service, ignoring: Invalid argument
jun 13 11:08:50 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:11] Failed to add dependency on network,.service, ignoring: Invalid argument
jun 13 11:08:50 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:12] Failed to add dependency on xinetd,.service, ignoring: Invalid argument
jun 13 11:08:53 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:11] Failed to add dependency on network,.service, ignoring: Invalid argument
jun 13 11:08:53 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:12] Failed to add dependency on xinetd,.service, ignoring: Invalid argument
jun 13 11:08:53 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:11] Failed to add dependency on network,.service, ignoring: Invalid argument
jun 13 11:08:53 si5148v.wallonie.intra systemd[1]: [/run/systemd/generator.late/cobblerd.service:12] Failed to add dependency on xinetd,.service, ignoring: Invalid argument
RHN Taskomatic is running (4548).

[root@si5148v ~]# /usr/sbin/spacewalk-service stop
Shutting down spacewalk services...
Stopping RHN Taskomatic...
Stopped RHN Taskomatic.
Stopping cobblerd (via systemctl):                         [  OK  ]
Redirecting to /bin/systemctl stop rhn-search.service
Redirecting to /bin/systemctl stop osa-dispatcher.service
Redirecting to /bin/systemctl stop httpd.service
Redirecting to /bin/systemctl stop tomcat.service
Redirecting to /bin/systemctl stop jabberd.service
Redirecting to /bin/systemctl stop postgresql.service
Done.
```

```
[root@si5148v ~]# systemctl status postgresql.service
● postgresql.service - PostgreSQL database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
   Active: active (running) since mer 2018-06-13 11:28:58 CEST; 4s ago
  Process: 47613 ExecStop=/usr/bin/pg_ctl stop -D ${PGDATA} -s -m fast (code=exited, status=0/SUCCESS)
  Process: 47671 ExecStart=/usr/bin/pg_ctl start -D ${PGDATA} -s -o -p ${PGPORT} -w -t 300 (code=exited, status=0/SUCCESS)
  Process: 47665 ExecStartPre=/usr/bin/postgresql-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 47674 (postgres)
   CGroup: /system.slice/postgresql.service
           ├─47674 /usr/bin/postgres -D /var/lib/pgsql/data -p 5432
           ├─47675 postgres: logger process
           ├─47677 postgres: checkpointer process
           ├─47678 postgres: writer process
           ├─47679 postgres: wal writer process
           ├─47680 postgres: autovacuum launcher process
           └─47681 postgres: stats collector process

jun 13 11:28:57 si5148v.wallonie.intra systemd[1]: Starting PostgreSQL database server...
jun 13 11:28:58 si5148v.wallonie.intra systemd[1]: Started PostgreSQL database server.
[root@si5148v yum.repos.d]# /usr/bin/spacewalk-schema-upgrade
Schema upgrade: [spacewalk-schema-2.5.24-1.el7] -> [spacewalk-schema-2.6.17-1.el7]
Searching for upgrade path: [spacewalk-schema-2.5.24-1] -> [spacewalk-schema-2.6.17-1]
Searching for upgrade path: [spacewalk-schema-2.5.24] -> [spacewalk-schema-2.6.17]
Searching for upgrade path: [spacewalk-schema-2.5] -> [spacewalk-schema-2.6]
The path: [spacewalk-schema-2.5] -> [spacewalk-schema-2.6]
Planning to run spacewalk-sql with [/var/log/spacewalk/schema-upgrade/20180613-114315-script.sql]

Plase make sure you have a valid backup of your database before continuing.

Hit Enter to continue or Ctrl+C to interrupt:
Executing spacewalk-sql, the log is in [/var/log/spacewalk/schema-upgrade/20180613-114315-to-spacewalk-schema-2.6.log].
The database schema was upgraded to version [spacewalk-schema-2.6.17-1.el7].
```

Procédez à la mise à jour de la configuration de spacewalk via:
```
[root@si5148v yum.repos.d]# spacewalk-setup --upgrade
** Database: Setting up database connection for PostgreSQL backend.
** Database: Embedded database installation SKIPPED.
** Database: Populating database.
** Database: Skipping database population.
* Configuring tomcat.
* Setting up users and groups.
** GPG: Initializing GPG and importing key.
* Performing initial configuration.
* Configuring apache SSL virtual host.
Should setup configure apache's default ssl server for you (saves original ssl.conf) [Y]? y
* Configuring jabberd.
* Creating SSL certificates.
** Skipping SSL certificate generation.
* Deploying configuration files.
* Update configuration in database.
* Setting up Cobbler..
Cobbler requires tftp and xinetd services be turned on for PXE provisioning functionality. Enable these services [Y]? y
This portion of the Spacewalk upgrade process has successfully completed.
```

Redémarrez les services:
```
[root@si5148v yum.repos.d]# systemctl daemon-reload
[root@si5148v yum.repos.d]# /usr/sbin/spacewalk-service restart
Shutting down spacewalk services...
Redirecting to /bin/systemctl stop taskomatic.service
Stopping cobblerd (via systemctl):                         [  OK  ]
Redirecting to /bin/systemctl stop rhn-search.service
Redirecting to /bin/systemctl stop osa-dispatcher.service
Redirecting to /bin/systemctl stop httpd.service
Redirecting to /bin/systemctl stop tomcat.service
Redirecting to /bin/systemctl stop jabberd.service
Redirecting to /bin/systemctl stop postgresql.service
Done.
Starting spacewalk services...
Redirecting to /bin/systemctl start postgresql.service
Redirecting to /bin/systemctl start jabberd.service
Redirecting to /bin/systemctl start tomcat.service
Waiting for tomcat to be ready ...
Redirecting to /bin/systemctl start httpd.service
Redirecting to /bin/systemctl start osa-dispatcher.service
Job for osa-dispatcher.service failed because the control process exited with error code. See "systemctl status osa-dispatcher.service" and "journalctl -xe" for details.
Redirecting to /bin/systemctl start rhn-search.service
Starting cobblerd (via systemctl):                         [  OK  ]
Redirecting to /bin/systemctl start taskomatic.service
Done.
[root@si5148v yum.repos.d]#
```

Fixer la version du package c3p0 via le repo jpackage
```
[root@si5148v ~]# yum --showduplicates list c3p0
[root@si5148v ~]# yum install c3p0-0.9.1.2-2.jpp5
[root@si5148v ~]# yum remove c3p0-0.9.2.1-6.el7
[root@si5148v ~]# yum downgrade c3p0-0.9.1.2-2.jpp5
```

Resoudre le changement de schema de jabberd l'empêchant de redémarrer proprement
```
[root@si5148v ~]# cd /var/lib/jabberd/
[root@si5148v ~]# rm -rf *
```

Redémarrer tous les services:
```
[root@si5148v yum.repos.d]# systemctl daemon-reload
[root@si5148v yum.repos.d]# /usr/sbin/spacewalk-service restart
Shutting down spacewalk services...
Redirecting to /bin/systemctl stop taskomatic.service
Stopping cobblerd (via systemctl):                         [  OK  ]
Redirecting to /bin/systemctl stop rhn-search.service
Redirecting to /bin/systemctl stop osa-dispatcher.service
Redirecting to /bin/systemctl stop httpd.service
Redirecting to /bin/systemctl stop tomcat.service
Redirecting to /bin/systemctl stop jabberd.service
Redirecting to /bin/systemctl stop postgresql.service
Done.
Starting spacewalk services...
Redirecting to /bin/systemctl start postgresql.service
Redirecting to /bin/systemctl start jabberd.service
Redirecting to /bin/systemctl start tomcat.service
Waiting for tomcat to be ready ...
Redirecting to /bin/systemctl start httpd.service
Redirecting to /bin/systemctl start osa-dispatcher.service
Job for osa-dispatcher.service failed because the control process exited with error code. See "systemctl status osa-dispatcher.service" and "journalctl -xe" for details.
Redirecting to /bin/systemctl start rhn-search.service
Starting cobblerd (via systemctl):                         [  OK  ]
Redirecting to /bin/systemctl start taskomatic.service
Done.
[root@si5148v yum.repos.d]#
```

Se connecter sur spacewalk et vérifier que tout va bien.

