---
description: Learn how to streamline CI/CD for EKS with our comprehensive guide on leveraging AWS CodePipeline. Master automated deployment on Amazon EKS effortlessly.
---

# CI CD for EKS Using AWS CodePipeline

Now, let's see how you can set up CI/CD for a containerized application on EKS using AWS CodePipeline.


## Step 1: Create a Private GitHub Repository

In the first step let's create a private github repository. Let's call it `node-app`. You can name it anything you prefer.

Make sure to choose `gitignore` for Node.js application. This will create a `.gitignore` file for Node.js application.



## Step 2: Clone the GitHub Repository

Now, let's clone the repository on local machine so that we can make changes and push it to the repo.

```
git clone <github-repository-url>
```


## Step 3: Test the Application Locally

This is a simple Node.js app using express. Make sure you have `Node.js` and `npm` installed on your local machine so that you can test it locally.

```
# Verify if node is installed
node --version

# Verify if npm is installed
npm --version
```

=== ":octicons-file-code-16: `server.js`"

    ```js linenums="1"
    const express = require('express')
    const app = express()
    var os = require('os');

    const PORT = process.env.PORT || 5000;

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

    app.listen(PORT, () => {
      console.log(`Server is running on port ${PORT}`);
    });
    ```

=== ":octicons-file-code-16: `package.json`"

    ```json linenums="1"
    {
      "name": "nodeapp",
      "version": "1.0.0",
      "description": "",
      "main": "server.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC",
      "dependencies": {
        "express": "^4.18.2"
      }
    }
    ```

Assuming your folder structure looks like the one below:

```
|-- nodeapp
│   |-- package.json
│   |-- server.js
```

Run the following command to install the npm packages:

```
npm install
```

Run the following command to test the app:

```
node server.js
```

Now, open another terminal and call localhost on port 5000.

```
# Call root endpoint
curl localhost:5000

## Call health endpoint
curl localhost:5000/health
```

Or, visit these URLs in any browser.



## Step 4: Containerize the App and Test

Let's create a `Dockerfile` and create a Docker image out of it. Also, add a `.dockerignore` file.


!!! note
    A `.dockerignore` is a configuration file that describes files and directories that you want to exclude when building a Docker image.


=== ":octicons-file-code-16: `Dockerfile`"

    ```Dockerfile linenums="1"
    FROM node:18

    # Create app directory
    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json
    COPY package*.json ./

    # Install packages mentioned package.json
    RUN npm install

    # Bundle app source
    COPY . .

    EXPOSE 5000
    CMD [ "node", "server.js" ]
    ```

=== ":octicons-file-code-16: `.dockerignore`"

    ```yaml linenums="1"
    # Dependency directories
    node_modules/
    jspm_packages/

    # Environment variable files
    config.env
    .env
    .env.development.local
    .env.test.local
    .env.production.local
    .env.local

    # Logs
    logs
    *.log
    ```

Assuming your folder structure looks like the one below:

```
|-- nodeapp
│   |-- Dockerfile
│   |-- .dockerignore
│   |-- server.js
```

Run the following command to build the docker image:

```
docker build -t my-node-app .
```

Now, run a container from the docker image:

```
docker run --name my-node-container -p 5000:5000 my-node-app
```

Verify the application endpoints:

```
curl localhost:5000
curl localhost:5000/health
```

Stop and delete the container:

```
# list containers
docker ps -a | grep my-node-container

# stop the container
docker stop my-node-container

# Delete the container
docker rm my-node-container
```


