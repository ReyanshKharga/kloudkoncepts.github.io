---
description: Learn how to containerize a Node.js and Express app. Step-by-step guide for running your web application in Docker containers.
---


# Containerizing a Node.js and Express App

Let's see how we can containerize a `Node.js` and `Express` application.


## Step 1: Create Application Using Node.js and Express

The first step is to create an application using `Node.js` and `Express` framework.

Create `server.js` and `package.json` files as follows:

=== ":octicons-file-code-16: `server.js`"

    ```js linenums="1"
    const express = require('express')
    const app = express()
    var os = require('os');

    const PORT = process.env.PORT || 5000;

    // Returns the hostname, version and other app details
    app.get('/', function (_req, res) {
    let host = os.hostname();
    data = {
        Host: host,
        Version: "v1"
    }
    res.status(200).json(data);
    });

    app.get('/health', function(_req, res) {
    res.status(200).send('Healthy');
    });

    app.get('/random', function(_req, res) {
    randomNumber = Math.floor(Math.random() * 10) + 1;
    console.log(`The random number generated is: ${randomNumber}`);
    res.status(200).send(randomNumber.toString());
    });

    app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
    });
    ```

=== ":octicons-file-code-16: `package.json`"

    ```json linenums="1"
    {
        "dependencies": {
            "express": "^4.18.2"
        }
    }
    ```

Here's what your folder structure should look like:

```
|-- my-folder
│   |-- package.json
│   |-- server.js
```

The above Express app has the following routes:

```
GET /
GET /health
GET /random
```

!!! question "What does each route return?"
    * `GET /` returns a JSON object containing Host and Version.
    * `GET /health` returns the health status.
    * `GET /random` returns a random integer between 1 and 10


## Step 2: Test the Application Locally

Go to the folder containing `package.json` and run the following commands:

```
# Install npm packages
npm install

# Start the express server
node server.js
```

!!! note
    You must have `Node.js` installed on your local machine to run the above commands.

If there are no errors, your Express application will begin serving requests on the specified port, which in our case is `5000`.

Verify if the endpoints are working as expected:
```
curl localhost:5000
curl localhost:5000/health
curl localhost:5000/random
```

You can also access the endpoints mentioned above directly from your browser.

!!! note
    When you run `npm install`, it generates a `package-lock.json` file to record and lock down the specific versions of installed packages and their dependencies. This ensures consistency in package versions across different environments and collaborations.


## Step 3: Create Dockerfile

Now, let's containerize our application. To do that, we first need to create a `Dockerfile` with instructions on how to build the image.

Create the `Dockerfile` as follows:

=== ":octicons-file-code-16: `Dockerfile`"

    ```Dockerfile linenums="1"
    FROM node:18

    # Create app directory
    WORKDIR /usr/src/app

    COPY package*.json ./

    RUN npm install

    # Bundle app source
    COPY . .

    EXPOSE 5000

    CMD [ "node", "server.js" ]
    ```

!!! note "Brief explanation of each command in Dockerfile"

    1. `FROM node:18`: Specifies the base image as Node.js version 18, which serves as the foundation for your image.

    2. `WORKDIR /usr/src/app`: Sets the working directory inside the container to `/usr/src/app`.

    3. `COPY package*.json ./`: Copies the `package.json` and `package-lock.json` (if present) from the local directory into the container's working directory.

    4. `RUN npm install`: Runs the `npm install` command inside the container to install the application's dependencies.

    5. `COPY . .`: Copies the contents of the local directory (your Node.js application code) into the container's working directory.

    6. `EXPOSE 5000`: Informs Docker that the container will listen on port 5000, though it doesn't actually publish the port to the host.

    7. `CMD [ "node", "server.js" ]`: Specifies the command that will be executed when the container starts. In this case, it runs your Node.js application using the `server.js` script.

These steps are essential for building a Docker image for your `Node.js` application, including setting up the environment, copying code and dependencies, and defining how to run the application within the container.


Here's what your folder structure should look like at this point:

```
|-- my-folder
│   |-- package.json
│   |-- server.js
|   |-- Dockerfile
```


## Step 4: Build the Docker Image

With the application code and `Dockerfile` prepared, we are now ready to build the Docker image for our application.

Build the Docker image as follows:
```
# Command template
docker build -t <image-name>:<image-tag> <path-to-Dockerfile>

# Actual command
docker build -t my-node-express-image:v1 .
```

The above command will build an image named `my-node-express-image` with tag `v1`.


## Step 5: Verify the Image

List images and verify if the newly created image is present:

```
docker images
```

Look for the image `name` and the `tag` we specified while building the image.


## Step 6: Run a Container From the Image

Now that we have the image ready, let's create a container from it and test the application.

```
docker run -d --name my-node-express-container -p 5001:5000 my-node-express-image:v1
```

!!! question "What does the above command do?"

    1. It runs a container named `my-node-express-container` in detached mode, using the image `my-node-express-image:v1`. 
    2. Additionally, it exposes port `5000` of the container to port `5001` on the host




## Step 7: Test the Application

1. Test the application from within the container:

    - Start a shell session inside the container

        ```
        docker exec -it my-node-express-container bash
        ```

    - Access the endpoints

        ```
        curl localhost:5000
        curl localhost:5000/health
        curl localhost:5000/random
        ```

2. Test the application from host computer (Outside the container):

    Open any browser on your host computer and hit the following endpoints to verify if port mapping is working as expected.

    ```
    localhost:5001
    localhost:5001/health
    localhost:5001/random
    ```


## Step 8: View Container Logs

```
# Command template
docker logs -f <container-id/container-name>

# Actual command
docker logs -f my-node-express-container
```


## Step 9: Clean Up

```
# Stop the container
docker stop my-node-express-container

# Remove the container
docker rm my-node-express-container
```