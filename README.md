# Docker

## Docker Basic Commands

---

### Docker Images

```shell
docker images # list of docker images
```

### Docker Container

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

### Docker pull - push

```shell
docker pull <image name> 

docker pull <image name>:<build version> # pull a specific version
# default build: latest

docker push <registryname>/<username>/<imagename>:<tag>

```

### Docker exec

```shell
docker exec -it <container name> <command>
e.g.: docker exec -it my-nginx bash
```

### Docker logs

```shell
docker logs <container name>
docker logs -f <container name> # real time
docker logs -n <number> <container name> # watch last n number of logs
docker logs --since <timestamp> <container name> # logs since a specific timestamp 
docker logs --until <timestamp> <container name> # logs until a specific timestamp
docker logs --details <container name> # show extra details
docker logs --timestamps <container name> # show timestamps 

```

### Docker Tag

```shell
docker tag <image-name>:<current-tag> <image-name>:<new-tag>
```

### Docker Network

```shell
docker network ls # list all networks
docker network create <network name> # create a new network
docker network connect <network name> <container name> # connect a container to a network
docker network disconnect <network name> <container name> # disconnect a container from a network

docker run -d --network <network name> -p bind-port:container-port <image-name> # run a container in a specific network

```

# Docker Inspect

```shell
docker inspect <container/image/volume id or name> # inspect a container/image/volume
docker inspect --type=<type> --format='{{.<format>}}' <id/name> 
e.g.: docker inspect --type=container --format='{{json .Config.ExposedPorts}}' 453cefdfe984
```

---

## Dockerfile Instructions

## Dockerfile

Basic Dockerfile instructions:

- `FROM` \- base image.
- `ARG` \- build-time variable.
- `ENV` \- environment variable.
- `LABEL` \- image metadata.
- `WORKDIR` \- set working directory.
- `COPY` / `ADD` \- copy files into image (`COPY` preferred).
- `RUN` \- run commands during build.
- `EXPOSE` \- document container ports.
- `VOLUME` \- declare mount points.
- `USER` \- set runtime user.
- `ENTRYPOINT` \- fixed entrypoint for the container.
- `CMD` \- default command or arguments.
- `HEALTHCHECK` \- probe container health.
- `ONBUILD` \- trigger when used as a base image.
- `STOPSIGNAL` \- signal to stop container.
- `SHELL` \- change default shell for RUN commands.

Minimal example:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
ENV NODE_ENV=production
USER node
CMD ["node", "server.js"]
```

### Docker Copy Vs ADD

Differences between `COPY` and `ADD`:

1. `COPY` \- copies files and directories from the build context into the image. Simple, predictable, and preferred for most cases.
2. `ADD` \- does everything `COPY` does plus:
   1. can extract local compressed tar archives into the image.
   2. can download files from remote URLs and add the fetched content.
3. Recommendation \- use `COPY` unless you specifically need `ADD`'s extra behaviors (auto-extraction or URL fetch).
4. Best practices \- Always use `COPY` for moving your code/config. Only use `ADD` to extract tar.gz file.

### Docker `RUN`
`RUN` installs software, updates libraries, and creates folders.

#### Layers and optimization
Every single RUN line in your Dockerfile creates a new Image Layer.

**Too Many Layers:**
```dockerfile
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y git
RUN apt-get clean
```
Docker creates 4 separate layers.
```text
Layer A: The result of update.
Layer B: Layer A + python3.
Layer C: Layer B + git.
Layer D: Layer C + clean.
```


**Optimized Layers:**
```dockerfile
RUN apt-get update && \
    apt-get install -y python3 git && \
    apt-get clean
```




## Docker Volume

Docker supports two main ways to persist container data: named *volumes* and *bind mounts*.

1. Named volumes
    - Managed by Docker and stored on the Docker host (commonly at `/var/lib/docker/volumes`).
    - Lifecycle is decoupled from containers (volumes persist after container removal).
    - Portable between hosts only via explicit export/import or by using the same host.
    - Good for production data, backups, and sharing data between containers.

2. Bind mounts
    - Map a host filesystem path into the container (e.g., `/host/path:/container/path`).
    - Container sees the exact host files and permissions.
    - Useful for development, editing files on the host, or accessing host-specific data.
    - Less portable and can introduce host-specific permission/SELinux issues.

Common commands and examples:

```bash
# create a named volume
docker volume create mydata

# run container with named volume
docker run -d --name app -v mydata:/app/data myimage

# run container with bind mount
docker run -d --name app -v /home/user/app:/app myimage

# alternative using --mount (more explicit)
docker run -d --name app --mount source=mydata,target=/app/data myimage

# inspect, list, remove
docker volume ls
docker volume inspect mydata
docker volume rm mydata
docker volume prune