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



