# Instructions

* To build the Dockerfile:
```shell
docker build -t <image-name> .
```

* To run the container:

```shell
docker run -d -p 8080:80 --name <container-name> <image-name>
```