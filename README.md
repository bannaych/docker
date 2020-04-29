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
persistent storage volume which we can then use to create MSSQL server containers on, allowing is to decouple the data from the application, this gives us the ability to delete containers, snapshot docker volumes and create instant clones of the containers for rapid agile development.






