# Docker

**Create a new container using run**
```
              1     2      3    4
$ docker run -it ubuntu:latest bash
```
1) Interactive terminal
2) Image name
3) Image flag (optional - if not specified default is latest)
4) What process to run ("what would you like to do?") (Optional - seems to give you this with just the -it option)

To exit a shell in a container, either press Ctrl + D, or type `exit`.


**Create and start container using compose up**
```
$ docker compose up
```

Builds, (re)creates, starts, and attaches to containers for a service.

Unless they are already running, this command also starts any linked services.


**See running images (containers)**
```
$ docker ps
```

See _all_ containers (even stopped)
```
$ docker ps -a
```

See _last_ container to exit
```
$ docker ps -l
```


**See images**
```
$ docker images
```


**Make a new image (based on another)**

First get the id or name of the image you wish to base the new image on. Then use it to commit a new image and then give it a name with `tag`: 
```
$ docker commit <id_or_name_of_other_container>
$ docker tag <sha_id_of_image_you_just_created> <new_name>
```
Or do both in one single step (easier):
```
$ docker commit <id_or_name_of_other_container> <new_name>
```

Basically, `docker run` takes an image to a container, and `docker commit` creates an image from a container.


**Run a few specific commands and then exit the container**
```
                    1     2   3     4   5     6
$ docker run -it ubuntu bash -c "sleep 3;echo all done"
```
1) Image you are basing the container on
2) Process you want to run
3) commands (?)
4) cmd1
5) ";" marks the end of a command
6) cmd2


**Run a container, but delete it after it exits**

Use the `--rm` flag (leave it running for 5 seconds, then exit and delete).
```
$ docker run --rm -it ubuntu sleep 5
```


**Run a container in the background**
```
$ docker run -d -it ubuntu bash
```

Then, in another terminal/tab, first get the name of the running container using `ps`, then attach to it.
```
$ docker ps
$ docker attach <name>
```

To exit you from the attached container, but leave the original running, press Ctrl +p, Ctrl + q.


**Restart a stopped container**

```
$ docker start -it `$(docker ps -q -l)`

or

$ docker start -it container_id
```


**Docker exec - Running more things in your container**

docker exec
- Starts another process in a running container
- Great for debugging and DB admin
- Can't be used to add parts, volumes etc( use run instead)

```
                          1          2
$ docker exec -it <container_name> bash 
```
1) The container you want to run additional things in
2) The process to execute in the container


**Looking at container output**

docker logs
- Keeps the output of containers
- View with `docker logs <container_name>
- Don't let the output get too large

As an example, first run a container as detached (`-d`) and make it run some non-existing commands (that will throw errors). Give it a name as well (`--name example`) so that you can easily refer to it:
```
$ docker run --name example -d ubuntu bash -c "lose /etc/password"
```
Then run `docker logs` to find what went wrong:
```
$ docker logs example
bash: lose: command not found
```


**Stopping and removing containers**

To stop a container (gracefully):
```
$ docker stop container_name
```

To kill a container (not gracefully - does not notify child processes and creates zombie processes)
```
$ docker kill container_name
```

To remove a (single) container
```
$ docker rm container_name
```

To remove an image
```
$ docker rmi image_id

or

$docker rmi image_name
```

To remove all containers, images, networks etc in a directory/created by up
```
$ docker compose down
```

**No space left on device error**

If you try to `up` your container but receive this type of message:
```
Error response from daemon: failed to mkdir /var/lib/docker/volumes/a3d0dd125fe1e9399c28381c431b7becfc2212eb03762dc5ad2e1f63e9e602f0/_data/is-regexp: mkdir /var/lib/docker/volumes/a3d0dd125fe1e9399c28381c431b7becfc2212eb03762dc5ad2e1f63e9e602f0/_data/is-regexp: no space left on device
```

Then try to run this:
```
docker system prune
```

And click `y` when asked if you want to continue.


