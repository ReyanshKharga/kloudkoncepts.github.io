---
description: Publish your Docker image to Docker Hub effortlessly with our step-by-step guide. Share your containerized applications with the world.
---


# Publish Image to Docker Hub

To share your image with your team or the public, you should store it in a Docker registry. Docker Hub serves as a registry that enables you to either privately share your images with your team or make them accessible to the public.

Let's see how you can do that.


## Step 1: Create an Account on Docker Hub

Visit [Docker Hub]{:target="_blank"} and create an account.


## Step 2: Create a Repository

A Docker `repository` is a collection of Docker images that are stored and made available for others to use and download.

Let's create a public repository named `node-express`.


## Step 3: Push Image to Docker Repository

1. Login to Docker Hub

    ```
    docker login
    ```


2. Re-tag the Docker image you want to push

    ```
    # Command template
    docker tag <existing-image>:<tag> <docker-hub-username>/<repository-name>:<tag>

    # Actual command
    docker tag my-node-express-image:v1 reyanshkharga/node-express:v1
    ```


3. Push the image to repository

    ```
    # Command template
    docker push <docker-hub-username>/<repository-name>:<tag>

    # Actual command
    docker push reyanshkharga/node-express:v1
    ```


## Step 4: Verify the Image in Docker Hub

Go to Docker Hub and verify if the image is pushed. Now, if the image is public, anyone can pull the image.



<!-- Hyperlinks -->
[Docker Hub]: https://hub.docker.com/