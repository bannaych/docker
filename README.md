# MSSQL Server on Docker with persistent volumes

Docker and containers have become a integral part of virtualisation, being able to spin up containers
almost instantly have transformed the way be build, develop and manage applications.

Many of us have been using some sort of virtualisation for decades as this is not a new concept, Containers have gathered great momentem and most companies have either adopted conatainers or seriously thinking about it because of the speed, simplicity and automation they provide.

**So what use containers ?**

- Decouple application runtime from infrastrcuture
- Run instantly
- Process and user isolation for application multitenancy
- Key building blocks for microservices

**But what happens when you delete the container ?**

The main issue is when delete a container, you delete the writeable layer and any data that was in there, hence there is no data persistence. Docker volumes provide us a way to decouple your application from its state to the point where you can simply throw away the container and replace it with a new container image start up your application and point it to your data

Depending on your application, this may or may not be required, but for most DBAs they want their data in tact. For them Data is GOLD.. 

In this blog I will show you how we can use the Pure Storage Docker plugin to create
persistent storage volume which we can then use to create MSSQL server containers on in a linux environment, allowing is to decouple the data from the application, this gives us the ability to delete containers, snapshot docker volumes and create instant clones of the containers for rapid agile development.

# Environment

For this demo I'm using a linux host to run my docker SQL Server containers on, you can just as easily run Docker on windows to get the same result.

|Role|FQDN|IP|OS|RAM|CPU|Pure Docker plugin
|----|----|----|----|----|----|----|
|Docker Virtual Machine|docker.localdomain|192.168.111.198|OEL 7|5G|5|3.0

**Note:** Latest version on plugin is 3.8

# Installation 

* Install the Plugin
```
# docker plugin install purestorage/docker-plugin:3.0 --alias pure
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

* Create the Docker volume to be used for MSSQL

```
docker volume create --driver=pure:latest --opt size=20GB --name=sqldata1 --label=sqldata1
sqldata1
```
* Check the array to make sure volume has been created


* Check to make sure the Linx OS can see the new volume
```
#fdisk -l

Disk /dev/mapper/3624a9370a21265762db64ece0005168a: 20.0 GB, 20000000000 bytes, 39062500 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 4194304 bytes
```

* Create the MSSQL 2019 container called sql3 using the volume we just created - **sqldata1**
```
# docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=P@ssw0rd" -p 1433:1433 --volume-driver pure --volume sqldata1:/var/opt/mssql --name sql3 -d mcr.microsoft.com/mssql/server:2019-CU3-ubuntu-18.04
729c2574a03498c3536e870e4ebcdd74731dfdd16745352e3af04e9e0d9dfd82
```
* Check to makre sure the docker container is created
```
# docker ps
CONTAINER ID        IMAGE                                                  COMMAND                  CREATED             STATUS              PORTS                    NAMES
729c2574a034        mcr.microsoft.com/mssql/server:2019-CU3-ubuntu-18.04   "/opt/mssql/bin/perm…"   20 seconds ago      Up 14 seconds       0.0.0.0:1433->1433/tcp   sql3
```
* Log into the container and confirm the volume has been mounted
```
# docker exec -it 729c2574a034 bash
mssql@729c2574a034:/$ df -h
Filesystem                                     Size  Used Avail Use% Mounted on
overlay                                         66G   47G   19G  71% /
tmpfs                                           64M     0   64M   0% /dev
tmpfs                                          2.8G     0  2.8G   0% /sys/fs/cgroup
shm                                             64M     0   64M   0% /dev/shm
/dev/mapper/ol-root                             66G   47G   19G  71% /etc/hosts
/dev/mapper/3624a9370a21265762db64ece0005168a   19G  153M   19G   1% /var/opt/mssql
tmpfs                                          2.8G     0  2.8G   0% /proc/acpi
tmpfs                                          2.8G     0  2.8G   0% /proc/scsi
tmpfs                                          2.8G     0  2.8G   0% /sys/firmware
```
* Log into the new SQL Server container using SSMS ( SqL Server Management Studio )

* Connect the database running on the Docker host **192.168.111.198** using sa authentication

* Create a new database called **sqldemo**

* Confirm the database is running on the new docker volume


