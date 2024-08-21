# Container Images

## What Constitutes an Image? (And What Doesn't)

- **App Binaries**: The core executable files for your application.
- **Metadata**: Information about the image, including how it should be run.
- **Official Definition**:  
  "An image is an ordered collection of root filesystem changes and the corresponding execution parameters intended for use within a container runtime."
- **Not a Full Operating System**:  
  An image does not include a kernel, as the host system's kernel is used. There are also no kernel modules, such as drivers.
- **Size Variation**:  
  - Images can be as small as a single file, like a Golang static binary.
  - They can also be as large as a full Ubuntu distribution with additional software like Apache, PHP, and more.

## Docker Tags and Images

In Docker, images serve as the foundation for creating containers. Each image can have one or more tags that refer to specific versions or instances of that image. Tags are essential for differentiating between multiple versions of the same image. For example, the `latest` tag refers to the most recent version, while version-specific tags like `1.27.1` indicate particular versions.

### Working with Docker Images and Tags: An Example

```zsh
# List all Docker images currently available on your system.
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image ls
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

# Pull the latest version of the nginx image.
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
e4fff0779e6d: Pull complete 
2a0cb278fd9f: Pull complete 
7045d6c32ae2: Pull complete 
03de31afb035: Pull complete 
0f17be8dcff2: Pull complete 
14b7e5e8f394: Pull complete 
23fa5a7b99a6: Pull complete 
Digest: sha256:447a8665cc1dab95b1ca778e162215839ccbb9189104c79d7ec3a81e14577add
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

# Docker suggests a next step after pulling the image.
What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx

# Pull a specific version of the nginx image.
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker pull nginx:1.27.1
1.27.1: Pulling from library/nginx
Digest: sha256:1540e37eebb9abc5afa4256de1bade6542d50bf69b61b1dd855cb7804aaaf444
Status: Downloaded newer image for nginx:1.27.1
docker.io/library/nginx:1.27.1

# Docker suggests a next step after pulling the image.
What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx:1.27.1
```

### Explanation

In the example above:

1. **Pulling the Latest Image**: The command `docker pull nginx` retrieves the latest version of the nginx image from Docker Hub. If no tag is specified, Docker uses the `latest` tag by default.

2. **Pulling a Specific Version**: The command `docker pull nginx:1.27.1` retrieves a specific version of the nginx image, identified by the `1.27.1` tag. This approach is useful for maintaining consistency across environments by always using a specific image version.

3. **Redundancy in Pulling**: Both `nginx:latest` and `nginx:1.27.1` refer to the same image version in this case. Docker will recognize that the image layers are identical and avoid downloading them again, thus preventing unnecessary redundancy.

4. **Image Digest**: Each image has a unique digest (a SHA256 hash) that ensures its integrity and consistency across different environments. The digest is displayed after pulling an image.

Understanding the difference between tags and image versions allows you to manage your Docker images more efficiently, reducing unnecessary downloads and redundancy.

## Image Layers and the Image Cache: A Deeper Look

1. **Image Layers**: Docker images are built in layers, where each layer represents a change or addition to the image.
2. **Union File System**: Layers are combined using a union file system, which allows for efficient storage and retrieval.
3. **History and Inspect Commands**: Use `docker image history <image>` to view the history of an image.
4. **Copy-on-Write**: When an image is modified, only the changes are stored, thanks to Docker's copy-on-write mechanism.

### Using the `docker image history` Command

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image history nginx:latest
Error response from daemon: No such image: nginx:latest
```

In this example, the command attempts to retrieve the history of the `nginx:latest` image. The error indicates that the image does not exist locally.

**Note**: Every new image begins with a base layer called "scratch." Any subsequent changes to the image create new layers.

### Exploring `docker image history` Options

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image history --help

Usage:  docker image history [OPTIONS] IMAGE

Show the history of an image

Aliases:
  docker image history, docker history

Options:
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -H, --human           Print sizes and dates in human-readable format (default true)
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```

### Using the `docker image inspect` Command

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image inspect --help

Usage:  docker image inspect [OPTIONS] IMAGE [IMAGE...]

Display detailed information on one or more images

Options:
  -f, --format string   Format output using a custom template:
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
```