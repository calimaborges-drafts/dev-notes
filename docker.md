## Start interactive shell for Ubuntu

```bash
$ docker run -i -t ubuntu /bin/bash
```

## Start mapping port

```bash
$ docker run -p <host_port>:<guest_port> -i -t ubuntu /bin/bash
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

