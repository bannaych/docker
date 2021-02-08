# Oracle 19c cloning using the Pure Storage Docker Plugin

Docker and containers have become a integral part of virtualisation, being able to spin up containers
almost instantly have transformed the way be build, develop and manage applications.

In this blog I will show you how we can rapidly create Oracle 19c container clones using the Pure Storage Docker plugin
with persistant volumes, and the new Oracle 19c container database image on the Oracle Container Registry

https://container-registry.oracle.com/pls/apex/f?p=113:4:2886566670416# 

Installing the Pure Plugin
```
docker plugin install purestorage/docker-plugin:v3.10 --alias pure --grant-all-permissions
```

* Enter the Array Configuration in the /etc/pure-docker-plugin/pure.json file
```
 {
     "FlashArrays":[
         {
             "MgmtEndPoint":“192.168.111.130",
             "APIToken":" 476d58bc-3c91-0d10-1b9e-0f31058c4621"
         }
     ]
 }
```


# Check the plugin version
```
12:26:36 root@docker ~ → docker plugin ls
ID                  NAME                DESCRIPTION                      ENABLED
a0b3cc51480d        pure:latest         Pure Storage plugin for Docker   true
```

# Login to the Oracle Container Registry
```
12:23:47 root@docker ~ → docker login container-registry.oracle.com
Username (cbannayan@purestorage.com):
Password:
Login Succeeded

```
# Create the Persistent Volume using the Pure docker plugin
```
12:29:55 root@docker ~ → docker volume create --driver=pure:latest --opt size=5G --name=ora19c --label=orcl
ora19c

12:30:58 root@docker ~ → docker volume ls|grep ora19
pure:latest         ora19c

```
# Confirm the volume has been created on the FlashArray
```
12:32:36 root@docker ~ → ssh pureuser@192.168.111.130
pureuser@192.168.111.130's password:
Last login: Sun Jan 24 16:29:52 2021 from 192.168.112.1

Mon Feb 08 12:51:00 2021
Welcome pureuser. This is Purity Version 6.0.4 on FlashArray Pure-SYD-m20-1
**********************************************************************
WARNING: The user pureuser is still configured with the default password.
         Please change the password now: pureadmin setattr pureuser --password
**********************************************************************

pureuser@Pure-SYD-m20-1> purevol list *ora19c*
Name                  Size  Source  Created                   Serial
docker-docker-ora19c  5G    -       2021-02-08 12:48:50 AEDT  A21265762DB64ECE000E1FB0

```

# Check to see if any containers are running
```
12:36:44 root@docker ~ → docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
# Check the Docker Oracle Image
```
12:46:58 root@docker ~ → docker image ls
REPOSITORY                                          TAG                     IMAGE ID            CREATED             SIZE
container-registry.oracle.com/database/enterprise   19.3.0.0                6ee1b2e4403f        2 months ago        7.87GB

```

# Create the Oracle 19c Container
```
12:48:20 root@docker ~ → docker run -d --name zz-ora19c --mount source=ora19c,target=/opt/oracle/oradata container-registry.oracle.com/database/enterprise:19.3.0.0
155481d8de4e37eb7033aead74267b7b5220c51f1415e1912b260d9d0d681ce4

12:49:06 root@docker ~ → docker ps
CONTAINER ID        IMAGE                                                        COMMAND                  CREATED             STATUS                             PORTS               NAMES
155481d8de4e        container-registry.oracle.com/database/enterprise:19.3.0.0   "/bin/sh -c 'exec $O…"   27 seconds ago      Up 22 seconds (health: starting)                       zz-ora19c

```
# Log into the container and confirm the database is running on the Volume
```
12:49:25 root@docker ~ → docker exec -it 155481d8de4e bash
[oracle@155481d8de4e ~]$
[oracle@155481d8de4e ~]$ ps -ef|grep pmon
oracle     506     1  0 01:49 ?        00:00:00 ora_pmon_ORCLCDB
oracle     647   601  0 01:51 pts/0    00:00:00 grep --color=auto pmon
[oracle@155481d8de4e ~]$
[oracle@155481d8de4e ~]$ df -k
Filesystem                                    1K-blocks     Used Available Use% Mounted on
overlay                                        68328860 56622648  11706212  83% /
tmpfs                                             65536        0     65536   0% /dev
tmpfs                                           2922740        0   2922740   0% /sys/fs/cgroup
shm                                               65536        0     65536   0% /dev/shm
/dev/mapper/ol-root                            68328860 56622648  11706212  83% /etc/hosts
/dev/mapper/3624a9370a21265762db64ece000e1fb0   5232640   266060   4966580   6% /opt/oracle/oradata
tmpfs                                           2922740        0   2922740   0% /proc/acpi
tmpfs                                           2922740        0   2922740   0% /proc/scsi
tmpfs                                           2922740        0   2922740   0% /sys/firmware

```
We can see that our persistent volume with the serial number ending in fb0 is mount on /opt/oracle/oradata, the same service number as the volume ora19c

# Log into the Oracle Database
```
[oracle@155481d8de4e ~]$ sqlplus / as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Mon Feb 8 01:52:19 2021
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL> select name from v$database;

NAME
---------
ORCLCDB

SQL> select name from v$datafile;

NAME
--------------------------------------------------------------------------------
/opt/oracle/oradata/ORCLCDB/system01.dbf
/opt/oracle/oradata/ORCLCDB/sysaux01.dbf
/opt/oracle/oradata/ORCLCDB/undotbs01.dbf
/opt/oracle/oradata/ORCLCDB/users01.dbf

```

# Create a table in the database
```
SQL> create table table1 as select * from dba_users;

Table created.

SQL> select count(8) from table1;

  COUNT(8)
----------
        36

SQL>
```

Now that we have our first database container running, lets use this as our GOLD image to create clones from, Using the Docker
and Pure Plugin we can create rapid clones of our containers in seconds. First lets create a clone of our database persistant volume ora19c

# Create the clone volume 
```
12:57:09 root@docker ~ → docker volume create --driver=pure --name ora19clone -o source=orac19c
ora19clone
12:57:31 root@docker ~ →
12:57:53 root@docker ~ → docker volume ls |grep ora19clone
pure:latest         ora19clone

```
# Lets confirm the clone has been created from the source
```
pureuser@Pure-SYD-m20-1>
pureuser@Pure-SYD-m20-1> purevol list *ora19*
Name                      Size  Source                Created                   Serial
docker-docker-ora19c      5G    -                     2021-02-08 12:48:50 AEDT  A21265762DB64ECE000E1FB0
docker-docker-ora19clone  5G    docker-docker-ora19c  2021-02-08 14:16:54 AEDT  A21265762DB64ECE000E2073

As you can see, the ora19clone is the same 5G in size and its source is ora19c 

```

# Create the clone container using the clone volume
```
02:02:21 root@docker ~ → docker run -d --name zz-ora19c-clone --volume-driver pure -v ora19clone:/opt/oracle/oradata container-registry.oracle.com/database/enterprise:19.3.0.0
344e25d5c357effdbe6b8ddf901f8e36d90faac5430e12cbaa41bfafeec0b075
02:02:35 root@docker ~ →

```
# log into the container and confirm Oracle has started and cloned
```
02:03:20 root@docker ~ → docker ps
CONTAINER ID        IMAGE                                                        COMMAND                  CREATED             STATUS                             PORTS               NAMES
344e25d5c357        container-registry.oracle.com/database/enterprise:19.3.0.0   "/bin/sh -c 'exec $O…"   54 seconds ago      Up 48 seconds (health: starting)                       zz-ora19c-clone
b86a816fe327        container-registry.oracle.com/database/enterprise:19.3.0.0   "/bin/sh -c 'exec $O…"   11 minutes ago      Up 11 minutes (healthy)                                zz-ora19c

02:03:23 root@docker ~ → docker exec -it 344e25d5c357 bash
[oracle@344e25d5c357 ~]$ df -k
Filesystem                                    1K-blocks     Used Available Use% Mounted on
overlay                                        68328860 56338860  11990000  83% /
tmpfs                                             65536        0     65536   0% /dev
tmpfs                                           2922740        0   2922740   0% /sys/fs/cgroup
shm                                               65536        0     65536   0% /dev/shm
/dev/mapper/ol-root                            68328860 56338860  11990000  83% /etc/hosts
/dev/mapper/3624a9370a21265762db64ece000e2073   5232640  3895484   1337156  75% /opt/oracle/oradata
tmpfs                                           2922740        0   2922740   0% /proc/acpi
tmpfs                                           2922740        0   2922740   0% /proc/scsi
tmpfs                                           2922740        0   2922740   0% /sys/firmware

SQL> select count(*) from table1;

  COUNT(*)
----------
        36

SQL>
```
The Oracle Container has started without doing a new install and the table "table1" we created is there.


In this blog we have seen how using the Pure Storage Docker Plugin can easily and quickly deploy and clone Oracle 19c database containers using persistent storage. 
