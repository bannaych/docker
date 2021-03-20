
# Install Portainer

Portainer runs as container, so installing it is just like installing any other container on Docker.

* Lets create a persistent volume to run Portainer on

```
# docker volume create port_vol
port_vol

```

* Lets create the Portainer container

```
# docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always \
-v /var/run/docker.sock:/var/run/docker.sock -v port_vol:/data \
portainer/portainer-ce

```

* Lets confirm the new Portainer container is running

```
docker ps
CONTAINER ID        IMAGE                                                        COMMAND                  CREATED             STATUS                 PORTS                                            NAMES
b9da841acf8b        portainer/portainer-ce                                       "/portainer"             2 hours ago         Up 2 hours             0.0.0.0:8000->8000/tcp, 0.0.0.0:9000->9000/tcp   portainer
```
