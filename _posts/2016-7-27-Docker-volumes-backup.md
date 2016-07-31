---

layout: post
title: 'Docker volumes backup'
date: '2016-7-27'
header-img: "img/post-bg-web.jpg"
tags:
     - Docker
     - backup
author: 'Bill Quan'
---



## Docker volumes backup

Introduction of docker volumes is [here](http://docs.docker.com/engine/tutorials/dockervolumes/ "docker volume").

Learn to use docker new volume apis first.

```shell
docker volume create --name data
docker run -d -v data:/container/path/for/volume container_image my_command
```

If you create a container with a `-v volume_name:/container/fs/path` docker will automatically create a named volume for you that can:

1. Be listed through the `docker volume ls`
2. Be identified through the `docker volume inspect volume_name`
3. Backed up as a normal dir
4. Backed up as before through a `--volumes-from` connection



### Backup

```
docker run --rm --volumes-from DATA -v $(pwd):/backup busybox tar cvf /backup/backup.tar /data
```

- --rm: remove the container when it exits
- --volumes-from DATA: attach to the volumes shared by the DATA container
- -v $(pwd):/backup: bind mount the current directory into the container; to write the tar file to
- busybox: a small simpler image - good for quick maintenance
- tar cvf /backup/backup.tar /data: creates an uncompressed tar file of all the files in the /data directory



### RESTORE:

```
# create a new data container
$ sudo docker run -v /data -name DATA2 busybox true
# untar the backup files into the new container᾿s data volume
$ sudo docker run --rm --volumes-from DATA2 -v $(pwd):/backup busybox tar xvf /backup/backup.tar
data/
data/sven.txt
# compare to the original container
$ sudo docker run --rm --volumes-from DATA -v $(pwd):/backup busybox ls /data
sven.txt
```





