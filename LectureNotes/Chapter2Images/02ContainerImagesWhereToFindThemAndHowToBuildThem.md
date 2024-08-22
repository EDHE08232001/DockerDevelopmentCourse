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

In Docker, tagging an image is a crucial step for version control and organization. Tags act as pointers to specific image versions, making it easier to manage and deploy images across various environments. This section covers the basic commands for tagging Docker images and pushing them to Docker Hub.

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

In this example, here's what happens:

1. **Pulling the Image:** You downloaded (pulled) the `nginx:latest` image from the official `nginx` repository on Docker Hub. This image consists of a set of layers that make up the `nginx` software.

2. **Retagging:** When you retagged the image using `docker image tag nginx edhe08232001/nginx`, you created a new tag (`edhe08232001/nginx:latest`) that points to the same image layers as the original `nginx:latest` image. This tag now exists locally on your machine.

3. **Pushing:** When you pushed the `edhe08232001/nginx:latest` tag to your Docker Hub repository, Docker checked if the image layers already existed on Docker Hub. Since you pulled the image from the official `nginx` repository and didn't make any changes, those layers likely already exist on Docker Hub.

   - Docker doesn't re-upload the layers that are already present in Docker Hub. Instead, it just adds your new tag (`edhe08232001/nginx:latest`) to point to those existing layers in your personal Docker Hub repository.

### Result

- **New Tag:** Your repository (`edhe08232001/nginx`) now contains the tag `latest`, which references the exact same image layers as the original `nginx:latest` image in the official `nginx` repository.

- **No Duplication:** The image layers aren't duplicated or copied anew; only the tag is added to your repository, pointing to the same content that is already present on Docker Hub.

So, in essence, by retagging and pushing, you have created a new reference under your Docker Hub account without duplicating the image content. The image is not new in terms of its contents—it's simply a new label (tag) that points to an existing set of image layers on Docker Hub.

## Docker Files

Dockerfile Example:

```Dockerfile
# NOTE: this example is taken from the default Dockerfile for the official nginx Docker Hub Repo
# https://hub.docker.com/_/nginx/
# NOTE: This file is slightly different than the video, because nginx versions have been updated 
#       to match the latest standards from docker hub... but it's doing the same thing as the video
#       describes

FROM debian:bookworm-slim
# all images must have a FROM
# usually from a minimal Linux distribution like debian or (even better) alpine
# if you truly want to start with an empty container, use FROM scratch

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

# optional environment variable that's used in later lines and set as envvar when container is running
ENV NGINX_VERSION   1.25.3
ENV NJS_VERSION     0.8.2
ENV PKG_RELEASE     1~bookworm


RUN set -x \
# create nginx user/group first, to be consistent throughout docker variants
    && groupadd --system --gid 101 nginx \
    && useradd --system --gid nginx --no-create-home --home /nonexistent --comment "nginx user" --shell /bin/false --uid 101 nginx \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates \
    && \
    NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
    NGINX_GPGKEY_PATH=/usr/share/keyrings/nginx-archive-keyring.gpg; \
    export GNUPGHOME="$(mktemp -d)"; \
    found=''; \
    for server in \
        hkp://keyserver.ubuntu.com:80 \
        pgp.mit.edu \
    ; do \
        echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
        gpg1 --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
    done; \
    test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
    gpg1 --export "$NGINX_GPGKEY" > "$NGINX_GPGKEY_PATH" ; \
    rm -rf "$GNUPGHOME"; \
    apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/* \
    && dpkgArch="$(dpkg --print-architecture)" \
    && nginxPackages=" \
        nginx=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-xslt=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-geoip=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-image-filter=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-njs=${NGINX_VERSION}+${NJS_VERSION}-${PKG_RELEASE} \
    " \
    && case "$dpkgArch" in \
        amd64|arm64) \
# arches officialy built by upstream
            echo "deb [signed-by=$NGINX_GPGKEY_PATH] https://nginx.org/packages/mainline/debian/ bookworm nginx" >> /etc/apt/sources.list.d/nginx.list \
            && apt-get update \
            ;; \
        *) \
# we're on an architecture upstream doesn't officially build for
# let's build binaries from the published source packages
            echo "deb-src [signed-by=$NGINX_GPGKEY_PATH] https://nginx.org/packages/mainline/debian/ bookworm nginx" >> /etc/apt/sources.list.d/nginx.list \
            \
# new directory for storing sources and .deb files
            && tempDir="$(mktemp -d)" \
            && chmod 777 "$tempDir" \
# (777 to ensure APT's "_apt" user can access it too)
            \
# save list of currently-installed packages so build dependencies can be cleanly removed later
            && savedAptMark="$(apt-mark showmanual)" \
            \
# build .deb files from upstream's source packages (which are verified by apt-get)
            && apt-get update \
            && apt-get build-dep -y $nginxPackages \
            && ( \
                cd "$tempDir" \
                && DEB_BUILD_OPTIONS="nocheck parallel=$(nproc)" \
                    apt-get source --compile $nginxPackages \
            ) \
# we don't remove APT lists here because they get re-downloaded and removed later
            \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
# (which is done after we install the built packages so we don't have to redownload any overlapping dependencies)
            && apt-mark showmanual | xargs apt-mark auto > /dev/null \
            && { [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; } \
            \
# create a temporary local APT repo to install from (so that dependency resolution can be handled by APT, as it should be)
            && ls -lAFh "$tempDir" \
            && ( cd "$tempDir" && dpkg-scanpackages . > Packages ) \
            && grep '^Package: ' "$tempDir/Packages" \
            && echo "deb [ trusted=yes ] file://$tempDir ./" > /etc/apt/sources.list.d/temp.list \
# work around the following APT issue by using "Acquire::GzipIndexes=false" (overriding "/etc/apt/apt.conf.d/docker-gzip-indexes")
#   Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
#   ...
#   E: Failed to fetch store:/var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages  Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
            && apt-get -o Acquire::GzipIndexes=false update \
            ;; \
    esac \
    \
    && apt-get install --no-install-recommends --no-install-suggests -y \
                        $nginxPackages \
                        gettext-base \
                        curl \
    && apt-get remove --purge --auto-remove -y && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list \
    \
# if we have leftovers from building, let's purge them (including extra, unnecessary build deps)
    && if [ -n "$tempDir" ]; then \
        apt-get purge -y --auto-remove \
        && rm -rf "$tempDir" /etc/apt/sources.list.d/temp.list; \
    fi \
# forward request and error logs to docker log collector
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
# create a docker-entrypoint.d directory
    && mkdir /docker-entrypoint.d

# COPY docker-entrypoint.sh /
# COPY 10-listen-on-ipv6-by-default.sh /docker-entrypoint.d
# COPY 20-envsubst-on-templates.sh /docker-entrypoint.d
# COPY 30-tune-worker-processes.sh /docker-entrypoint.d
# ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 80
# expose these ports on the docker virtual network
# you still need to use -p or -P to open/forward these ports on host

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]
# required: run this command when container is launched
# only one CMD allowed, so if there are multiple, last one wins
```

A **Dockerfile** is a script that contains a series of instructions on how to build a Docker image. Each instruction in the Dockerfile adds a layer to the image, and when the instructions are completed, you get a final Docker image that can be deployed as a container.

### Example Dockerfile Breakdown

Here’s a detailed explanation of the provided Dockerfile:

```Dockerfile
# Base Image
FROM debian:bookworm-slim
```

- **FROM**: The `FROM` instruction initializes a new build stage and sets the base image for subsequent instructions. All Dockerfiles must start with a `FROM` instruction. In this example, the base image is `debian:bookworm-slim`, a minimal version of Debian Linux. Using a lightweight base image helps reduce the final image size.

```Dockerfile
LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"
```

- **LABEL**: The `LABEL` instruction adds metadata to the image. In this case, it provides information about the maintainer of the Dockerfile, which can be useful for documentation and future reference.

```Dockerfile
ENV NGINX_VERSION   1.25.3
ENV NJS_VERSION     0.8.2
ENV PKG_RELEASE     1~bookworm
```

- **ENV**: The `ENV` instruction sets environment variables that will be available in subsequent instructions and when the container is running. Here, `NGINX_VERSION`, `NJS_VERSION`, and `PKG_RELEASE` are defined, which are later used in the `RUN` command to ensure the correct versions of NGINX and its related packages are installed.

```Dockerfile
RUN set -x \
# create nginx user/group first, to be consistent throughout docker variants
    && groupadd --system --gid 101 nginx \
    && useradd --system --gid nginx --no-create-home --home /nonexistent --comment "nginx user" --shell /bin/false --uid 101 nginx \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates \
    ...
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    && mkdir /docker-entrypoint.d
```

- **RUN**: The `RUN` instruction executes commands in a new layer on top of the current image and commits the results. It’s often used to install packages, create files or directories, and configure the environment. 

  In this example:
  - It starts with `set -x`, which makes the shell print each command before executing it, useful for debugging.
  - It creates a system group and user for `nginx` to ensure that the processes run with non-root privileges.
  - It installs necessary packages (`gnupg1`, `ca-certificates`) and retrieves a GPG key to verify the NGINX packages.
  - Depending on the system architecture, it either directly installs pre-built NGINX packages or compiles them from source.

  The final steps forward the NGINX logs to Docker’s log collector and create a directory for additional entry-point scripts.

```Dockerfile
EXPOSE 80
```

- **EXPOSE**: The `EXPOSE` instruction informs Docker that the container will listen on the specified network ports at runtime. Here, port `80` is exposed, which is the default HTTP port for NGINX. This instruction doesn’t actually publish the port on the host; you’ll need to use the `-p` flag at runtime to do so.

```Dockerfile
STOPSIGNAL SIGQUIT
```

- **STOPSIGNAL**: This instruction sets the system call signal that will be sent to the container to initiate a graceful shutdown. In this example, `SIGQUIT` is used, which tells NGINX to quit immediately and shut down gracefully.

```Dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

- **CMD**: The `CMD` instruction provides defaults for an executing container. In this case, it tells Docker to start NGINX with the `-g "daemon off;"` flag, which runs NGINX in the foreground. Only one `CMD` instruction is allowed in a Dockerfile; if you have multiple, the last one will be used.

### Summary of Dockerfile Structure

- **FROM**: Defines the base image.
- **LABEL**: Adds metadata to the image.
- **ENV**: Sets environment variables.
- **RUN**: Executes commands to install software, set up the environment, and configure the system.
- **EXPOSE**: Documents the ports on which the container listens.
- **STOPSIGNAL**: Specifies how the container should be stopped.
- **CMD**: Defines the default command to run when the container starts.

This structure forms a repeatable and transparent process for creating Docker images. By understanding each part of the Dockerfile, you can customize and optimize your images for various use cases.

## Building Images: Running `docker image build`

Example:

```zsh
edwardhe@Edwards-MacBook-Air DockerBuildExample % docker image build --help
Start a build

Usage:  docker buildx build [OPTIONS] PATH | URL | -

Start a build

Aliases:
  docker build, docker builder build, docker image build, docker buildx b

Options:
      --add-host strings              Add a custom host-to-IP mapping (format: "host:ip")
      --allow strings                 Allow extra privileged entitlement (e.g., "network.host", "security.insecure")
      --annotation stringArray        Add annotation to the image
      --attest stringArray            Attestation parameters (format: "type=sbom,generator=image")
      --build-arg stringArray         Set build-time variables
      --build-context stringArray     Additional build contexts (e.g., name=path)
      --builder string                Override the configured builder instance (default "desktop-linux")
      --cache-from stringArray        External cache sources (e.g., "user/app:cache", "type=local,src=path/to/dir")
      --cache-to stringArray          Cache export destinations (e.g., "user/app:cache", "type=local,dest=path/to/dir")
      --call string                   Set method for evaluating build ("check", "outline", "targets") (default "build")
      --cgroup-parent string          Set the parent cgroup for the "RUN" instructions during build
      --check                         Shorthand for "--call=check" (default )
  -f, --file string                   Name of the Dockerfile (default: "PATH/Dockerfile")
      --iidfile string                Write the image ID to a file
      --label stringArray             Set metadata for an image
      --load                          Shorthand for "--output=type=docker"
      --metadata-file string          Write build result metadata to a file
      --network string                Set the networking mode for the "RUN" instructions during build (default "default")
      --no-cache                      Do not use cache when building the image
      --no-cache-filter stringArray   Do not cache specified stages
  -o, --output stringArray            Output destination (format: "type=local,dest=path")
      --platform stringArray          Set target platform for build
      --progress string               Set type of progress output ("auto", "plain", "tty", "rawjson"). Use plain to show container output (default "auto")
      --provenance string             Shorthand for "--attest=type=provenance"
      --pull                          Always attempt to pull all referenced images
      --push                          Shorthand for "--output=type=registry"
  -q, --quiet                         Suppress the build output and print image ID on success
      --sbom string                   Shorthand for "--attest=type=sbom"
      --secret stringArray            Secret to expose to the build (format: "id=mysecret[,src=/local/secret]")
      --shm-size bytes                Shared memory size for build containers
      --ssh stringArray               SSH agent socket or keys to expose to the build (format: "default|<id>[=<socket>|<key>[,<key>]]")
  -t, --tag stringArray               Name and optionally a tag (format: "name:tag")
      --target string                 Set the target build stage to build
      --ulimit ulimit                 Ulimit options (default [])

Experimental commands and flags are hidden. Set BUILDX_EXPERIMENTAL=1 to show them.

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview 
edwardhe@Edwards-MacBook-Air DockerBuildExample % 
edwardhe@Edwards-MacBook-Air DockerBuildExample % docker image build -t customnginx .
[+] Building 57.6s (7/7) FINISHED                                                                                                                                                    docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                                                 0.0s
 => => transferring dockerfile: 6.83kB                                                                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/debian:bookworm-slim                                                                                                                              4.5s
 => [auth] library/debian:pull token for registry-1.docker.io                                                                                                                                        0.0s
 => [internal] load .dockerignore                                                                                                                                                                    0.0s
 => => transferring context: 2B                                                                                                                                                                      0.0s
 => [1/2] FROM docker.io/library/debian:bookworm-slim@sha256:2ccc7e39b0a6f504d252f807da1fc4b5bcd838e83e4dec3e2f57b2a4a64e7214                                                                       10.3s
 => => resolve docker.io/library/debian:bookworm-slim@sha256:2ccc7e39b0a6f504d252f807da1fc4b5bcd838e83e4dec3e2f57b2a4a64e7214                                                                        0.0s
 => => sha256:4735ff618e63fa90d91e11a6cd0ea73bcfe3859924de33076eaff2630a1f0232 1.46kB / 1.46kB                                                                                                       0.0s
 => => sha256:e4fff0779e6ddd22366469f08626c3ab1884b5cbe1719b26da238c95f247b305 29.13MB / 29.13MB                                                                                                     6.2s
 => => sha256:2ccc7e39b0a6f504d252f807da1fc4b5bcd838e83e4dec3e2f57b2a4a64e7214 1.85kB / 1.85kB                                                                                                       0.0s
 => => sha256:70d4c04302bdcd71c4fa21b6c12e99888380a07f04e3d44452b961bca046489d 529B / 529B                                                                                                           0.0s
 => => extracting sha256:e4fff0779e6ddd22366469f08626c3ab1884b5cbe1719b26da238c95f247b305                                                                                                            3.9s
 => [2/2] RUN set -x     && groupadd --system --gid 101 nginx     && useradd --system --gid nginx --no-create-home --home /nonexistent --comment "nginx user" --shell /bin/false --uid 101 nginx    42.2s
 => exporting to image                                                                                                                                                                               0.5s 
 => => exporting layers                                                                                                                                                                              0.4s 
 => => writing image sha256:ac33385c2afd9eaa4214f491cd25ed744df50bc031e42d86de449ced17114840                                                                                                         0.0s 
 => => naming to docker.io/library/customnginx                                                                                                                                                       0.0s 
                                                                                                                                                                                                          
View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/x737ra5rwoxr3jdl7muwqkkdv                                                                                                

 3 warnings found (use docker --debug to expand):
 - LegacyKeyValueFormat: "ENV key=value" should be used instead of legacy "ENV key value" format (line 16)
 - LegacyKeyValueFormat: "ENV key=value" should be used instead of legacy "ENV key value" format (line 17)
 - LegacyKeyValueFormat: "ENV key=value" should be used instead of legacy "ENV key value" format (line 15)

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview 
edwardhe@Edwards-MacBook-Air DockerBuildExample % 
edwardhe@Edwards-MacBook-Air DockerBuildExample % ls
Dockerfile
edwardhe@Edwards-MacBook-Air DockerBuildExample % docker image list
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
customnginx   latest    ac33385c2afd   11 seconds ago   187MB
```

### Building Docker Images: Using the `docker image build` Command

The `docker image build` command is essential for creating Docker images from a Dockerfile. This command processes the instructions in the Dockerfile, builds the image layer by layer, and finally outputs a Docker image that can be run as a container.

### Basic Syntax

```shell
docker image build [OPTIONS] PATH | URL | -
```

- **PATH**: The path to the directory containing your Dockerfile.
- **URL**: A Git URL where the Dockerfile is located.
- **-**: Reads the Dockerfile from the standard input (stdin).

#### Commonly Used Options

- **`--file` (-f)**: Specify the Dockerfile to use (default is `PATH/Dockerfile`).
- **`--tag` (-t)**: Name and optionally tag the image in the `name:tag` format.
- **`--no-cache`**: Do not use cache when building the image, ensuring all steps are executed freshly.
- **`--pull`**: Always attempt to pull newer versions of the base image.
- **`--build-arg`**: Pass build-time variables to the Dockerfile, useful for dynamic configurations.
- **`--platform`**: Set the target platform for the build, useful for cross-compilation.

#### Example: Building an Image

Below is an example that demonstrates how to build a Docker image using a Dockerfile.

```zsh
docker image build -t customnginx .
```

- **`-t customnginx`**: This option tags the image as `customnginx`.
- **`.`**: This specifies the current directory as the build context, which should contain the Dockerfile.

#### Step-by-Step Build Process

Here’s a breakdown of what happens during the build:

1. **Loading the Dockerfile**: Docker reads the instructions from the specified Dockerfile.
    - Example: `load build definition from Dockerfile`
2. **Fetching the Base Image**: Docker pulls the base image (e.g., `debian:bookworm-slim`) from the registry if it's not already available locally.
    - Example: `load metadata for docker.io/library/debian:bookworm-slim`
3. **Executing Dockerfile Instructions**: Each instruction in the Dockerfile (e.g., `RUN`, `COPY`, `ADD`) is executed, creating a new layer in the image.
    - Example: `RUN set -x && groupadd ...`
4. **Exporting the Image**: After executing all instructions, Docker exports the built layers into a final image.
    - Example: `exporting to image`
5. **Tagging the Image**: The image is tagged according to the `-t` option provided.
    - Example: `naming to docker.io/library/customnginx`

#### Viewing the Built Image

Once the build process is complete, you can view the created image using:

```zsh
docker image list
```

This command lists all Docker images on your system. Example output:

```zsh
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
customnginx   latest    ac33385c2afd   11 seconds ago   187MB
```

#### Debugging and Warnings

During the build process, Docker may output warnings or errors. For instance, legacy syntax warnings like using `ENV key value` instead of `ENV key=value` may appear:

```plaintext
3 warnings found (use docker --debug to expand):
 - LegacyKeyValueFormat: "ENV key=value" should be used instead of legacy "ENV key value" format (line 16)
```

To resolve these, ensure your Dockerfile follows the recommended syntax.

#### Further Exploration

- **Explore Image Vulnerabilities**: After building the image, you can view potential vulnerabilities and recommendations by running:

```shell
docker scout quickview
```

This command provides a summary of security issues and suggestions for improving the image's security posture.

**Note**: If you want to ensure that all images are removed, even if they are associated with containers, you can combine the docker rmi command with the docker image prune -a command:

`docker rmi $(docker images -q) -f`

- docker rmi: Removes images by ID.
- $(docker images -q): This returns a list of image IDs.
- -f (or --force): Forces the removal of the images, even if they are being used by containers.

## Extending Official Images

This shows how can we extend/change an official image from docker hub

```Dockerfile
FROM nginx:latest
# highly recommend you always pin version for anything beyond dev/learn

WORKDIR /usr/share/nginx/html
# changing working directory to root of nginx webhost
# using WORKDIR is prefered to using `RUN cd /some/path`

COPY index.html index.html

# I do not have to specify any EXPOSE or CMD because they in my FROM
```

### A Better Example

#### Original Dockerfile

```Dockerfile
# Base Image
FROM nginx:1.21.0

# Set the working directory
WORKDIR /usr/share/nginx/html

# Copy a simple HTML file to the container
COPY index.html /usr/share/nginx/html/index.html

# Expose port 80 (default for nginx)
EXPOSE 80

# Run nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]
```

This Dockerfile uses the nginx:1.21.0 image as the base, sets the working directory to the default web root of Nginx, copies an index.html file into that directory, and then exposes port 80, which is the default port for HTTP traffic. The CMD instruction tells Docker to run Nginx in the foreground.

#### Extended Dockerfile

```Dockerfile
# Extend the official Nginx image
FROM nginx:1.21.0

# Set the working directory
WORKDIR /usr/share/nginx/html

# Copy additional static files
COPY index.html /usr/share/nginx/html/index.html
COPY about.html /usr/share/nginx/html/about.html
COPY styles.css /usr/share/nginx/html/styles.css

# Add a custom Nginx configuration file
COPY custom-nginx.conf /etc/nginx/nginx.conf

# Install additional software (e.g., curl)
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean

# Expose ports
EXPOSE 80 443

# Override the default command with a custom entrypoint script
COPY entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

# CMD can be left out if the entrypoint handles starting the service
```

#### Explanation of Extensions

1. **FROM Instruction**: We start by extending the `nginx:1.21.0` image. This means that our Docker image will inherit everything from the base image (Nginx in this case).

2. **WORKDIR**: The working directory is set to the root of the Nginx web host (`/usr/share/nginx/html`). This is where Nginx serves its content from, and all subsequent file operations will be relative to this directory.

3. **COPY Instructions**: 
    - The `index.html`, `about.html`, and `styles.css` files are copied into the container's web directory, allowing Nginx to serve them.
    - A custom Nginx configuration file (`custom-nginx.conf`) is copied to override the default configuration. This could include custom server settings, virtual hosts, or security policies.

4. **RUN Instruction**: 
    - We install `curl` using the `apt-get` package manager. This can be useful for debugging, health checks, or other purposes.
    - The `apt-get clean` command is used to remove unnecessary files and reduce the image size.

5. **EXPOSE Instruction**: 
    - Ports 80 (HTTP) and 443 (HTTPS) are exposed. This indicates to Docker that these ports should be made available to the host.

6. **ENTRYPOINT and CMD Instructions**: 
    - We override the default `CMD` with a custom `entrypoint.sh` script. This allows for more complex startup logic, such as environment variable substitution or dynamic configuration generation before starting Nginx.
    - The `ENTRYPOINT` makes this script the primary executable, ensuring it runs first when the container starts.

#### How to Extend

Extending an official Docker image like Nginx is a common practice when you need to customize the base functionality to suit your needs. The key steps are:

1. **Select a Base Image**: Choose an official image that provides the core functionality you need (e.g., Nginx for a web server).

2. **Add Custom Content**: Use `COPY` to include any files or directories that should be part of your application (e.g., HTML files, configuration files).

3. **Install Additional Software**: Use `RUN` to install packages or perform other setup tasks not included in the base image.

4. **Customize Configuration**: Override or add configuration files to change the behavior of the software inside the container.

5. **Define Entry Point**: Use `ENTRYPOINT` or `CMD` to specify what should happen when the container starts, allowing you to control the container’s execution flow.

6. **Expose Ports**: Use `EXPOSE` to define which network ports should be accessible from outside the container.

By following these steps, you can create Docker images that are tailored to your application's specific requirements.

## Practice Assignment: Build Your Own Image
1. Dockerfiles are part process workflow and part art
2. Take existing Node.js app and Dockerize it
3. Make **Dockerfile**. Build it. Test it. Push it. (rm it). Run it.
4. Expect it to be iterative. Rarely do I get it right the first time
5. Details in `dockerfile-assignment-1/Dockerfile`
6. Use the Alpine version of the official `node` 6.x image
7. Expected result is web site at http://localhost
8. Tag and Push to your Docker Hub account (free)
9. Remove your image and created container from local cache, run again from Hub
10. Afther all above, do cleanup

### Instructions from the app developer

- you should use the `node` official image, with the alpine 6.x branch (`node:6-alpine`)
  - (Yes this is a 2-year old image of node, but all official images are always available on Docker Hub forever, to ensure even old apps still work. It is common to still need to deploy old app versions, even years later.)
- This app listens on port 3000, but the container should listen on port 80 of the Docker host, so it will respond to [http://localhost:80](http://localhost:80) on your computer
- Then it should use the alpine package manager to install tini: `apk add --no-cache tini`.
- Then it should create directory /usr/src/app for app files with `mkdir -p /usr/src/app`, or with the Dockerfile command `WORKDIR /usr/src/app`.
- Node.js uses a "package manager", so it needs to copy in package.json file.
- Then it needs to run 'npm install' to install dependencies from that file.
- To keep it clean and small, run `npm cache clean --force` after the above, in the same RUN command.
- Then it needs to copy in all files from current directory into the image.
- Then it needs to start the container with the command `/sbin/tini -- node ./bin/www`. Be sure to use JSON array syntax for CMD. (`CMD [ "something", "something" ]`)
- In the end you should be using FROM, RUN, WORKDIR, COPY, EXPOSE, and CMD commands

### My Dockerfile

```Dockerfile
# Use the official Node.js image based on Alpine Linux
# The `FROM` instruction sets the base image for subsequent instructions.
# Here, we are using the `node:6-alpine` image, which is a lightweight version
# of Node.js 6.x built on top of Alpine Linux, a minimalistic distribution
# that helps keep the image size small.
FROM node:6-alpine

# Install tini via Alpine package manager
# The `RUN` instruction is used to execute commands during the build process.
# `apk add --no-cache tini` installs the `tini` package, a small but powerful
# init system, which helps manage the lifecycle of the Node.js process and 
# handles tasks like process reaping. The `--no-cache` option ensures that 
# package index files aren't stored in the Docker image, keeping it small.
RUN apk add --no-cache tini

# Set working directory to /usr/src/app
# The `WORKDIR` instruction sets the working directory for any subsequent 
# `COPY`, `RUN`, and `CMD` instructions. If the directory doesn't exist,
# it will be created automatically. This is where our application files 
# will reside inside the container.
WORKDIR /usr/src/app

# Copy the package.json file into the image
# The `COPY` instruction copies files and directories from the host file system
# into the Docker image. Here, we're copying the `package.json` file from the 
# current directory on the host into the working directory (`/usr/src/app`) 
# of the container. The `package.json` file contains the metadata for the project,
# including dependencies that need to be installed.
COPY package.json ./

# Install dependencies and clean up npm cache
# This `RUN` instruction does two things in a single layer:
# 1. Runs `npm install` to install all dependencies listed in `package.json`.
# 2. Cleans up the npm cache using `npm cache clean --force` to reduce the 
#    size of the image. This ensures that no unnecessary files are left behind.
# Combining commands with `&&` allows them to be executed in one layer, 
# further optimizing the image size.
RUN npm install && npm cache clean --force

# Copy all the files from the current directory to the container
# The second `COPY` instruction copies all files from the current directory 
# on the host to the working directory (`/usr/src/app`) in the container.
# This includes the application code and any other necessary files.
COPY . .

# Expose port 80 to the Docker host
# The `EXPOSE` instruction informs Docker that the container listens on the 
# specified network ports at runtime. Here, we're exposing port 80, which is 
# the default HTTP port. Although this doesn't actually publish the port, 
# it's a good practice to document the intended network usage.
EXPOSE 80

# Start the application using tini to manage Node.js
# The `CMD` instruction specifies the command to run when the container starts.
# Here, we're using `tini` as the init system to manage the Node.js process,
# ensuring that the application is properly started and any child processes
# are handled correctly. The `CMD` instruction uses the JSON array format, 
# which is the preferred format as it prevents the command from being 
# interpreted by the shell.
CMD ["/sbin/tini", "--", "node", "./bin/www"]
```

#### Additional Notes:

- **Layer Caching:** Docker builds images in layers. Each instruction in the Dockerfile creates a new layer, and Docker caches these layers. This is why it's efficient to order commands that change less frequently towards the top of the Dockerfile (e.g., installing dependencies) and commands that change frequently (e.g., copying the source code) towards the bottom. This way, Docker can reuse cached layers and only rebuild layers that have changed.
  
- **Alpine Linux:** Using Alpine Linux as a base image is a common practice in Docker due to its small size (~5 MB), which helps in creating minimal images. However, because it is minimalistic, some common tools and libraries might be missing, so you might need to install them manually if required.

- **Tini:** `Tini` is a simple init system that helps in properly managing child processes within containers. Containers run with a single process by default, and if that process spawns child processes, they can become orphaned when the main process exits. `Tini` ensures proper process handling, making it a good practice to use in production environments.

- **CMD vs. ENTRYPOINT:** `CMD` sets the default command and/or parameters when a container is run without specifying a command. If `ENTRYPOINT` is used instead, it configures the container to run as an executable. Typically, you'd use `CMD` for default behavior that can be overridden by the user, and `ENTRYPOINT` for a fixed command that should always run.

This enhanced study note should give you a deeper understanding of each part of your Dockerfile and the best practices involved in Dockerizing an application.

#### Note:

Let's break down the `COPY` commands and their components in the Dockerfile:

##### `COPY package.json ./`

- **`package.json`**:
  - This is the source file that you want to copy into the Docker image. It is located in the current directory on your host machine (where you're running the `docker build` command). The `package.json` file contains metadata about your Node.js project, including its dependencies, scripts, and other configuration details.

- **`./`**:
  - This is the destination path inside the Docker image. The `./` refers to the current working directory within the Docker image, which is set by the previous `WORKDIR /usr/src/app` instruction in the Dockerfile. So, `./` here means that `package.json` will be copied to the `/usr/src/app` directory inside the container.

##### `COPY . .`

- **First `.` (dot)**:
  - This refers to the source directory on the host machine. The single dot (`.`) represents the current directory where the Docker build context is located (the directory from which you ran the `docker build` command). This means all files and subdirectories within this current directory will be copied into the Docker image.

- **Second `.` (dot)**:
  - This is the destination directory inside the Docker image. Like in the previous command, this dot represents the current working directory inside the Docker image, which is `/usr/src/app` (as set by `WORKDIR`). So, all the files from the current directory on your host will be copied into `/usr/src/app` inside the Docker image.

##### Summary:

- **`package.json`**: Located in the current directory on your host machine, it's copied to `/usr/src/app` inside the Docker image.
- **First `.` in `COPY . .`**: Represents the entire current directory on your host machine, including all files and subdirectories.
- **Second `.` in `COPY . .`**: Represents the current working directory inside the Docker image, which is `/usr/src/app`.

This setup ensures that all necessary files, including `package.json` and the rest of the application code, are copied into the correct directory within the Docker image, allowing the application to run properly in the container.

### My ZSH Command Steps

```zsh
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker build -t edhe08232001/node-app .
[+] Building 26.7s (12/12) FINISHED                                                                                                                                          docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                                         0.0s
 => => transferring dockerfile: 835B                                                                                                                                                         0.0s
 => [internal] load metadata for docker.io/library/node:6-alpine                                                                                                                             4.1s
 => [auth] library/node:pull token for registry-1.docker.io                                                                                                                                  0.0s
 => [internal] load .dockerignore                                                                                                                                                            0.0s
 => => transferring context: 527B                                                                                                                                                            0.0s
 => [1/6] FROM docker.io/library/node:6-alpine@sha256:17258206fc9256633c7100006b1cfdf25b129b6a40b8e5d37c175026482c84e3                                                                       5.5s
 => => resolve docker.io/library/node:6-alpine@sha256:17258206fc9256633c7100006b1cfdf25b129b6a40b8e5d37c175026482c84e3                                                                       0.0s
 => => sha256:e9fa13fdf0f5f7d338eba59bab8b18d1b49e35550bb17659a4806be99423cecc 15.45MB / 15.45MB                                                                                             3.4s
 => => sha256:ccc877228d8f9d2009e6d278a5757937b4b379b91561353c60462cc2b9ad884b 1.33MB / 1.33MB                                                                                               1.4s
 => => sha256:17258206fc9256633c7100006b1cfdf25b129b6a40b8e5d37c175026482c84e3 1.72kB / 1.72kB                                                                                               0.0s
 => => sha256:ac775e5c37b78443690a840b22973dd5a60ff2cc17ecd1d3921a45e831071d61 951B / 951B                                                                                                   0.0s
 => => sha256:dfc29bfa7d41dd1616257cfe7cb4462c8fd566e9c4d2b2cefb5ea2adf57ad6b8 5.23kB / 5.23kB                                                                                               0.0s
 => => sha256:bdf0201b3a056acc4d6062cc88cd8a4ad5979983bfb640f15a145e09ed985f92 2.76MB / 2.76MB                                                                                               2.2s
 => => extracting sha256:bdf0201b3a056acc4d6062cc88cd8a4ad5979983bfb640f15a145e09ed985f92                                                                                                    0.2s
 => => extracting sha256:e9fa13fdf0f5f7d338eba59bab8b18d1b49e35550bb17659a4806be99423cecc                                                                                                    1.8s
 => => extracting sha256:ccc877228d8f9d2009e6d278a5757937b4b379b91561353c60462cc2b9ad884b                                                                                                    0.1s
 => [internal] load build context                                                                                                                                                            0.0s
 => => transferring context: 429.80kB                                                                                                                                                        0.0s
 => [2/6] RUN apk add --no-cache tini                                                                                                                                                        1.5s
 => [3/6] WORKDIR /usr/src/app                                                                                                                                                               0.0s 
 => [4/6] COPY package.json ./                                                                                                                                                               0.0s 
 => [5/6] RUN npm install && npm cache clean --force                                                                                                                                        15.1s 
 => [6/6] COPY . .                                                                                                                                                                           0.0s
 => exporting to image                                                                                                                                                                       0.2s
 => => exporting layers                                                                                                                                                                      0.1s
 => => writing image sha256:2f3190174d4ada6353a40e990aee4728f237bbb6540a7af52e64beb22b3dee67                                                                                                 0.0s
 => => naming to docker.io/edhe08232001/node-app                                                                                                                                             0.0s

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/y0duhe9mxvrr2v33u7y75225v

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview 
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker run -p 80:3000 edhe08232001/node-app
GET / 200 128.248 ms - 290
GET /stylesheets/style.css 200 6.276 ms - 111
GET /images/picard.gif 200 5.207 ms - 417700
GET /favicon.ico 404 9.505 ms - 970
^C%                                                                                                                                                                                               
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker push edhe08232001/node-app
Using default tag: latest
The push refers to repository [docker.io/edhe08232001/node-app]
79ce2aa036e6: Pushed 
9ca6ff453045: Pushed 
f30f736412f0: Pushed 
2ea5e11f7493: Pushed 
be06dc12ec87: Pushed 
f168d52a989d: Mounted from library/node 
17b7c23fba03: Mounted from library/node 
a464c54f93a9: Mounted from library/node 
latest: digest: sha256:861f88163b3df68a27cab8efd5d330a63ff08b10b36d66bf5560056581b3c680 size: 1995
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker rm -f $(docker ps -aq)
2a2f806cb65f
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker rmi edhe08232001/node-app
Untagged: edhe08232001/node-app:latest
Untagged: edhe08232001/node-app@sha256:861f88163b3df68a27cab8efd5d330a63ff08b10b36d66bf5560056581b3c680
Deleted: sha256:2f3190174d4ada6353a40e990aee4728f237bbb6540a7af52e64beb22b3dee67
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker image list
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker container list
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker container run -p 80:3000 edhe08232001/node-app
Unable to find image 'edhe08232001/node-app:latest' locally
latest: Pulling from edhe08232001/node-app
bdf0201b3a05: Already exists 
e9fa13fdf0f5: Already exists 
ccc877228d8f: Already exists 
023a5a1d9f48: Already exists 
4d152678ac22: Already exists 
c0416c0632ad: Already exists 
697ae7bcfdd3: Already exists 
c654f67cf82e: Already exists 
Digest: sha256:861f88163b3df68a27cab8efd5d330a63ff08b10b36d66bf5560056581b3c680
Status: Downloaded newer image for edhe08232001/node-app:latest
GET / 304 88.035 ms - -
GET /images/picard.gif 304 3.548 ms - -
GET /stylesheets/style.css 304 0.937 ms - -
^C%                                                                                                                                                                                               
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker system prune -af
Deleted Containers:
5e44e4e008636f5a783e773260cfa7d7f47c030e57b3abbe3140be0517f476b1

Deleted Images:
untagged: edhe08232001/node-app:latest
untagged: edhe08232001/node-app@sha256:861f88163b3df68a27cab8efd5d330a63ff08b10b36d66bf5560056581b3c680
deleted: sha256:2f3190174d4ada6353a40e990aee4728f237bbb6540a7af52e64beb22b3dee67

Deleted build cache objects:
mtpf44p8iqb7v7g1155ls0hns
ph6apr6evs495mgw7tg7ede2r
zt9day14rtx8upf4dbp7el3nw
i021deqxoux3i0n2ibp1cdxqw
sckbd6smm6c3kvvashvfsegkq
49u2u9y85ze4lfgvpjzheq3qk
60pmjhu0wh5w318vflt3ph0k6
bdef19ipd175gh9z4p5gv6s4p
upzfzgbbjq7by4ce35s97gen4
pks0hsboimvzke8w5zb2fnuiu
pikpxovfwpstscl308a2rmg4k

Total reclaimed space: 8.554MB
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker container list
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
edwardhe@Edwards-MacBook-Air dockerfile-assignment-1 % docker image list
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

#### My ZSH Command Steps Explanation

```zsh
# Build the Docker image with a specific tag
# `docker build` builds a Docker image from a Dockerfile and a "context" (the files in the current directory).
# `-t edhe08232001/node-app` tags the built image with the name `edhe08232001/node-app`. This makes it easier to refer to the image later.
# `.` specifies the build context, meaning the current directory.
docker build -t edhe08232001/node-app .

# Output (example):
# [+] Building 26.7s (12/12) FINISHED
# This output shows the progress of each step in the Dockerfile, with timings.

# Run the Docker container, mapping port 80 on the host to port 3000 in the container
# `docker run` creates and starts a container from an image.
# `-p 80:3000` maps port 80 on the host (your computer) to port 3000 in the container, allowing you to access the application at `http://localhost`.
docker run -p 80:3000 edhe08232001/node-app

# Output (example):
# GET / 200 128.248 ms - 290
# This shows HTTP requests being made to the server running in the container.

# Push the Docker image to Docker Hub
# `docker push` uploads an image to a Docker registry, in this case, Docker Hub.
# `edhe08232001/node-app` specifies the image to push. The image must be tagged with a repository name (like `edhe08232001/node-app`) to be pushed.
docker push edhe08232001/node-app

# Output (example):
# The push refers to repository [docker.io/edhe08232001/node-app]
# This output shows the progress of pushing each layer of the image to Docker Hub.

# Remove all containers (both running and stopped)
# `docker rm` removes one or more containers.
# `-f` forces the removal of running containers (stops them before removal).
# `$(docker ps -aq)` gets the IDs of all containers (`-a` lists all containers, and `-q` returns only the IDs).
docker rm -f $(docker ps -aq)

# Remove the Docker image from the local system
# `docker rmi` removes one or more Docker images.
# `edhe08232001/node-app` specifies the image to remove. By default, Docker tries to untag the image first, then delete it if no containers are using it.
docker rmi edhe08232001/node-app

# Output (example):
# Untagged: edhe08232001/node-app:latest
# This output shows that the image has been untagged and then deleted.

# List all Docker images on the system
# `docker image list` lists all Docker images stored locally.
docker image list

# Output (example):
# REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
# This output lists the repository name, tag, image ID, creation date, and size of each image.

# List all Docker containers (both running and stopped)
# `docker container list` lists all running containers by default.
# `-a` flag lists all containers, including stopped ones.
docker container list

# Run the Docker container again, pulling the image from Docker Hub if it doesn't exist locally
# `docker container run` creates and starts a container.
# `-p 80:3000` maps port 80 on the host to port 3000 in the container.
# `Unable to find image 'edhe08232001/node-app:latest' locally` indicates that Docker couldn't find the image locally, so it pulls it from Docker Hub.
docker container run -p 80:3000 edhe08232001/node-app

# Output (example):
# Digest: sha256:861f88163b3df68a27cab8efd5d330a63ff08b10b36d66bf5560056581b3c680
# This output shows the status of the container, including which image was pulled and the image digest.

# Remove all unused data (containers, networks, images, build cache)
# `docker system prune` cleans up unused Docker objects.
# `-a` flag includes unused images in the cleanup.
# `-f` flag forces the cleanup without prompting for confirmation.
docker system prune -af

# Output (example):
# Deleted Containers: 
# Deleted Images:
# This output shows the IDs of all removed containers and images, along with the total reclaimed space.

# List all Docker containers (should be empty after prune)
docker container list

# Output (example):
# CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
# This shows that no containers are currently present.

# List all Docker images (should be empty after prune)
docker image list

# Output (example):
# REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
# This shows that no images are currently present.
```

---

##### Additional Notes:

- **Docker Build Context:** The context is the set of files and directories that Docker uses to build the image. The `.` at the end of `docker build` means the current directory is used as the context. This is important because only files in the context can be accessed by the Dockerfile.

- **Docker Run with Port Mapping:** The `-p` flag in `docker run` is crucial for connecting the host machine to the container. The format `hostPort:containerPort` maps a port on the host to a port inside the container. This is how you can expose a web server running in the container to the outside world.

- **Docker Prune:** This command is powerful but should be used with caution. `docker system prune -af` will remove all unused containers, images, networks, and build caches. The `-f` flag skips the confirmation prompt, so make sure you don't need any of the data being pruned.

This enhanced guide should help you understand the reasoning behind each command and how the options and flags influence Docker's behavior.