# Containers Images

## What is in an image? (And what isn't)
- **App binaries**
- **Metadata** about the image data and how to run the image
- **Official Definition**:  
  "An image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime."
- **Not a complete OS**:  
  No kernel (since the host provides it), no kernel modules (e.g., drivers)
- **Small as one file (your app binary)** like golang static binary
- **Big** as a Ubuntu distro with apt, and apache, PHP, and more installed

## Docker Tags and Images

In Docker, images are essential for creating containers. Each image can have one or more tags that refer to specific versions or instances of that image. Tags are used to differentiate between multiple versions of the same image, such as a `latest` tag for the most recent version and version-specific tags like `1.27.1`.

### Example of Working with Docker Images and Tags

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

# Next steps suggestion by Docker after pulling the image.
What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx

# Pull a specific version of the nginx image.
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker pull nginx:1.27.1
1.27.1: Pulling from library/nginx
Digest: sha256:1540e37eebb9abc5afa4256de1bade6542d50bf69b61b1dd855cb7804aaaf444
Status: Downloaded newer image for nginx:1.27.1
docker.io/library/nginx:1.27.1

# Next steps suggestion by Docker after pulling the image.
What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx:1.27.1
```

### Explanation

In the above example:

1. **Pulling the Latest Image**: The command `docker pull nginx` pulls the latest version of the nginx image from Docker Hub. The `latest` tag is used by default if no tag is specified. 

2. **Pulling a Specific Version**: The command `docker pull nginx:1.27.1` pulls a specific version of the nginx image, identified by the tag `1.27.1`. This is useful when you want to ensure consistency in your environment by always using a specific version of an image.

3. **Redundancy in Pulling**: Both the `nginx:latest` and `nginx:1.27.1` refer to the same image version in this case, meaning there is no need to pull the second image if you have already pulled the latest one. Docker will recognize that the image layers are identical and won't download them again.

4. **Image Digest**: Each image has a unique digest (a SHA256 hash) that ensures its integrity and consistency across different environments. This digest is displayed after pulling an image.

By understanding the difference between tags and image versions, you can better manage your Docker images and avoid unnecessary downloads and redundancies.

## Images and Their Layers: Discover the Image Cache
1. Image Layers
2. Union File System
3. `history` and `inspect` command
4. copy on write

Command: `docker image history <image>`, use this command to view the history of a specific Docker image

```zsh
edwardhe@Edwards-MacBook-Air DockerDevelopmentCourse % docker image history nginx:latest
Error response from daemon: No such image: nginx:latest
```

In this example, the command attempts to retrieve the history of the nginx:latest image. The error indicates that the image does not exist locally.

### Using the history Command with Options

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

