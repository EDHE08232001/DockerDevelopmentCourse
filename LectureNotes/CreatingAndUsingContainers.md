# Crteaing And Using Containers

## Some Basic Commands

### Check Docker Version
This also verifies if cli can talk to the engine

```
docker version
```

### Examine Docker Configurations
```
docker info
```

## Docker Command Line Structure
old way(still works):

```
docker <command> (options)
```

new way:

```
docker <command> <sub-command> (options)
```

## Images vs Containers
    1. An image is an application we want to run
    2. A container is an instance of that image running as a process
    3. You can have many containers running off the same image
    4. In this lecture, our image will be Nginx web server
    5. Docker's default image "registry" is called Docker Hub (hub.docker.com)

### Example Command
```
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