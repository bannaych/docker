# MSSQL Server on Docker with persistent volumes

Docker and containers have become a integral part of virtualisation, being able to spin up containers
almost instantly have transformed the way be build, develop and manage applications.

Many of us have been using some sort of virtualisation for decades as this is not a new concept, Containers have gathered great momentem and most companies have either adopted conatainers or seriously thinking about it.

So what use containers ?

- Decouple application runtime from infrastrcuture
- Run instantly
- Process and user isolation for application multitenancy
- Key building blocks for microservices

But what happens

In this blog I will show you how we can use the pure storage Docker plugin to create
persistent storage volume which we can then use to create MSSQL server containers on.


