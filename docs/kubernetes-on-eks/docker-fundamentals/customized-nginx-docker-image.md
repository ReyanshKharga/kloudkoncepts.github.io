---
description: Craft your own tailored Nginx Docker image with our step-by-step guide. Learn the art of customization for Nginx container deployment.
---

# Create a Customized Nginx Docker Image

Let's see how you can create your own Docker image based on your requirements. In this tutorial, we will create a customized version of the Nginx image that serves an HTML page we want.

## Step 1: Create HTML File to Be Served by Nginx

By default, nginx serves the `index.html` file present in `/usr/share/nginx/html` directory. You can verify this by checking the content of `/etc/nginx/conf.d/default.conf`.

We'll replace the default `index.html` with our customized `index.html` file.

Let's create the customized `index.html` file as follows:

=== ":octicons-file-code-16: `index.html`"

    ```html linenums="1"
    <!doctype html>
    <html>
        <head>
            <title>Nginx</title>
        </head>
        <body>
            <h2>Hello from Nginx Container</h2>
        </body>
    </html>
    ```

## Step 2: Create Dockerfile

A `Dockerfile` is like a step-by-step instruction manual for creating a `Docker` image. It's a plain text file that contains a series of commands and settings that define how to construct a container image.

Create the `Dockerfile` as follows:

=== ":octicons-file-code-16: `Dockerfile`"

    ```Dockerfile linenums="1"
    FROM nginx:latest
    COPY ./index.html /usr/share/nginx/html/index.html
    ```

!!! question "What does this `Dockerfile` do?"
    1. It specifies the base image as the `latest` version of the official Nginx image.
    2. It copies the local `index.html` file into the Nginx container to replace the default one.

!!! note
    A "base image" is the initial image used as a starting point when creating a custom Docker image.

Here's what your folder structure should look like:

```
|-- my-folder
│   |-- index.html
│   |-- Dockerfile
```

## Step 3: Build the Image

Now that we have the `Dockerfile` and custom `index.html` file ready, we can use the `docker build` command to create the Docker image as follows:

```
# Command template
docker build -t <image-name>:<image-tag> <path-to-Dockerfile>

# Actual command
docker build -t my-nginx-image:v1 .
```

!!! question "What does the above command do?"
    1. It creates an image from the `Dockerfile` we provide
    2. It assigns the image a tag of `v1`

## Step 4: List Images

List Docker images to verify if the image we built is present:

```
docker images
```

You should see the newly built image in the list.

## Step 5: Run Container From the Image

Run a container from the newly created image as follows:

```
docker run -d --name my-nginx-container -p 81:80 my-nginx-image:v1
```

!!! question "What does the above command do?"
    1. It runs a container named `my-nginx-container` in detached mode, using the image `my-nginx-image:v1`
    2. Additionally, it exposes port `80` of the container to port `81` on the host


## Step 6: Access the Nginx Application

1. Access the application from inside the container:
    ```
    # Start a shell session inside the container
    docker exec -it my-nginx-container bash

    # Access the localhost endpoint
    curl localhost
    ```

    You'll notice that the custom Nginx page we provided is being served.

    You can also see the custom HTML we provided file as follows:
    ```
    cat /usr/share/nginx/html/index.html
    ```

2. Access the application from host machine:

    Since we've exposed our application to the host through port mapping, you can simply open any browser on your host and visit the following endpoint to access the application:

    ```
    localhost:81
    ```

    You'll see the custom nginx page we provided.


## Step 7: View Container Logs

View the container logs as follows:
```
# Command template
docker logs -f <container-id/container-name>

# Actual command
docker logs -f my-nginx-container
```

In your browser, visit `localhost:81` a few times, and you'll see the Nginx access log being streamed in the console.

## Step 8: Clean Up

```
# Stop the container
docker stop my-nginx-container

# Delete the container
docker rm my-nginx-container
```

!!! note
    A container needs to be stopped before it can be deleted. You can't delete a running container.