# Docker

## Docker Images

```shell
docker images # list of docker images


```

## Docker Container

```shell
docker ps # see the list of all running containers

docker run -p bind-port:container-port <image name> # run a docker image
e.g.: docker run -p 7777:80 nginx

# set custom name
docker run --name <Name> -p bind-port:container-port <image name>

# run in detached mode (use -d)
docker run -d --name <Name> -p bind-port:container-port <image name>

# stop a container
docker stop <container id or name>

```

## Docker pull

```shell
docker pull <image name> 

docker pull <image name>:<build version> # pull a specific version
# default build: latest

```

## Docker exec

```shell
docker exec -it <container name> <command>
e.g.: docker exec -it my-nginx bash
```

## Docker logs

```shell
docker logs <container name>
docker logs -f <container name> # real time
docker logs -n <number> <container name> # watch last n number of logs
docker logs --since <timestamp> <container name> # logs since a specific timestamp 
docker logs --until <timestamp> <container name> # logs until a specific timestamp
docker logs --details <container name> # show extra details
docker logs --timestamps <container name> # show timestamps 

```

## Docker Tag

```shell
docker tag <image-name>:<current-tag> <image-name>:<new-tag>
```

## Docker Network

```shell
docker network ls # list all networks
docker network create <network name> # create a new network
docker network connect <network name> <container name> # connect a container to a network
docker network disconnect <network name> <container name> # disconnect a container from a network

docker run -d --network <network name> -p bind-port:container-port <image-name> # run a container in a specific network

```
