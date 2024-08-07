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
docker container run --detach --publish 3306:3306 --name db --env MYSQL_RANDOM_ROOT_PASSWORD=true mysql
```

## What's going on in containers: CLI Process Monitoring

### Example
```zsh
=> docker container run --detach --name nginx nginx
=> docker container run --detach --name mysql --env MYSQL_RANDOM_ROOT_PASSWORD=true mysql
=> docker container ls
    ... Output: You have two processes/containers running
=> docker container top mysql
    ... Processes in mysql container
=> docker container top nginx
    ... Processes in nginx container
=> docker container inspect mysql
    ... How is mysql container configured and metadata config
=> docker container stats
    ... Live performance data of all containers
```

## Getting a Shell Inside Containers: No Need For SSH
1. `docker container run -it`: start new container interactively
2. `docker container exec -it`: run additional command in existing command

*Note*: -i is interactive and -t is pseudo-tty which stimulates a real terminal, like what SSH does

### Examples
Running a New Container with an Interactive Terminal

```zsh
# Run an nginx container with an interactive terminal and bash shell
=> docker container run -it --name proxy nginx bash
    # -it: Interactive terminal
    # --name proxy: Name the container 'proxy'
    # nginx: Use the 'nginx' image
    # bash: Override the default command and run 'bash' instead
    # Running this command gives you a terminal inside the running container

# Inside the container's bash shell
=># ls -al
    # List all files inside the container

=># exit
    # Exit the bash shell, which stops the container because the container's main process (bash) has ended
```

Running an Ubuntu Container with an Interactive Terminal

```zsh
# Run an Ubuntu container with an interactive terminal
=> docker container run -it --name ubuntu ubuntu
    # -it: Interactive terminal
    # --name ubuntu: Name the container 'ubuntu'
    # ubuntu: Use the 'ubuntu' image
    # If the image is not available locally, it will start downloading it
    # This command opens a shell inside the Ubuntu container

=># exit
    # Exit the shell, which stops the Ubuntu container because the container's main process (the shell) has ended
```

Restarting a Stopped Container in Interactive Mode

```zsh
# Restart the previously stopped Ubuntu container in interactive mode
=> docker container start -ai ubuntu
    # start: Restart the container
    # -a: Attach to the container's output
    # -i: Interactive mode
    # This command restarts the Ubuntu container and attaches to its shell

=># exit
    # Exit the shell, which stops the container again
```

Executing a Command Inside a Running Container

```zsh
# Run a bash shell inside an already running MySQL container
=> docker container exec -it mysql bash
    # exec: Execute a command in a running container
    # -it: Interactive terminal
    # mysql: Name of the running MySQL container
    # bash: The command to run inside the container (in this case, bash shell)
    # This command gives you a terminal inside the already running MySQL container

=># ps aux
    # Show all running processes inside the MySQL container

=># exit
    # Exit the bash shell
    # This only stops the bash shell process, not the MySQL container itself, because exec runs additional processes inside the container
```

Listing Running Containers

```zsh
# List all running containers
=> docker container ls
    # This command shows that the MySQL container is still running even after exiting the bash shell
    # This is because the primary process (MySQL server) inside the container continues to run
```

Additional Notes: Pulling an Alpine Image and Running a Shell

```zsh
# Pull the latest Alpine image
=> docker pull alpine
    # Pull the latest version of the Alpine Linux image

# Run an Alpine container with an interactive terminal
=> docker container run -it alpine sh
    # -it: Interactive terminal
    # alpine: Use the 'alpine' image
    # sh: Run the 'sh' shell (bash is not included in the Alpine image by default)
```

#### Summary
- Interactive Mode (-it): Provides a way to interact with the container via a terminal.
- Attaching to Output (-a): Useful for monitoring and interacting with running processes.
- Executing Commands (exec): Allows running additional commands inside an already running container without stopping it.

## Concept: Docker Network

### Docker Network Defaults
1. Each container connected to a private virtual network `bridge`
2. Each virtual network routes through NAT firewall on host IP
3. All containers on a virtual network can talk to each other without -p
4. Best pratice is to create a new virtual network for each app:
    - network `my_web_app` for mysql and php/apache containers
    - network `my_api` for mongo and nodejs conatiners
5. "Batteries included, but removable":
    - Defaults work well in many cases, but easy to swap to customize
6. Make new virtual networks
7. Attach containers to more than one virtual network (or none)
8. Skip virtual networks and use host ip (--net=host)
9. Use different docker network drivers to gain new abilities

#### To inspect ip of a container
Example:

```zsh
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' <container name>
```

#### To check host machine ip address
```zsh
ifconfig en0
```

### So container ip is not the host machines ip, what happened?

#### Docker Networking Basics
1. Network Namespaces:
    - Each Docker container runs in its own network namespace. This isolation allows containers to have their own IP addresses, separate from the host.
2. Bridge Network:
    - By default, Docker uses a bridge network called docker0. When a container is started, it gets an IP address from the range assigned to this bridge network. This IP address is visible within the context of Docker but not from the outside network unless specifically configured.
3. Host IP:
    - The host machine has its own IP address which is used to communicate with the outside world. This is the IP you see when you run `ifconfig en0` or `ip a`.

#### Explanation
- Container IP: The command docker container inspect --format '{{ .NetworkSettings.IPAddress }}' <container name> fetches the IP address of the container within the Docker network. This IP address is used for communication between containers on the same Docker network.
- Host IP: The command `ifconfig en0` or `ip a` fetches the IP address of the host machine's network interface (e.g., en0 on macOS or eth0 on Linux). This IP address is used for communication between the host machine and the outside world.

#### What happens when a container is started:
- Docker assigns it an IP address from the docker0 bridge network.
- This IP is internal to Docker and is not exposed to the host machine’s network directly.
- The host machine has its own IP address assigned by the network it's connected to (e.g., your local router or ISP).

## Docker Networks: CLI Management
- `docker network ls` shows networks
- `docker network inspect` inspects a network
- `docker network create --driver` creats a network
- `docker network connect` attaches a network to container
- `docker network disconnect` detaches a network from container

### When you run the `docker network ls`, you are most likely to see these 3
1. `--network bridge` default docker virtual network which is NAT'ed behind host ip
2. `--network host` it gains performance by skipping virtual networks but sacrifices security of container model
3. `--network none` removes eth0 and only leave you with localhost interface in container

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
9100cc00fe3b   bridge    bridge    local
8f4dc6256e0d   host      host      local
8735aa6a0362   none      null      local
```

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "9100cc00fe3b204ce57681803dbb1e6fbc3347e156367a172e2148a56ee814df",
        "Created": "2024-08-06T13:02:23.074269887Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "65535"
        },
        "Labels": {}
    }
]
```

### Configure network when start a container
```zsh
docker container run --detach --name <app name> --network <network name> <Image>
```

### Docker Network: Default Security
1. Create your apps so frontend/backend sit on the same docker network
2. Their inter-communication never leaves host
3. All externally exposed ports closed by default
4. You must expose manually with `-p` flag, which is better default security!!!
5. This gets even better later with Swarm and Overlay networks

## Docker Network: DNS and How Containers Find Each Other

### *Forget IP's*
Static IP's and using IP's for talking to containers is an anti-pattern. Do your best to avoid it.

### Docker DNS
Docker daemon has a built-in DNS server that containers use by default

### DNS Default Names
Docker defaults the hostname to the container's name, but you can also set aliases

Lets say, for example, you have two containers on yoru own customized network, a and b

you can do `docker container exec -it my_nginx ping new_nginx` and `docker container exec -it new_nginx ping my_nginx`

### Customized Networks
When you create your own custom network, Docker sets up DNS automatically. This allows containers to find each other by name, eliminating the need for static IPs.

#### Example of Creating a Custom Network and Running Containers
```zsh
docker network create my_custom_network
docker run -d --name my_nginx --network my_custom_network nginx
docker run -d --name new_nginx --network my_custom_network nginx
```

Now, my_nginx can ping new_nginx and vice versa using their container names.

### REMEMBER, the default bridge network does not have DNS buil-in server by default. So use --link
```zsh
docker run -d --name my_nginx nginx
docker run -d --name new_nginx --link my_nginx nginx
docker exec -it new_nginx ping my_nginx
```

### Summary: Docker Network
- Avoid using static IPs for container communication.
- Use Docker's built-in DNS for name resolution.
- Create custom networks for automatic DNS setup.
- Use container names or aliases to facilitate communication.
- Avoid the deprecated --link option; use custom networks instead.

## More Examples of Docker Commands

### Example: CLI App Testing
- Use different Linux distro containers to check `curl` cli tool version
- Use two different terminal windows to start 'bash' in both `centos:7` and `ubuntu:14.04`, using `-it`
- Learn the `docker container --rm` option so you can save cleanup
- Ensure `curl` is installed and on latest version for that distro
    * ubuntu: `apt -get update && apt -get install curl`
    * centos: `yum update curl`
- Check `curl --version`
