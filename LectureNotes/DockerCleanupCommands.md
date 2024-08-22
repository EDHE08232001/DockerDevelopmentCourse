# Docker Cleanup Commands Cheatsheet

This cheatsheet covers the essential Docker cleanup commands to help you manage and clean up your Docker environment efficiently.

## Stopping Containers
```bash
docker stop <container1> <container2>
```
- **Description**: Stops one or more running containers.
- **General Expression**: `docker stop <container> [<container> ...]`

## Removing Containers
```bash
docker rm <container1> <container2>
```
- **Description**: Removes one or more stopped containers.
- **General Expression**: `docker rm <container> [<container> ...]`

## Force Removing Running Containers
```bash
docker rm -f <container1> <container2>
```
- **Description**: Forces the removal of running containers by stopping them first.
- **General Expression**: `docker rm -f <container> [<container> ...]`
- **Flags**:
  - `-f`, `--force`: Forces the removal of running containers.

## Removing All Stopped Containers
```bash
docker container prune -f
```
- **Description**: Removes all stopped containers.
- **General Expression**: `docker container prune [flag]`
- **Flags**:
  - `-f`, `--force`: Skips confirmation prompt.

## Removing Docker Networks
```bash
docker network rm <network1> <network2>
```
- **Description**: Removes one or more Docker networks that are not in use by any containers.
- **General Expression**: `docker network rm <network> [<network> ...]`

## Pruning Unused Networks
```bash
docker network prune -f
```
- **Description**: Removes all unused Docker networks.
- **General Expression**: `docker network prune [flag]`
- **Flags**:
  - `-f`, `--force`: Skips confirmation prompt.

## Removing Docker Images
```bash
docker rmi <image1> <image2>
```
- **Description**: Removes one or more Docker images.
- **General Expression**: `docker rmi <image> [<image> ...]`

## Force Removing Images
```bash
docker rmi <image> -f
```
- **Description**: Forces the removal of images even if they are being used by containers.
- **General Expression**: `docker rmi <image> [<image> ...] [flag]`
- **Flags**:
  - `-f`, `--force`: Forces the removal of images.

## Removing All Unused Images
```bash
docker image prune -a -f
```
- **Description**: Removes unused images, including intermediate images.
- **General Expression**: `docker image prune [flag]`
- **Flags**:
  - `-a`, `--all`: Removes all unused images.
  - `-f`, `--force`: Skips confirmation prompt.

## Removing All Images by ID
```bash
docker rmi $(docker images -q) -f
```
- **Description**: Removes all Docker images by their IDs.
- **General Expression**: `docker rmi $(docker images -q) [flag]`

## Removing Docker Volumes
```bash
docker volume rm <volume1> <volume2>
```
- **Description**: Removes one or more Docker volumes.
- **General Expression**: `docker volume rm <volume> [<volume> ...]`

## Pruning Unused Volumes
```bash
docker volume prune -f
```
- **Description**: Removes all unused Docker volumes.
- **General Expression**: `docker volume prune [flag]`
- **Flags**:
  - `-f`, `--force`: Skips confirmation prompt.

## Pruning Unused Docker Objects
```bash
docker system prune -a -f --volumes
```
- **Description**: Removes all unused Docker objects, including containers, networks, images, and optionally volumes.
- **General Expression**: `docker system prune [flag] [--volumes]`
- **Flags**:
  - `-a`, `--all`: Removes all unused images.
  - `-f`, `--force`: Skips confirmation prompt.
  - `--volumes`: Removes all unused volumes.

## Force Removing All Docker Objects
```bash
docker system prune -a --force --volumes
```
- **Description**: Forcefully removes all stopped containers, unused networks, dangling images, and unused volumes.
- **General Expression**: `docker system prune [flag] [--volumes]`
- **Flags**:
  - `-a`, `--all`: Removes all unused images.
  - `--force`: Skips confirmation prompt.
  - `--volumes`: Removes all unused volumes.

## Force Removing All Unused Resources by ID
```bash
docker rmi $(docker images -q) -f && docker container prune -f && docker network prune -f && docker volume prune -f
```
- **Description**: A comprehensive cleanup command to forcefully remove all images, containers, networks, and volumes by ID.
- **General Expression**: `docker rmi $(docker images -q) [flag] && docker container prune [flag] && docker network prune [flag] && docker volume prune [flag]`

---

### General Command Expressions

- **Container**: `<container>` – Represents the name or ID of a container.
- **Image**: `<image>` – Represents the name or ID of an image.
- **Network**: `<network>` – Represents the name or ID of a network.
- **Volume**: `<volume>` – Represents the name or ID of a volume.
- **Flag**: `[flag]` – Represents an optional flag, e.g., `-f`, `-a`, `--volumes`.

### Notes:

- **Force Flags**: The `-f` or `--force` flag should be used with caution, as it will remove resources without asking for confirmation.
- **Prune Commands**: Prune commands are valuable for cleaning up unused resources, freeing up disk space, and maintaining a tidy Docker environment.