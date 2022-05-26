# Docker

**Create a new container**
```
              1     2      3    4
$ docker run -it ubuntu:latest bash
```
1) Interactive terminal
2) Image name
3) Image flag (optional - if not specified default is latest)
4) What process to run ("what would you like to do?") (Optional - seems to give you this with just the -it option)

To exit a shell in a container, either press Ctrl + D, or type `exit`.

**See running images**
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


