# Crteaing And Using Containers

## Some Basic Commands

### Check Docker Version
This also verifies if cli can talk to the engine

```zsh
docker version
```

### Examine Docker Configurations
```zsh
docker info
```

## Docker Command Line Structure
old way(still works):

```zsh
docker <command> (options)
```

new way:

```zsh
docker <command> <sub-command> (options)
```

## Images vs Containers
    1. An image is an application we want to run
    2. A container is an instance of that image running as a process
    3. You can have many containers running off the same image
    4. In this lecture, our image will be Nginx web server
    5. Docker's default image "registry" is called Docker Hub (hub.docker.com)

### Example Command
```zsh
docker container run --publish 80:80 nginx
```

1. Download the image 'nginx' from Docker Hub
2. Started a new container from that image
3. Opened port 80 on the host IP
4. Routes that traffic to the container IP, port 80

#### Concept of Port Mapping

In `--publish 80:80`, the first 80 is the port on your host machine (This is the physical or virtual machine where Docker is installed and running in this case. It could be your personal computer, a server, or a cloud instance). Any traffic reaches this port will be routed to the container. The second 80 is the container port, this is the port inside the container where the NGINX server is listening for incoming requests.

#### Components of Port Mapping
1. Host Machine:
    - This is the physical or virtual machine where Docker is installed and running. It could be your personal computer, a server, or a cloud instance.
2. Container:
    - A container is an isolated environment where your application runs. In this case, it’s an environment running the NGINX web server.
3. Ports:
    - Ports are communication endpoints for your applications. Each service or application listens for incoming network traffic on specific ports.

#### How Routing Works?
1. Request Initiaition
    - When you type 'httl://localhost' in your browser, it initiates a request to port 80 on your host machine
    - Port 80 is the default port for localhost
2. Host Port
    - The request reaches port 80 on your host machine. Docker intercepts this request because you have mapped this port to a container.
3. Routing to Container Port
    - Docker then routes this request to port 80 inside the container. This is where the NGINX server is running and listening for incoming HTTP requests.
4. Response
    - NGINX processes the request and sends a response back through the same route: from the container port 80 back to the host port 80, and finally to your browser.

#### Virtual Representation
```.md
Browser (http://localhost:80)
        |
        v
Host Machine (Port 80) ----> Docker Routes Traffic ----> Container (Port 80)
```

### Example Command
We can also run `docker container run --publish 80:80 --detach nginx`

When you include the --detach option, Docker will start the container and return control of the terminal to you immediately, showing you the container ID. This allows you to run other commands without waiting for the container to finish.

#### What It Does
1. Create and Start a Container:
        - Docker creates and starts a new container using the nginx image.
2. Port Mapping:
        - Port 80 on the host machine is mapped to port 80 inside the container.
3. Run in Background:
        - The --detach option makes the container run in the background.
4. Output:
        - Docker will return the container ID to the terminal, indicating that the container is running in the background.

### List Container Command
```zsh
docker container ls
```

### After Running A Container, You Can Also Stop It
```zsh
docker container stop <container_id>
```

In most cases, you only need to type the first several digits of the container id in order to make it unique

### ls Command with -a
```zsh
docker container ls -a
```

#### Concept: run vs. start
`docker container run` always starts a *new* container

use `docker container start` to start an existing stopped one

### run with --name
```zsh
docker container run --publish 80:80 --detach --name webhost nginx
```

Now you customized the name of a container

### With a Specified Name, You Can Check Logs
```zsh
docker container logs webhost
```

```zsh
docker container logs <container_name>
```

### top Command shows the processes running inside the container
```zsh
docker container top <container_name>
```

### To remove existing containing
```zsh
docker container rm <container_id> <container_id> <container_id>
```

However, only inactive containers can be removed

Nontheless, you can always add -f

```zsh
docker container rm -f <container_id> ...
```

## What happens in `docker container run <image_name>`?
1. Looks for that image locally in image cache
2. If docker doesn't find any specified image, docker then looks in remote image repository (defaults to docker hub)
3. Downloads the latest version (nginx: latest by default)
4. When required image is ready to go, docker creats a new container on that image and prepares to start
5. Gives the container a virtual IP on a private network inside docker engine (Custimize the Networking)
6. Opens up port 80 on host and forwards to port 80 in container in the earlier example (if ports specified, if no `--publish` no ports opened)
7. That container then starts by using CMD in the image DOCKERFILE

### A General Example
```zsh
docker container run --publish <HostPort>:<ContainerPort> --name <ContainerName> -d <Image>:<version> <Image: This changes CMD on start> -T
```

### A Concret Example
```zsh
docker container run --publish 8080:80 --name mynginx -d nginx:1.21.1 nginx -g "daemon off;"
```

#### Detailed Explanation of the Example
1. docker container run:
    - Starts a new Docker container.
2. --publish 8080:80:
    - Maps port 8080 on the host machine to port 80 inside the container. This means you can access the containerized NGINX server by navigating to http://localhost:8080.
3. --name mynginx:
    - Names the container mynginx. This makes it easier to manage the container (e.g., stopping it with docker container stop mynginx).
4. -d:
    - Runs the container in detached mode, freeing up the terminal for other commands.
5. nginx:1.21.1:
    - Specifies the Docker image nginx and version 1.21.1 to use for the container.
6. nginx -g "daemon off;":
    - Overrides the default command in the NGINX image. The command nginx -g "daemon off;" tells NGINX to run in the foreground (non-daemon mode), which is necessary for running it in a Docker container that needs to stay active.

## Container vs. VM: It's just a process

### Containers aren't Mini-VM's
    - They are just processes
    - Limited to what resources they can access
    - Exit when process stops

You can actually find containers as processes with `ps aux`, which shows all running processes. `ps aux | grep <process name>`

Or to show docker processes with `docker ps`

## More Examples
```zsh
docker container run --detach --publish 80:80 --name nginxWebHost nginx
```

```zsh
docker container run --detach --publish 8080:80 --name apacheServer httpd
```

```zsh
docker container run --detach --publish 3306:3306 --name db --env MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
```