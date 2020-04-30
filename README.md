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


So now we really need to confirm that what I have created is really a persistent volumem, and the only way to do 
that is to delete the container, create a new container with a different name and use the same database volume

* First lets stop the sql3 container
```
# docker stop sql3
sql3
```
* Now lets remove the container
```
# docker rm sql3
sql3
```
* Confirm the container has been deleted
```
docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
* Create a new container called  **sql4** using the same sqldata1 volume
```
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=P@ssw0rd" -p 1433:1433 --volume-driver pure --volume sqldata1:/var/opt/mssql --name sql4 -d mcr.microsoft.com/mssql/server:2019-CU3-ubuntu-18.04
5e8af40275b8c0602252f9643ce578764f3ab0e7fcd24088cf5db3d663b0a606
```

* Confirm the container has been created
```
docker ps
CONTAINER ID        IMAGE                                                  COMMAND                  CREATED             STATUS              PORTS                    NAMES
5e8af40275b8        mcr.microsoft.com/mssql/server:2019-CU3-ubuntu-18.04   "/opt/mssql/bin/perm…"   9 seconds ago       Up 3 seconds        0.0.0.0:1433->1433/tcp   sql4
```

* Now lets log into the container, check the volume has been mounted and the data is still there
```
[root@docker ~]# docker exec -it 5e8af40275b8 bash
df -k
Filesystem                                    1K-blocks     Used Available Use% Mounted on
overlay                                        68328860 48447740  19881120  71% /
tmpfs                                             65536        0     65536   0% /dev
tmpfs                                           2922740        0   2922740   0% /sys/fs/cgroup
shm                                               65536        0     65536   0% /dev/shm
/dev/mapper/ol-root                            68328860 48447740  19881120  71% /etc/hosts
/dev/mapper/3624a9370a21265762db64ece0005168a  19521008   176588  19344420   1% /var/opt/mssql
tmpfs                                           2922740        0   2922740   0% /proc/acpi
tmpfs                                           2922740        0   2922740   0% /proc/scsi
tmpfs                                           2922740        0   2922740   0% /sys/firmware

cd data
mssql@5e8af40275b8:/var/opt/mssql/data$ ls
Entropy.bin  mastlog.ldf  model_msdbdata.mdf  model_replicatedmaster.ldf  modellog.ldf	msdblog.ldf  sqldemo_log.ldf  tempdb2.ndf  tempdb4.ndf	tempdb6.ndf  templog.ldf
master.mdf   model.mdf	  model_msdblog.ldf   model_replicatedmaster.mdf  msdbdata.mdf	sqldemo.mdf  tempdb.mdf       tempdb3.ndf  tempdb5.ndf	tempdb7.ndf
mssql@5e8af40275b8:/var/opt/mssql/data$ ls -l
total 138756
-rw-r-----. 1 mssql root      256 Apr 30 01:04 Entropy.bin
-rw-r-----. 1 mssql root  4653056 Apr 30 02:14 master.mdf
-rw-r-----. 1 mssql root  2359296 Apr 30 02:14 mastlog.ldf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 model.mdf
-rw-r-----. 1 mssql root 14090240 Apr 30 02:14 model_msdbdata.mdf
-rw-r-----. 1 mssql root   524288 Apr 30 02:14 model_msdblog.ldf
-rw-r-----. 1 mssql root   524288 Apr 30 02:14 model_replicatedmaster.ldf
-rw-r-----. 1 mssql root  4653056 Apr 30 02:14 model_replicatedmaster.mdf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 modellog.ldf
-rw-r-----. 1 mssql root 14090240 Apr 30 02:14 msdbdata.mdf
-rw-r-----. 1 mssql root   524288 Apr 30 02:14 msdblog.ldf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 sqldemo.mdf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 sqldemo_log.ldf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 tempdb.mdf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 tempdb2.ndf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 tempdb3.ndf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 tempdb4.ndf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 tempdb5.ndf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 tempdb6.ndf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 tempdb7.ndf
-rw-r-----. 1 mssql root  8388608 Apr 30 02:14 templog.ldf
```
As we can see all the data from the sql3 database is still there, including the **sqldemo** database. But just to confirm
let log back into SSMS and confirm we can access the datbase



