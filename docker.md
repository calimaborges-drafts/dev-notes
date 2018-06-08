# Docker

## Build behind proxy

```bash
docker build -f app.dockerfile -t username/app --build-arg HTTP_PROXY=http://proxy.server:3128/ --build-arg HTTPS_PROXY=http://proxy.server:3128/ --build-arg npm_config_strict_ssl=false
```

```bash
docker-compose build --build-arg HTTP_PROXY=http://proxy.server:3128/ --build-arg HTTPS_PROXY=http://proxy.server:3128/
```


## Start interactive shell for Ubuntu

```bash
$ docker run -i -t ubuntu /bin/bash
```

## Mapping port

```bash
$ docker run -p 8080:80 -i -t ubuntu /bin/bash
```

Ao acessar a porta 8080 a máquina redirecionará para a porta 80 do docker.

## Mapping disk

```bash
docker run -v $PWD:/var/www -t -i web
```

## Commit modifications

```bash
docker commit <container> <some_name>
```

## View images

```bash
docker images
```

## Create docker machine

```bash
$ docker-machine create --driver virtualbox default
```
