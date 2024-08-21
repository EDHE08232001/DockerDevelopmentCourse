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

### Another Example

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker pull nginx:latest
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

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx:latest
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker pull nginx:mainline
mainline: Pulling from library/nginx
Digest: sha256:447a8665cc1dab95b1ca778e162215839ccbb9189104c79d7ec3a81e14577add
Status: Downloaded newer image for nginx:mainline
docker.io/library/nginx:mainline

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx:mainline
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image list
REPOSITORY   TAG        IMAGE ID       CREATED      SIZE
nginx        latest     5ef79149e0ec   6 days ago   188MB
nginx        mainline   5ef79149e0ec   6 days ago   188MB
```

Understanding the difference between tags and image versions allows you to manage your Docker images more efficiently, reducing unnecessary downloads and redundancy.

## Image Layers and the Image Cache: A Deeper Look

Docker images are constructed from layers, which are critical to the efficient use of resources and consistent image management. Understanding how these layers work can greatly improve how you build, manage, and optimize Docker images.

### 1. **Image Layers**

- **Concept**: Docker images are built in layers. Each layer represents a set of changes, like adding a file, installing a package, or configuring settings.
- **Layer Composition**: For example, an Ubuntu-based image may have layers for:
  1. The base Ubuntu system.
  2. Installed packages like Python.
  3. Application-specific files and dependencies.
- **Layer Efficiency**: Layers that remain unchanged between builds are reused, which speeds up the image creation process and reduces the amount of data transferred when images are shared.

**Example**:
```Dockerfile
# Base image layer
FROM ubuntu:20.04

# Layer 1: Install Python
RUN apt-get update && apt-get install -y python3

# Layer 2: Copy application files
COPY . /app

# Layer 3: Set up environment variables
ENV APP_ENV=production
```
In this Dockerfile:
- Each `RUN`, `COPY`, and `ENV` command creates a new layer.
- If the base image or Python installation hasn’t changed, Docker will reuse those layers from its cache in future builds.

### 2. **Union File System**

- **Concept**: The Union File System (UFS) allows Docker to stack these layers together into a single, coherent file system. When you interact with a container, it feels like a complete filesystem, but behind the scenes, it's composed of these layered snapshots.
- **Layer Interaction**: The UFS ensures that any changes made to the file system (like creating a file or modifying a configuration) are stored in a new top layer without altering the original layers.

**Example**:
- Imagine a Docker container built from an Ubuntu image with a Python layer on top. The UFS presents a unified view where both the Ubuntu OS and Python are available as part of a single file system, even though they reside in separate layers.

### 3. **History and Inspect Commands**

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
  -H, --human           Print sizes and dates in human readable format (default true)
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image inspect --help

Usage:  docker image inspect [OPTIONS] IMAGE [IMAGE...]

Display detailed information on one or more images

Options:
  -f, --format string   Format output using a custom template:
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
```

- **History Command**: Use `docker image history <image>` to view all the layers that make up an image. This command shows the commands used to create each layer, making it easier to understand how the image was constructed.

**Example**:
```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image history nginx:latest
IMAGE          CREATED          CREATED BY                                      SIZE
7e4d58f0e5f3   3 days ago       /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon o… 0B
```

- **Inspect Command**: Use `docker image inspect <image>` to get detailed information about an image, including its layers, configuration, and metadata.

**Example**:
```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image inspect nginx:latest
[
    {
        "Id": "sha256:7e4d58f0e5f36e...",
        "RepoTags": [
            "nginx:latest"
        ],
        "Created": "2023-08-15T07:29:51.196Z",
        "Size": 133114949,
        ...
    }
]
```

### 4. **Copy-on-Write**

- **Concept**: Docker uses a copy-on-write (CoW) mechanism. When a container based on an image is started, it begins with a read-only copy of the image. Any modifications made by the container are not written back to the image but are instead stored in a new, writable layer.
- **Efficiency**: This approach allows multiple containers to share the same image while maintaining their own isolated changes.

**Example**:
- Suppose you have a container running an Nginx server based on the `nginx:latest` image. If you modify the Nginx configuration file within the container, Docker stores this change in a new layer without altering the original `nginx:latest` image.

### Summary of Docker Layers and Mechanisms

By leveraging Docker's layered architecture and commands like `history` and `inspect`, you can better manage and optimize container images. Understanding these concepts also helps in debugging issues related to image size, build times, and deployment consistency.

## Image Tagging and Pushing To Docker Hub

Tagging:

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image tag --help

Usage:  docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Aliases:
  docker image tag, docker tag
```

Retagging and Pushing Example:

```zsh
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

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image list
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
nginx        latest    5ef79149e0ec   6 days ago   188MB
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image tag --help

Usage:  docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Aliases:
  docker image tag, docker tag
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image tag nginx edhe08232001/nginx
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image list                        
REPOSITORY           TAG       IMAGE ID       CREATED      SIZE
nginx                latest    5ef79149e0ec   6 days ago   188MB
edhe08232001/nginx   latest    5ef79149e0ec   6 days ago   188MB
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % 
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image push edhe08232001/nginx     
Using default tag: latest
The push refers to repository [docker.io/edhe08232001/nginx]
5f0272c6e96d: Mounted from library/nginx 
f4f00eaedec7: Mounted from library/nginx 
55e54df86207: Mounted from library/nginx 
ec1a2ca4ac87: Mounted from library/nginx 
8b87c0c66524: Mounted from library/nginx 
72db5db515fd: Mounted from library/nginx 
9853575bc4f9: Mounted from library/nginx 
latest: digest: sha256:127262f8c4c716652d0e7863bba3b8c45bc9214a57d13786c854272102f7c945 size: 1778
```