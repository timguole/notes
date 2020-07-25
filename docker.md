# Docker Note

查看各种信息

```shell
docker image ls
docker container ls # only list running containers
docker container ls --all # view all containers
docker container inspect container-name
```

运行一个新的容器

```shell
# basic form
docker container run image-name

# override default command
docker container run image-name cmd arg1 arg2 ...

# common options:
# --name <name>
# -t (allocate a tty)
#-i (interactive)
#-d (daemon-mode)
# -v </path/in/container> (bind a volume)
```

在容器中运行一个命令

```shell
docker container exec container-name cmd arg1 arg2 ...

# run an interactive shell
docker container exec -it container-name bash
```

启动和停止一个容器

```shell
docker container start container-name
docker container stop container-name
```

