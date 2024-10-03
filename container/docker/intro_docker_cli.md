# A Comprehensive Guide to Docker CLI Commands

Docker is an open platform for developing, shipping, and running applications. With the **Docker CLI** (Command Line Interface), developers can interact with Docker to build, manage, and orchestrate containers and images. This article provides a comprehensive overview of the **base commands** for the Docker CLI, covering essential tasks such as managing containers, images, networks, and volumes. Practical examples are provided to demonstrate the use of each command.

---

## 1. **docker version**

The `docker version` command displays detailed information about the Docker client and server (daemon) versions.

### Example:
```bash
docker version
```

This will output something like:

```
Client: Docker Engine - Community
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        f0df350
 OS/Arch:           linux/amd64
...
```

**Use Case**: This command is helpful when troubleshooting or verifying compatibility between different versions of Docker in a team.

---

## 2. **docker info**

The `docker info` command provides system-wide information about Docker, including details about the number of containers, images, the storage driver, and network settings.

### Example:
```bash
docker info
```

Output:

```
Containers: 3
 Running: 1
 Paused: 0
 Stopped: 2
Images: 7
Server Version: 20.10.7
...
```

**Use Case**: Use this command to gain a complete overview of the Docker environment, which is helpful when diagnosing problems or verifying the state of the system.

---

## 3. **docker pull**

The `docker pull` command is used to download a Docker image from a remote registry (usually Docker Hub) to your local machine.

### Example:
```bash
docker pull nginx
```

This command will pull the latest version of the **Nginx** image from Docker Hub. You can also specify a tag (version) if required:

```bash
docker pull nginx:1.19
```

**Use Case**: This command is used to fetch images that can later be run as containers. You can also pull images from private registries by providing the registry URL.

---

## 4. **docker build**

The `docker build` command creates a Docker image from a Dockerfile. This command is critical for automating the creation of images for specific applications.

### Example:
```bash
docker build -t my-app .
```

- `-t my-app`: Tags the image as `my-app`.
- `.`: Specifies the build context (the current directory, which contains the Dockerfile).

**Use Case**: Use `docker build` to create custom Docker images tailored to your application's environment.

---

## 5. **docker images**

The `docker images` command lists all Docker images currently available on your machine.

### Example:
```bash
docker images
```

Output:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              4bb46517cac3        3 days ago          133MB
my-app              v1.0                0d3f60f8f70b        1 week ago          221MB
```

**Use Case**: This command is useful for checking which images you have locally, their tags, and their sizes.

---

## 6. **docker run**

The `docker run` command creates and starts a new container from a specified image. It is one of the most commonly used Docker commands.

### Example:
```bash
docker run -d -p 8080:80 nginx
```

- `-d`: Runs the container in detached mode (in the background).
- `-p 8080:80`: Maps port 8080 on the host to port 80 on the container (useful for web services).
- `nginx`: The image from which to create the container.

To run a container interactively (e.g., to execute commands inside the container):

```bash
docker run -it ubuntu bash
```

- `-it`: Runs the container interactively with a TTY (allowing you to interact with the container via a shell).
- `ubuntu bash`: Runs an interactive `bash` shell inside an Ubuntu container.

**Use Case**: Use `docker run` to start containers from images, mapping ports, and running commands as needed.

---

## 7. **docker ps**

The `docker ps` command lists running containers. To include all containers (including stopped ones), add the `-a` option.

### Example:
```bash
docker ps
```

Output:

```
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                  NAMES
f65d5fa98d54   nginx       "nginx -g 'daemon of…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp   brave_shannon
```

To list all containers (including stopped ones):

```bash
docker ps -a
```

**Use Case**: `docker ps` is essential for checking which containers are currently running and for viewing their status.

---

## 8. **docker stop**

The `docker stop` command stops a running container gracefully by sending a **SIGTERM** signal, allowing the container to terminate its processes.

### Example:
```bash
docker stop <container_id_or_name>
```

You can stop multiple containers at once by listing their IDs or names:

```bash
docker stop container1 container2
```

**Use Case**: This command is used to gracefully stop running containers.

---

## 9. **docker rm**

The `docker rm` command removes stopped containers. You cannot remove running containers with this command unless you force it with the `-f` option.

### Example:
```bash
docker rm <container_id_or_name>
```

To remove all stopped containers:

```bash
docker container prune
```

**Use Case**: Use this command to clean up stopped containers and free system resources.

---

## 10. **docker rmi**

The `docker rmi` command removes Docker images from your local system. If an image is in use by a running container, you cannot remove it unless you force it with the `-f` option.

### Example:
```bash
docker rmi <image_id>
```

To remove multiple images:

```bash
docker rmi image1 image2
```

**Use Case**: Use `docker rmi` to remove unnecessary or old images, freeing up disk space.

---

## 11. **docker logs**

The `docker logs` command retrieves the logs from a running or stopped container.

### Example:
```bash
docker logs <container_id_or_name>
```

To continuously stream the logs (like `tail -f`):

```bash
docker logs -f <container_id_or_name>
```

**Use Case**: This command is useful for debugging or monitoring containers by inspecting their logs.

---

## 12. **docker exec**

The `docker exec` command runs a new command inside an already running container. This is commonly used to get a shell inside a running container.

### Example:
```bash
docker exec -it <container_id_or_name> /bin/bash
```

This example opens an interactive bash shell inside a running container. The `-it` option is used to keep the shell interactive.

**Use Case**: Use `docker exec` to run commands or interact with a running container without restarting it.

---

## 13. **docker cp**

The `docker cp` command copies files between the host machine and a container.

### Example:
To copy a file from the container to the host:
```bash
docker cp <container_id_or_name>:/path/in/container /path/on/host
```

To copy a file from the host to the container:
```bash
docker cp /path/on/host <container_id_or_name>:/path/in/container
```

**Use Case**: Use this command to retrieve logs, configuration files, or any other data from inside a running or stopped container.

---

## 14. **docker commit**

The `docker commit` command creates a new image from a container's changes. This is useful for capturing the current state of a container as an image.

### Example:
```bash
docker commit <container_id_or_name> new-image-name
```

**Use Case**: Use `docker commit` to save the current state of a container as a new image, useful when you've made changes inside a container that you want to preserve.

---

## 15. **docker network**

Docker uses networks to enable communication between containers. You can manage Docker networks using the `docker network` command.

- **List networks**:
  ```bash
  docker network ls
  ```

- **Create a network**:
  ```bash
  docker network create my-network
  ```

- **Connect a container to a network**:
  ```bash
  docker network connect my-network <container_id_or_name>
  ```

**Use Case**: `docker network` is used to create and manage isolated networks for containers, allowing them to communicate with each other securely.

---

## 16. **docker volume**

Docker volumes are used to persist data generated by and used by Docker containers. The `docker volume` command allows you to create and manage these volumes.

- **List volumes**:
  ```bash
  docker volume ls
  ```

- **Create a volume**:
  ```bash
  docker volume create my-volume
  ```

- **Attach a volume to a container**:
  ```bash
  docker run -v my-volume:/data my-app
  ```

**Use Case**

: Use Docker volumes to persist data beyond the life cycle of a container, ensuring that important data is not lost when containers are stopped or removed.

---

## Conclusion

The Docker CLI offers a wide range of commands that simplify managing containers, images, networks, and volumes. From basic tasks like pulling images and running containers to more advanced operations like creating custom networks and persisting data with volumes, these commands are essential for working effectively with Docker.

By mastering these base Docker CLI commands, you can efficiently manage and orchestrate your containerized applications, making it easier to build, run, and deploy your apps across different environments.

