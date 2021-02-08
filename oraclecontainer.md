# Oracle 19c cloning using the Pure Storage Docker Plugin

Docker and containers have become a integral part of virtualisation, being able to spin up containers
almost instantly have transformed the way be build, develop and manage applications.

In this blog I will show you how we can rapidly create Oracle 19c container clones using the Pure Storage Docker plugin
and the new Oracle 19c container database image on the Oracle Container Registry

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

