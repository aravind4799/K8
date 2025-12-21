# Docker

## Table of Contents

- [Dockerfile](#dockerfile)
- [Building Docker Images](#building-docker-images)
- [Pushing Images to Docker Hub](#pushing-images-to-docker-hub)
- [Layered Caching](#layered-caching)
- [Basic Docker Commands](#basic-docker-commands)
- [Running Containers](#running-containers)
- [Finding Base Image Information](#finding-base-image-information)
- [Commands and Arguments](#commands-and-arguments)
  - [Container Lifecycle](#container-lifecycle)
  - [Default Container Behavior](#default-container-behavior)
  - [Overriding Commands at Runtime](#overriding-commands-at-runtime)
  - [CMD Instruction](#cmd-instruction)
  - [ENTRYPOINT Instruction](#entrypoint-instruction)
  - [Combining CMD and ENTRYPOINT](#combining-cmd-and-entrypoint)
  - [Overriding ENTRYPOINT](#overriding-entrypoint)

---

## Dockerfile

*A Dockerfile is a text file that contains a set of instructions in command format. Each instruction tells Docker how to build a layer in your image. Dockerfiles provide a blueprint for creating Docker images.*

**Example Dockerfile:**
```dockerfile
FROM Ubuntu
RUN apt-get update && apt-get -y install python
RUN pip install flask flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

**Common Dockerfile Instructions:**
- **`FROM`** - Specifies the base image to start from
- **`RUN`** - Executes commands during the build process
- **`COPY`** - Copies files from the host into the image
- **`ADD`** - Similar to COPY but can also handle URLs and extract archives
- **`ENTRYPOINT`** - Sets the default command to run when the container starts
- **`CMD`** - Provides default arguments for the ENTRYPOINT or runs a command
- **`WORKDIR`** - Sets the working directory for subsequent instructions
- **`ENV`** - Sets environment variables
- **`EXPOSE`** - Documents which ports the container will listen on

## Building Docker Images

*Use the `docker build` command to create a Docker image from a Dockerfile.*

**Basic syntax:**
```bash
docker build Dockerfile -t <username>/<image-name>
```

**Example:**
```bash
docker build Dockerfile -t aravi/my-name
```

**Key points:**
- The `-t` flag tags the image with a name (and optionally a tag like `:v1.0`)
- You can specify the Dockerfile path (default is `Dockerfile` in the current directory)
- You can also use `docker build .` to build from the current directory (Docker automatically looks for a file named `Dockerfile`)

**Tagging with version:**
```bash
docker build -t aravi/my-name:v1.0 .
docker build -t aravi/my-name:latest .
```

## Pushing Images to Docker Hub

*After building an image, you can push it to a remote Docker registry like Docker Hub.*

**Push to Docker Hub:**
```bash
docker push aravi/my-name
```

**Full workflow:**
1. **Login to Docker Hub:**
   ```bash
   docker login
   ```
   *Enter your Docker Hub username and password*

2. **Build the image:**
   ```bash
   docker build -t aravi/my-name .
   ```

3. **Push the image:**
   ```bash
   docker push aravi/my-name
   ```

**Important Notes:**
- The image name must match your Docker Hub username (or organization) prefix
- Make sure you're logged in with `docker login` before pushing
- The image will be publicly available (unless you have a private repository)

## Layered Caching

*Docker uses a layered caching mechanism to speed up builds and efficiently manage image layers.*

**How it works:**
- Each instruction in a Dockerfile creates a new layer in the image
- Docker caches the result of each layer after it's successfully built
- When you rebuild an image, Docker checks each layer from top to bottom
- If a layer hasn't changed, Docker reuses the cached layer instead of rebuilding it
- If a particular step fails, Docker will reuse all the previous successful layers from the cache

**Example Dockerfile layers:**
```dockerfile
FROM Ubuntu                    # Layer 1: Base Ubuntu layer (120 MB)
RUN apt-get update            # Layer 2: Changes in apt packages (306 MB)
RUN pip install flask         # Layer 3: Changes in pip packages (6.3 MB)
COPY . /opt/source-code       # Layer 4: Source code (229 B)
ENTRYPOINT flask run          # Layer 5: Update Entrypoint (0 B)
```

**Benefits of layered caching:**
- **Faster builds** - Unchanged layers are reused, saving time and resources
- **Efficient storage** - Layers are shared between images when they use the same base layers
- **Incremental builds** - If a step fails, you can fix it and rebuild without redoing successful steps
- **Optimization strategy** - Place frequently changing instructions (like `COPY .`) at the end of the Dockerfile to maximize cache hits

**Example scenario:**
1. You build an image successfully
2. You make a small change to your source code
3. You rebuild the image
4. Docker reuses cached layers for `FROM`, `RUN`, and other unchanged instructions
5. Only the `COPY` and subsequent layers are rebuilt
6. This makes the rebuild much faster than building from scratch

**Best practices for caching:**
- Order Dockerfile instructions from least frequently changed to most frequently changed
- Place `COPY` commands that copy source code near the end (after dependency installations)
- Combine `RUN` commands when possible to reduce layer count
- Use `.dockerignore` to exclude unnecessary files from the build context

## Basic Docker Commands

**List running containers:**
```bash
docker ps
```
*Shows all currently running containers with their IDs, images, status, ports, and names.*

**List all containers (including stopped):**
```bash
docker ps -a
```
*Shows all containers, both running and stopped. Useful for seeing containers that have exited or were stopped.*

**List Docker images:**
```bash
docker images
```
*Shows all Docker images stored locally on your system, including their repository name, tags, image IDs, creation date, and size.*

## Running Containers

### Port Mapping (-p flag)

*The `-p` (or `--publish`) flag maps ports from the container to the host machine, allowing you to access services running inside the container from your host.*

**Syntax:**
```bash
docker run -p <host_port>:<container_port> <image_name>
```

**Example:**
```bash
docker run -p 8282:8080 aravi/my-name
```

**Explanation:**
- `<host_port>` (8282) - The port on your host machine where you want to access the service
- `<container_port>` (8080) - The port inside the container where the application is listening
- `<image_name>` - The image name or tag that was specified with `-t` during the build process

*In this example, if your application inside the container listens on port 8080, you can access it from your host machine at `localhost:8282`. Traffic from port 8282 on your host will be forwarded to port 8080 inside the container.*

**Multiple port mappings:**
```bash
docker run -p 8282:8080 -p 3000:3000 aravi/my-name
```
*You can map multiple ports by using multiple `-p` flags.*

### Volumes (-v flag)

*The `-v` (or `--volume`) flag mounts a directory from the host machine into the container, enabling persistent storage. When the Docker container exits or is removed, the data stored in the volume persists on the host machine.*

**Syntax:**
```bash
docker run -v <host_directory>:<container_directory> <image_name>
```

**Example:**
```bash
docker run -v /host/data:/container/data aravi/my-name
```

**Explanation:**
- `<host_directory>` - The directory on your host machine (e.g., `/host/data` or `./data` for current directory)
- `<container_directory>` - The directory inside the container where you want to mount the host directory (e.g., `/container/data` or `/app/data`)
- This creates a bind mount that links the host directory to the container directory

**Why use volumes:**
- **Data persistence** - When the container is stopped or removed, data in the volume remains on the host
- **Data sharing** - Multiple containers can share the same volume
- **Backup and migration** - Since data is on the host, it's easier to backup and migrate
- **Performance** - Direct access to host filesystem can be faster than container filesystem

**Example use case:**
```bash
# Run a database container with persistent storage
docker run -v /my/host/data:/var/lib/mysql mysql:latest

# Even if the container stops or is removed, the database data remains in /my/host/data on your host machine
```

**Using named volumes (Docker-managed):**
```bash
docker run -v my-volume:/container/data aravi/my-name
```
*Docker manages the volume location. You can list volumes with `docker volume ls` and inspect them with `docker volume inspect my-volume`.*

## Finding Base Image Information

*You can run a command in a container to inspect the base image or OS information without starting a long-running container.*

**Example:**
```bash
docker run python:3.6 cat /etc/*release*
```

**Explanation:**
- `docker run python:3.6` - Runs a container from the `python:3.6` image
- `cat /etc/*release*` - Executes a command that displays OS release information
- The container starts, runs the command, displays the output, and then exits

*This is useful for:*
- Finding out which Linux distribution an image is based on
- Checking OS version information
- Inspecting system configuration without needing to interactively enter the container
- Understanding what base image was used in a Dockerfile

**Other useful inspection commands:**
```bash
# Check OS version
docker run python:3.6 cat /etc/os-release

# List installed packages (Debian/Ubuntu)
docker run python:3.6 dpkg -l

# Check kernel version
docker run python:3.6 uname -a
```

*These commands run and exit immediately, making them perfect for quick inspections without leaving containers running.*

## Commands and Arguments

### Container Lifecycle

*A Docker container lives only for as long as the process inside it is running. Once the main process exits, the container stops.*

**Example:**
```bash
docker ps
```
*If no containers are running, this shows nothing. Containers exit when their main process completes or terminates.*

**Key concept:**
- Containers are not virtual machines - they're processes
- When the process exits, the container stops
- A stopped container appears in `docker ps -a` but not in `docker ps`

### Default Container Behavior

**Running Ubuntu container:**
```bash
docker run ubuntu
```

**What happens:**
- The Ubuntu image has a default `CMD ["bash"]` instruction
- When you run the container, it tries to start the bash program
- Bash looks for a terminal (TTY) to attach itself to
- The Docker container doesn't have an interactive terminal by default
- The bash process immediately exits because it can't find a terminal
- **The container exits immediately**

**This is why:**
```bash
docker ps
# Shows nothing - container has exited
```

### Overriding Commands at Runtime

*You can override the default command when running a container by specifying a command after the image name.*

**Example:**
```bash
docker run ubuntu sleep 10
```

*This runs the `sleep 10` command instead of the default bash command. The container will:*
1. Start
2. Run `sleep 10`
3. Wait for 10 seconds
4. Exit when sleep completes

*This works, but you have to specify the command every time you run the container.*

### CMD Instruction

*The `CMD` instruction in a Dockerfile sets the default command that will run when the container starts. You can override it at runtime by providing a command.*

**Syntax options:**
```dockerfile
# Shell form
CMD sleep 5

# JSON array form (recommended)
CMD ["sleep", "5"]
```

**Example Dockerfile:**
```dockerfile
FROM ubuntu
CMD ["sleep", "5"]
```

**Building and running:**
```bash
docker build -t ubuntu-sleeper .
docker run ubuntu-sleeper
```

*The container will sleep for 5 seconds and then exit.*

**Overriding CMD at runtime:**
```bash
docker run ubuntu-sleeper sleep 10
```

*This overrides the default `sleep 5` with `sleep 10`. The container will sleep for 10 seconds instead.*

**Key behavior with CMD:**
- **CMD gets completely replaced** when you provide a command at runtime
- Whatever you specify after the image name replaces the entire CMD instruction
- `docker run ubuntu-sleeper sleep 10` replaces `["sleep", "5"]` with `sleep 10`

### ENTRYPOINT Instruction

*The `ENTRYPOINT` instruction sets the main command that will always run. Unlike CMD, arguments provided at runtime get **appended** to the ENTRYPOINT, not replaced.*

**Syntax:**
```dockerfile
ENTRYPOINT ["sleep"]
```

**Example Dockerfile:**
```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
```

**Running with argument:**
```bash
docker run ubuntu-sleeper 10
```

*This appends `10` as an argument to `sleep`, effectively running `sleep 10`.*

**Key behavior with ENTRYPOINT:**
- **ENTRYPOINT appends arguments** - whatever you provide gets added to the entrypoint command
- `docker run ubuntu-sleeper 10` becomes `sleep 10`
- `docker run ubuntu-sleeper 20` becomes `sleep 20`

**What happens if you don't provide an argument:**
```bash
docker run ubuntu-sleeper
```

*This would run just `sleep` with no arguments, resulting in an error: "operand is missing" (sleep requires a number).*

### Combining CMD and ENTRYPOINT

*You can use both `ENTRYPOINT` and `CMD` together. ENTRYPOINT provides the base command, and CMD provides default arguments. If you provide arguments at runtime, they replace the CMD default arguments.*

**Example Dockerfile:**
```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

**Important:** Both must be in JSON array format for this to work correctly.

**Running without arguments:**
```bash
docker run ubuntu-sleeper
```
*This uses the default CMD value, running `sleep 5`.*

**Running with custom argument:**
```bash
docker run ubuntu-sleeper 10
```
*This replaces the CMD default (`5`) with `10`, running `sleep 10`.*

**How it works:**
- `ENTRYPOINT ["sleep"]` - Base command (always runs)
- `CMD ["5"]` - Default argument (can be overridden)
- Combined: `sleep 5` (by default)
- With override: `sleep 10` (when you provide `10`)

**Summary:**
- **ENTRYPOINT** = The command that always runs (base command)
- **CMD** = Default arguments that can be overridden
- Runtime arguments replace CMD, but ENTRYPOINT remains

### Overriding ENTRYPOINT

*To override both ENTRYPOINT and CMD, use the `--entrypoint` flag.*

**Syntax:**
```bash
docker run --entrypoint <new_command> <image_name> <arguments>
```

**Example:**
```bash
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```

*This:*
- Overrides the ENTRYPOINT from `sleep` to `sleep2.0`
- Passes `10` as an argument to `sleep2.0`
- Effectively runs: `sleep2.0 10`

**Use cases:**
- When you need to completely change the entrypoint command
- For debugging or testing different entry points
- When the default entrypoint doesn't work for your use case

**Summary table:**

| Instruction | Runtime Override Behavior | Example |
|------------|---------------------------|---------|
| `CMD ["sleep", "5"]` | Completely replaced | `docker run image sleep 10` → runs `sleep 10` |
| `ENTRYPOINT ["sleep"]` | Arguments appended | `docker run image 10` → runs `sleep 10` |
| `ENTRYPOINT ["sleep"]` + `CMD ["5"]` | CMD replaced, ENTRYPOINT kept | `docker run image 10` → runs `sleep 10` |
| `--entrypoint` flag | Both overridden | `docker run --entrypoint cmd image arg` → runs `cmd arg` |

### Real-World Example: Nginx

**Dockerfile for Nginx:**
```dockerfile
FROM ubuntu
CMD ["nginx"]
```

*When you run `docker run nginx-image`, it starts the nginx server. The container stays running as long as nginx is running.*

**Container lifecycle:**
- Container starts → nginx starts
- Container runs → nginx runs (serving web requests)
- Container stops → when nginx process stops or is terminated
