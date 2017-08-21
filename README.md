# README

TODO: Create introduction... 


```
# At the time of creation, the root password for MySQL is: Ph0en!x7 
```

Before running 'vagrant up' be sure to review/complete the following:


1.  Adjust memory settings in Vagrantfile, if needed (see mem variable or use Environment Setting [provide example])

```
mem = mem / 1024 / 4
vb.memory = ENV['VAGRANT_MEMORY'] || mem

# From your terminal you can override memory defined in Vagrant file using:
export VAGRANT_MEMORY=xxxxx
```


2.  Edit synced folder line in Vagrant file

```
config.vm.synced_folder "/Users/jfhogarty/Documents/Programming/Elixir/DEVBOX7A-B", "/vagrant"
```


3.  Edit the provisioning/set_mysql_root_pw.sh file and adjust NEW_PASS if you do not want to use the provided password.


```
# NOTE:  I noticed an issue when provisioning once where the mysql pw script didn't work properly.
#        It looked like maybe there were some special characters in the generated password.
#        Keep an eye out for this!
```


# Post Vagrant Up

After you have executed 'vagrant up' we want to log in and verify our tools have
been installed:

```
➜  devbox7-b vagrant ssh
----------------------------------------------------------------
  CentOS Linux 7.2 - API DEV BOX              built 2017-07-14
----------------------------------------------------------------
[vagrant@devbox7-b ~]$

[vagrant@devbox7-b ~]$ rvm -v
rvm 1.29.2 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io/]
[vagrant@devbox7-b ~]$

[vagrant@devbox7-b ~]$ ruby -v
ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-linux]
[vagrant@devbox7-b ~]$

[vagrant@devbox7-b ~]$ node -v
v7.10.1
[vagrant@devbox7-b ~]$

[vagrant@devbox7-b ~]$ mysql --version
mysql  Ver 14.14 Distrib 5.7.18, for Linux (x86_64) using  EditLine wrapper
[vagrant@devbox7-b ~]$

[vagrant@devbox7-b ~]$ sudo systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2017-07-17 15:58:22 UTC; 2min 13s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 906 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 875 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 918 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─918 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Jul 17 15:58:21 devbox7-b systemd[1]: Starting MySQL Server...
Jul 17 15:58:22 devbox7-b systemd[1]: Started MySQL Server.
[vagrant@devbox7-b ~]$

[vagrant@devbox7-b ~]$ elixir -v
Erlang/OTP 20 [erts-9.0] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:10] [hipe] [kernel-poll:false]

Elixir 1.4.5
[vagrant@devbox7-b ~]$
[vagrant@devbox7-b ~]$ erl -v
Erlang/OTP 20 [erts-9.0] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V9.0  (abort with ^G)
1>
BREAK: (a)bort (c)ontinue (p)roc info (i)nfo (l)oaded
       (v)ersion (k)ill (D)b-tables (d)istribution
^C[vagrant@devbox7-b ~]$


[vagrant@devbox7-b ~]$ mix -h
mix                   # Runs the default task (current: "mix run")
mix app.start         # Starts all registered apps

{--snip--}

# We will focus on confirming the next two are present for Phoenix
mix phoenix.new       # Creates a new Phoenix v1.2.4 application
mix phx.new           # Creates a new Phoenix v1.3.0-rc.2 application using the experimental generators

{--snip--}

mix test              # Runs a project's tests
mix xref              # Performs cross reference checks
iex -S mix            # Starts IEx and runs the default task
[vagrant@devbox7-b ~]$ 


[vagrant@devbox7-b ~]$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.18 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Now that we have confirmed that our basic setup is up and running successfully,
the next step is to enable replication between the master (devbox7-a) and the
slave (devbox7-b).


## Setting Up MySQL Replication (Slave)

Master (devbox7-a): 192.168.70.10
Slave  (devbox7-b): 192.168.70.11

Replication User:   replicant 
Replication Pass:   b0rgs#Human$
Replication Host:   devbox7-b.localdomain 


Log in to mysql on the Slave (devbox7-b) VM as the root user.


```
mysql -u root -p

# Next, stop the slave:
STOP SLAVE;

# Example:
mysql> STOP SLAVE;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql>
```


Now we will setup the slave (devbox7-b) to talk to the master (devbox7-a).


```
# NOTE:  Be sure to use the appropriate settings if different from below.

mysql> CHANGE MASTER TO
    -> MASTER_HOST='192.168.70.10',
    -> MASTER_USER='replicant',
    -> MASTER_PASSWORD='b0rgs#Human$',
    -> MASTER_LOG_FILE='mysql-bin.000001',
    -> MASTER_LOG_POS=1185;

# Now re-start the slave:
START SLAVE;
```





## EXTRA - Confirming MySQL Replication is working

```
# On the Master (devbox7-a)
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     1185 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> create database mytest;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mytest             |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql>



# On the Master (devbox7-a)
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mytest             |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql>

# As we can see, the 'mytest' database was replicated from the Master to the Slave
```

