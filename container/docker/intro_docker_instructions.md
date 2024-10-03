# Understanding Dockerfile Instructions: A Complete Guide with Examples

A **Dockerfile** is the blueprint for creating Docker images, which are essential for running containerized applications. Each line in a Dockerfile is an **instruction** that defines a step for building the image, from defining the base image to configuring the application’s runtime environment. Understanding these instructions is crucial for building efficient, optimized, and secure Docker images.

In this article, we'll break down the most commonly used Dockerfile instructions, explaining their purpose and providing examples to illustrate how they work.

---

## 1. **FROM**: Defining the Base Image

The `FROM` instruction sets the **base image** for your Docker image. It is typically the first line in the Dockerfile. All subsequent instructions are applied on top of this base image. You can pull base images from public registries like **Docker Hub** or use custom images.

### Example:

```Dockerfile
FROM node:14-alpine
```

This example pulls the official **Node.js** image with the version 14, using the lightweight **Alpine** variant. The **Alpine** version reduces the size of the image, making it ideal for production.

---

## 2. **WORKDIR**: Setting the Working Directory

The `WORKDIR` instruction sets the **working directory** inside the container where subsequent commands will be run. If the directory does not exist, it will be created automatically.

### Example:

```Dockerfile
WORKDIR /app
```

Here, the working directory inside the container is set to `/app`. Any subsequent instructions that reference files (e.g., `COPY`, `RUN`, or `CMD`) will use this directory as the current path.

---

## 3. **COPY**: Copying Files into the Image

The `COPY` instruction copies files or directories from the host system (where you're building the image) into the container image.

### Example:

```Dockerfile
COPY ./src /app/src
```

In this example, all the files in the local `./src` directory are copied into the `/app/src` directory inside the container.

---

## 4. **ADD**: Adding Files and Archives

The `ADD` instruction is similar to `COPY`, but with additional functionality:
- It can **extract compressed files** (like `.tar` files).
- It can **download files** from URLs.

However, in most cases, **COPY** is preferred because it is more predictable and limited in scope.

### Example:

```Dockerfile
ADD https://example.com/app.tar.gz /app
```

This example downloads a `.tar.gz` file from a URL and extracts it into the `/app` directory in the container.

---

## 5. **RUN**: Executing Commands at Build Time

The `RUN` instruction executes commands inside the container at **build time**. It is commonly used to install software packages or run scripts.

### Example:

```Dockerfile
RUN apt-get update && apt-get install -y curl
```

This example updates the package lists and installs the **curl** utility in the container.

You can chain multiple commands together with `&&` to minimize the number of `RUN` instructions, which helps reduce the number of layers in the final image.

---

## 6. **CMD**: Setting the Default Command to Run

The `CMD` instruction defines the **default command** that runs when a container is started from the image. If an overriding command is passed during `docker run`, it will replace the `CMD`.

### Example:

```Dockerfile
CMD ["node", "app.js"]
```

In this example, the container will execute the `node app.js` command when it starts. If someone runs `docker run <image> bash`, it will override `CMD` and run `bash` instead.

**Note**: Only one `CMD` instruction is allowed in a Dockerfile. If there are multiple `CMD` instructions, only the last one will be used.

---

## 7. **ENTRYPOINT**: Configuring the Main Executable

The `ENTRYPOINT` instruction specifies a command that always runs when the container starts, regardless of any additional commands passed to `docker run`. This makes it different from `CMD`, which can be overridden.

### Example:

```Dockerfile
ENTRYPOINT ["python", "app.py"]
```

Here, `python app.py` will always be executed when the container starts. Any additional arguments passed with `docker run` will be appended to the `ENTRYPOINT`.

### Combined with `CMD`:

You can combine `ENTRYPOINT` with `CMD` to pass default arguments that can be overridden.

```Dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
```

In this setup:
- `python app.py --port 8000` will run by default.
- If the user runs `docker run <image> --port 9000`, it will override the port argument.

---

## 8. **EXPOSE**: Exposing Ports

The `EXPOSE` instruction is used to **document** which ports the application inside the container listens on. It does not actually publish the port to the host machine. For that, you need to use the `-p` flag when running the container.

### Example:

```Dockerfile
EXPOSE 8080
```

This indicates that the application inside the container is expected to listen on port `8080`. To make this port accessible to the host, you would run the container with `docker run -p 8080:8080`.

---

## 9. **ENV**: Setting Environment Variables

The `ENV` instruction sets **environment variables** inside the container, which can be accessed by the application at runtime. These variables are persisted across all subsequent layers and commands.

### Example:

```Dockerfile
ENV NODE_ENV=production
ENV APP_PORT=3000
```

This sets two environment variables, `NODE_ENV` and `APP_PORT`, which can be referenced within the container during runtime.

---

## 10. **VOLUME**: Defining Volumes for Persistent Data

The `VOLUME` instruction creates a mount point with a specific path that can be used to **persist data** outside the container. This data will survive container restarts, and multiple containers can share the same volume.

### Example:

```Dockerfile
VOLUME /app/data
```

This defines `/app/data` as a mount point inside the container, where external data can be stored.

---

## 11. **ARG**: Defining Build-Time Variables

The `ARG` instruction allows you to define **build-time variables** that can be passed to the Dockerfile during the build process. These variables are not available during runtime like environment variables (`ENV`).

### Example:

```Dockerfile
ARG NODE_VERSION=14
FROM node:${NODE_VERSION}-alpine
```

In this example, the `NODE_VERSION` variable is set to `14`, and it is used in the `FROM` instruction to dynamically pull the corresponding Node.js version. You can pass build arguments when building the image using:

```bash
docker build --build-arg NODE_VERSION=16 -t my-node-app .
```

This command will build the image with Node.js version 16 instead of the default 14.

---

## 12. **USER**: Running as a Non-Root User

By default, containers run as the **root** user, which can pose security risks. The `USER` instruction allows you to specify a **non-root user** to run the container, improving security.

### Example:

```Dockerfile
# Create a new user with minimal privileges
RUN adduser --disabled-password --gecos "" appuser

# Switch to the new user
USER appuser
```

This example creates a non-root user called `appuser` and switches to that user for the rest of the Dockerfile. The application will run as `appuser` instead of root when the container starts.

---

## 13. **HEALTHCHECK**: Monitoring Container Health

The `HEALTHCHECK` instruction tells Docker how to test if a container is still functioning correctly. If the check fails, Docker can restart the container or notify an external system.

### Example:

```Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \
  CMD curl --fail http://localhost:8080/health || exit 1
```

This example checks if the application running inside the container is healthy by sending a request to the `/health` endpoint every 30 seconds. If the request fails, the container will be marked as unhealthy.

---

## 14. **STOPSIGNAL**: Defining the Stop Signal

The `STOPSIGNAL` instruction sets the system signal used to stop the container. By default, Docker sends the `SIGTERM` signal when stopping a container, but you can customize this to use other signals like `SIGKILL` or `SIGINT`.

### Example:

```Dockerfile
STOPSIGNAL SIGTERM
```

This tells Docker to use the `SIGTERM` signal to stop the container.

---

## 15. **ONBUILD**: Triggering Instructions in Child Images

The `ONBUILD` instruction sets up a trigger that will run when the image is used as the base for another image. This is useful for creating **base images** that perform specific actions when extended by other Dockerfiles.

### Example:

```Dockerfile
ONBUILD COPY . /app
ONBUILD RUN npm install
```

In this example, when another Dockerfile extends this image with a `FROM` instruction, the `COPY` and `RUN` instructions will automatically be executed.

---

## Complete Dockerfile Example

Below is a complete Dockerfile example that demonstrates many of the instructions covered in this article:

```Dockerfile
# Step 1: Use an official Node.js runtime as the base image
FROM node:14-alpine

# Step 2: Set the working directory inside the container
WORKDIR /app

# Step 3: Copy the application source code to the container
COPY package.json package-lock.json ./

# Step 4: Install only production dependencies
RUN npm install --only=production

# Step 5: Copy the rest of the source code to the working directory
COPY . .

# Step 6: Expose the application port
EXPOSE 3000

# Step 7: Set environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Step 8: Define a health check to monitor container health
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \
  CMD curl --fail http://localhost:3000/health || exit 1

# Step 9: Set the default command to run the application
CMD ["npm", "start"]
```

### Explanation:

1. **Base Image**: The base image is a lightweight Alpine version of Node.js.
2. **Working Directory**: The working directory is set to `/app`.
3. **COPY and RUN**: The `package.json` and `package-lock.json` are copied to the container, and only production dependencies are installed.
4. **Expose and ENV**: The application listens on port 3000, which is exposed. Environment variables are set to ensure the app runs in production mode.
5. **Health Check**: A health check is defined to ensure the application is healthy.
6. **CMD**: The `npm start` command is run when the container starts.

By combining these instructions, this Dockerfile creates a production-ready image that is efficient, secure, and well-optimized for deployment.