---
description: Deploy containers from public Docker images with ease. Explore our guide on running applications using readily available Docker images.
---

# Run Container From a Public Docker Image

Before we create our own Docker image, let's see how we can use publicly available Docker images to run containers. 

`Docker Hub` is one of the public registries you can use to download the official images for many popular services such as `Nginx`, `MySQL`, `MongoDB`, `Redis`, `Node.js`, `Java`, `Python`, etc.. These images are created either by the companies that developed these services or by the Docker community. 

!!! note
    Docker Hub hosts not only the official images but also an extensive collection of public images contributed by developers, serving diverse purposes and use cases.


## List Images and Containers

Execute the following `Docker CLI` commands to view images and containers on your host.

List Docker Images:
```
docker images
```

List Docker Containers:
```
docker ps -a
```

You can also use the Docker Desktop GUI to view the images and containers on your host.

At this point, you will see that you don't have any images or containers. 


## Run an Nginx Container

You can go to [Docker Hub Explore Page]{:target="_blank"} and search for the images that you want to use.

Execute the following command to run an Nginx container:

```
docker run nginx
```

!!! success "Here's what happens when you run the above command"

    1. The Docker Daemon recognizes that no tag is provided for the image, so it will use the `latest` tag.
    2. The Docker Daemon checks whether the `nginx:latest` image is present on the host where the command was executed.
    3. If the `nginx:latest` image is not present on the host, it will first pull the `nginx:latest` image from Docker Hub, which is the `default` registry.
    4. Now that the `nginx:latest` image is available on the host, the Daemon will use this image to run the container.

The command above runs the container in the foreground and stream the container logs. You can press ++ctrl+c++ to exit but the container will stop.

To avoid that, you can use `--detach` or `-d` option to run the container in the background.

```
docker run -d nginx
```

!!! tip "Using specific image tag"
    If you prefer to use a specific tag of the image instead of the `latest` tag to run the container, you can use the following syntax to do so.

    ```
    docker run -d <image-name>:<image-tag>
    ```

!!! tip "Assigning container name"
    A random name is assigned to the container if you don't specify a name. You can use `--name` option to assign a desired name to the container.

    ```
    docker run -d --name my-nginx-container nginx:latest
    ```

## List Containers

List only running containers:
```
docker ps
```

You can use `--all` or `-a` option to show all containers, `Running` or `Stopped`:
```
docker ps -a
```

You can use `--quiet` or `-q` option to display only the container IDs:
```
# List running container IDs
docker ps -q

# List all container IDs
docker ps -a -q
```

## Stop a Container

```
# Stop container using ID
docker stop <container-id>

{OR}

# Stop container using name
docker stop <container-name>
```

!!! note
    Stopping a container doesn't delete the container; it simply halts the container's execution. All the data and configuration associated with that container remain intact.


## Start a Stopped Container

```
# Start container using ID
docker start <container-id>

{OR}

# Start container using name
docker start <container-name>
```

!!! note
    Deleting a container permanently removes it from your system. This action deletes all the data and configuration associated with that container.


## Execute a Command in the Running Container

The `docker exec` command runs a new command in a running container. The command runs in the default directory of the container.

1. Execute a command in a running container without starting a shell session:

    ```
    # List files and directories
    docker exec -it <container-name/container-id> ls
    ```

2. Start a shell session in the container:

    ```
    docker exec -it <container-name/container-id> sh
    {OR}
    docker exec -it <container-name/container-id> bash
    ```

    !!! note "Difference between `sh` and `bash`"
        `bash` and `sh` are two different shells in the Unix operating system. `bash` is an extended version of `sh` with additional features and improved syntax.

    Once the shell session is started, you can interact with the container just like you would with your own machine. You can run shell commands like `ls`, `pwd`, or any other desired commands.


    In our case since the container is running Nginx, we can request the default nginx page as follows:

    ```
    curl localhost
    ```

    You can also view the nginx configuration files and default Html page that is served:

    ```
    # View nginx configuration file
    cd /etc/nginx/conf.d
    cat default.conf

    # View the content of the default Html page that Nginx serves
    cat /usr/share/nginx/html/index.html
    ```


## Port Mapping

In the previous section we learned how to access the application from within the container. But, how do you access the application running inside the container from outside?

You can use `Port Mapping` to make the processes inside the container available from the outside.

```
docker run -d --name nginx -p 81:80 nginx:latest
```

In the above command, we are creating the `nginx` container in detached mode and using the `-p` option to map port `80` of the container, where Nginx is running, to port `81` on the host.

Once the container is up and running, you can open any browser on your local machine and access the Nginx application running in the container by visiting `localhost:81`.


## View Container Logs

```
docker logs -f <container-id/container-name>
```

!!! note
    The `-f` flag in the `docker logs` command stands for "follow". It allows you to continuously stream the logs of the specified Docker container in real-time.

## Delete a Container

```
# Delete container using ID
docker rm <container-id>

{OR}

# Delete container using name
docker rm <container-name>
```

!!! note
    A container needs to be stopped before it can be deleted. You can't delete a running container.

## Delete All Stopped Containers

What if you have multiple stopped containers and you want to delete all of them in a single command?

You can use the following command to delete all stopped containers:
```
docker container prune
```


## Delete an Image

You can use the following commands to delete Docker images:
```
# Delete image using image ID
docker rmi <image-id>

{OR}

# Delete image using image name and tag
docker rmi <image-name>:<image-tag>
```

## Delete All Unused Images

You can use the following command to remove all images on your Docker host that are not associated with a container:
```
docker image prune -a
```

## Clean Up

You can use the following command to perform a comprehensive cleanup of your Docker system. It removes not only unused containers and images but also associated volumes:
```
docker system prune -a --volumes
```

!!! warning
    This command is an effective way to reclaim disk space and clean up your Docker environment, but it should be used with caution because it permanently deletes containers, images, networks, and volumes that are not actively in use. Make sure you understand the potential consequences before running it in a production environment.


<!-- Hyperlinks -->
[Docker Hub Explore Page]: https://hub.docker.com/search?q=