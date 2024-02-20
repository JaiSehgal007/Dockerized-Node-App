# Docker Introduction and Usage Guide

## Problem Statement
As a developer, setting up the development environment locally and ensuring consistency across team members can be challenging. Here are some common problems faced:
1. Extra effort required to set up the entire environment.
2. Version discrepancies leading to errors and changes in the environment.
3. OS incompatibility issues.
4. Repetitive setup tasks, especially when deploying to the cloud or dealing with auto-scaling.

In essence, Docker provides a solution to replicate environments effectively.

## What is Docker?
Docker allows the creation of containers where configurations such as OS, tools, etc., can be specified. These containers can then be shared with the team or deployed on a cloud platform. Regardless of the host machine's configuration, Docker containers ensure consistent behavior.

Containers are lightweight, enabling easy creation and destruction as needed.

## Installing Docker
There are two main components:
- **Docker Daemon:** The tool responsible for spinning up, building, and scaling containers.
- **Docker Desktop:** A GUI that provides insights into the system's state.

To install Docker, follow these steps:
1. Verify Docker version: `docker -v`
2. Run an interactive Ubuntu container: `docker run -it ubuntu`
3. If Docker Daemon is not running, use the following command: 

```bash
"C:\Program Files\Docker\Docker\DockerCli.exe" -SwitchDaemon
```


## Containers and Images
Think of containers as analogous to machines, and images as operating systems. Containers cannot function independently; they rely on images to run. Multiple containers can run from the same image, and each container is isolated.

When working in development:
- Create custom images with required configurations (e.g., Ubuntu + Node + MongoDB).
- Publish these images on Docker Hub for easy sharing.
- Developers can run these images in local containers or use them in cloud environments.

Useful Docker commands:
- `docker container ls`: Lists running containers.
- `docker container ls -a`: Lists all containers.
- `docker start <container-name>`: Starts a stopped container.
- `docker stop <container-name>`: Stops a container.
- `docker run --rm my_python_app:latest`: Executes a command and deletes the container afterward.
- `docker exec <container-name> <command>`: Runs a command inside a container.
- `docker exec -it <container-name> bash`: Starts an interactive shell in a container.
- `docker images` or `docker image ls`: Lists available images.
- `docker run`: Creates and starts a new Docker container from an image.
- `docker start`: Starts one or more existing stopped containers.
- `docker exec`: Runs a command inside a running container.

## Port Mapping
Containers are isolated environments. Port mapping allows exposing ports within containers to the host machine. For example:

```bash
docker run -it -p 1020:1025 mailhog/mailhog
```

This command maps port 1025 of the container to port 1020 on the local machine.

## Environment Variables
Environment variables can be passed into Docker containers using the -e flag. For example:

```bash
docker run -it -p 8000:8000 -e key=value -e key2=value2 node
```

This sets environment variables within the Docker container.


# How to Dockerize an Application

Now that we have created a basic Node.js application which is having some files, we want to make an image of these files so that it can be run on other containers easily.

For that, we create a `Dockerfile`, in which we write all the configuration of how to make the image.

### Step 1: Select the Base Image

We start by selecting the base image. In this case, we'll use Ubuntu as the base image:

```Dockerfile
FROM ubuntu
```

This creates a base layer of Ubuntu in the container.

### Step 2: Install Node.js
Now, we need to install Node.js on the Ubuntu base image. You can search the web on how to install Node.js on Ubuntu. The following commands are typically used:


```Dockerfile
RUN apt-get update && \
    apt-get install -y curl
```

The -y flag means yes to all questions asked in the installation process.


### Step 3: Copy Files and Specify Entrypoint
After setting up Node.js, we copy the package.json file into the container:

```Dockerfile
COPY package.json package.json
```

Then, we specify the entrypoint:

```Dockerfile
ENTRYPOINT [ "node", "index.js" ]
```

This means whenever this image is run in a container, we run node index.js.

### Step 4: Build the Image
Now, we build the Docker image:

```bash
docker build -t dockerized-node-app .
```

The `.` represents the current directory.

## Caching the Image
Docker caches the image layers. If nothing has changed, the subsequent builds will be faster. It's good practice to install all dependencies before copying files to make use of layer caching.

## Publishing the Image
To publish this image to Docker Hub, first, go to Docker Hub, create an account, and then create a repository. Then, tag your local image with the repository name and push it:


```bash
docker tag dockerized-node-app <username>/dockerized-node-app
docker push <username>/dockerized-node-app
```

## What is Docker Compose?
Docker Compose is a tool for defining and running multi-container Docker applications. It allows you to define the services, networks, and volumes in a single YAML file.

To use Docker Compose, create a docker-compose.yml file:

```yaml
version: '3.8'

services:
  my_postgres_service:
    image: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_DB: review
      POSTGRES_PASSWORD: password

  redis:
    image: redis
    ports:
      - "6379:6379"

```

Then, run:

```bash
docker-compose up
```

This will create and start all the configurations defined in the docker-compose.yml file.

To stop and remove containers, networks, and volumes, run:

```bash
docker-compose down
```

To run in detached mode (in the background), use:

```bash
docker-compose up -d
```

## Using Services in Another Image

You can use services defined in Docker Compose in another image by specifying dependencies in the docker-compose.yml file:

```yaml
version: '3.8'

services:
  my_postgres_service:
    image: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_DB: review
      POSTGRES_PASSWORD: password

  redis:
    image: redis
    ports:
      - "6379:6379"

  node_app:
    build:
      context: ./path/to/your/node/app
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - my_postgres_service
      - redis

```

This allows you to use the `my_postgres_service` and `redis` services in the `node_app` service.


## Docker Networking

Docker networking enables containers to communicate with the outside world, including the internet.

1. **Bridge Mode Network:**
   - Default driver.
   - Connects containers to the bridge network if no network is specified explicitly.
   - Command to inspect bridge network:
```bash
    docker network inspect bridge
```
   - Bridge network establishes connectivity between the host machine and the Docker container.

2. **Host Mode:**
   - Container directly connects to the host machine`s network.
   - No need for port mapping as they share the same network.

3. **None Mode:**
   - Container has no access to the internet.

### Custom Network Creation

To create a custom network in bridge mode:
```bash
docker network create -d bridge <network_name>
```

Example:
``` bash
docker network create -d bridge youtube
```

Creating containers on the custom network:
``` bash
docker run -it --network=youtube --name tony_stark ubuntu
docker run -it --network=youtube --name doc_strange busybox
```

Containers on the same network can communicate.

## Docker Volumes

Docker volumes provide persistent storage for containers.

- **Volume Mounting:**
  - Mounts a directory/volume from the host to the container.
  - Example command:
``` bash
    docker run -it -v C:\docker_folders:/home/abc
```
  - Data in the mounted volume persists even if the container is destroyed.

- **Custom Volumes:**
  - Create a custom volume:
``` bash
    docker volume create <volume_name>
```
  - Mount custom volume while creating a container:
``` bash
    --mount source=<volume_name>
```


# Efficient Caching in Layers and Docker Multi-Stage Builds

## Efficient Caching in Layers

In Docker, layers provide a way to optimize the build process by caching intermediate results. However, changing code in one layer can invalidate subsequent layers' cache. Here are some strategies to optimize caching:

### Default Approach

```Dockerfile
Layer1:
FROM ubuntu

Layer2:
RUN apt-get update
RUN apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -
RUN apt-get upgrade -y
RUN apt-get install -y nodejs

Layer3:
COPY package.json package.json
COPY package-lock.json package-lock.json
COPY .gitignore .gitignore
COPY index.js index.js

Layer4:
RUN npm install

Layer5:
ENTRYPOINT [ "node", "index.js" ]
```

### Optimization Strategies

1. Separate Copying Dependencies and Code:

```Dockerfile
Layer3:
COPY package.json package.json
COPY package-lock.json package-lock.json

Layer4:
RUN npm install

Layer5:
COPY .gitignore .gitignore
COPY index.js index.js

```

2. Copy Everything with a Single Command:

```dockerfile
Layer5:
COPY . .
```

3. Use .dockerignore File to exclude unnecessary files.

### Final Dockerfile:

```dockerfile
FROM ubuntu

RUN apt-get update
RUN apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -
RUN apt-get upgrade -y
RUN apt-get install -y nodejs

WORKDIR /app

COPY package.json package.json
COPY package-lock.json package-lock.json

RUN npm install

COPY .gitignore .gitignore
COPY index.js index.js

ENTRYPOINT [ "node", "index.js" ]
```

# Docker Multi-Stage Builds
Multi-stage builds help optimize Docker images by allowing us to discard intermediate build artifacts. Here's how it works:

```dockerfile
FROM ubuntu

# Install dependencies
RUN apt-get update
RUN apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -
RUN apt-get upgrade -y
RUN apt-get install -y nodejs
RUN apt-get install typescript

COPY package.json /app/package.json
COPY package-lock.json /app/package-lock.json

# Install npm dependencies and compile TypeScript
RUN npm install
RUN tsc -p .

COPY .gitignore /app/.gitignore
COPY index.js /app/index.js

ENTRYPOINT [ "node", "app/index.js" ]

```

## Multi-Stage Build:
```dockerfile
FROM ubuntu as build

# Install dependencies
RUN apt-get update
RUN apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -
RUN apt-get upgrade -y
RUN apt-get install -y nodejs
RUN apt-get install typescript

COPY package.json /app/package.json
COPY package-lock.json /app/package-lock.json

# Install npm dependencies and compile TypeScript
RUN npm install
RUN tsc -p .

FROM node as runner

WORKDIR /app
COPY --from=build /app /app

ENTRYPOINT [ "node", "index.js" ]
```





