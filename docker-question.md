# CKAD Practice Questions: Container Management

---

## **Overview**
Container management is a crucial skill for Kubernetes Application Developers. This section focuses on building, managing, and troubleshooting containers using tools like Docker and Podman.

---

## **Topics Covered**
- Build: Create container images from Dockerfiles or Podmanfiles.
- Run: Start containers with specific configurations.
- Pull: Download images from remote repositories.
- Push: Upload container images to registries.
- Save: Export container images to local files.
- Debug: Inspect and debug running containers or failed builds.
- Remove: Delete containers, images, and volumes.

---

## **Practice Questions**

### 1. Build and Push a Docker Image to a Container Registry

**Scenario**:
- Create a simple Dockerfile for an NGINX web server.
- Build the Docker image and push it to a container registry (Docker Hub or a private registry).
- Use both Docker and Podman to complete the tasks.

<details>
<summary>Details</summary>

#### Dockerfile

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
```

#### Steps Using Docker

1. Build the image:
   ```bash
   docker build -t my-nginx:1.0 .
   ```
2. Tag the image for Docker Hub:
   ```bash
   docker tag my-nginx:1.0 <your-dockerhub-username>/my-nginx:1.0
   ```
3. Push the image to Docker Hub:
   ```bash
   docker push <your-dockerhub-username>/my-nginx:1.0
   ```

#### Steps Using Podman

1. Build the image:
   ```bash
   podman build -t my-nginx:1.0 .
   ```
2. Tag the image for Docker Hub:
   ```bash
   podman tag my-nginx:1.0 <your-dockerhub-username>/my-nginx:1.0
   ```
3. Push the image to Docker Hub:
   ```bash
   podman push <your-dockerhub-username>/my-nginx:1.0
   ```

</details>

---

### 2. Pull and Save a Docker Image Locally

**Scenario**:
- Pull the official `busybox` image.
- Save it locally as a `.tar` file for future use.

<details>
<summary>Details</summary>

#### Steps Using Docker

1. Pull the image:
   ```bash
   docker pull busybox:latest
   ```
2. Save the image as a `.tar` file:
   ```bash
   docker save busybox:latest -o busybox.tar
   ```

#### Steps Using Podman

1. Pull the image:
   ```bash
   podman pull docker.io/library/busybox:latest
   ```
2. Save the image as a `.tar` file:
   ```bash
   podman save -o busybox.tar docker.io/library/busybox:latest
   ```

</details>

---

### 3. Load a Saved Image into Your Local Environment

**Scenario**:
- Load the previously saved `busybox.tar` image into your local environment.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Load the image:
   ```bash
   docker load -i busybox.tar
   ```
2. Verify the image is loaded:
   ```bash
   docker images | grep busybox
   ```

#### Steps Using Podman

1. Load the image:
   ```bash
   podman load -i busybox.tar
   ```
2. Verify the image is loaded:
   ```bash
   podman images | grep busybox
   ```

</details>

---

### 4. Remove a Docker Image Locally

**Scenario**:
- Remove the `busybox` image from your local environment.

<details>
<summary>Details</summary>

#### Steps Using Docker

1. Remove the image:
   ```bash
   docker rmi busybox:latest
   ```
2. Verify the image is removed:
   ```bash
   docker images | grep busybox
   ```

#### Steps Using Podman

1. Remove the image:
   ```bash
   podman rmi docker.io/library/busybox:latest
   ```
2. Verify the image is removed:
   ```bash
   podman images | grep busybox
   ```

</details>

---

### 5. Retag an Existing Docker Image

**Scenario**:
- Retag the `busybox` image with a new tag and push it to a private container registry.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Pull the image (if not already pulled):
   ```bash
   docker pull busybox:latest
   ```
2. Retag the image:
   ```bash
   docker tag busybox:latest <private-registry-url>/busybox:custom-tag
   ```
3. Push the image:
   ```bash
   docker push <private-registry-url>/busybox:custom-tag
   ```

#### Steps Using Podman

1. Pull the image (if not already pulled):
   ```bash
   podman pull docker.io/library/busybox:latest
   ```
2. Retag the image:
   ```bash
   podman tag docker.io/library/busybox:latest <private-registry-url>/busybox:custom-tag
   ```
3. Push the image:
   ```bash
   podman push <private-registry-url>/busybox:custom-tag
   ```

</details>

---

### 6. Clean Up Unused Images and Containers

**Scenario**:
- Identify and remove unused images and stopped containers from your environment.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Remove unused images:
   ```bash
   docker image prune -f
   ```
2. Remove stopped containers:
   ```bash
   docker container prune -f
   ```

#### Steps Using Podman

1. Remove unused images:
   ```bash
   podman image prune -f
   ```
2. Remove stopped containers:
   ```bash
   podman container prune -f
   ```

</details>

---

### 7. Build an Image with Multiple Tags

**Scenario**:
- Build a Docker image and assign it multiple tags during the build process.

<details>
<summary>Details</summary>


#### Dockerfile

```dockerfile
FROM alpine:latest
RUN echo "Multi-tag example"
```

#### Steps Using Docker

1. Build the image with multiple tags:
   ```bash
   docker build -t my-app:latest -t my-app:v1.0 .
   ```
2. Verify the tags:
   ```bash
   docker images | grep my-app
   ```

#### Steps Using Podman

1. Build the image with multiple tags:
   ```bash
   podman build -t my-app:latest -t my-app:v1.0 .
   ```
2. Verify the tags:
   ```bash
   podman images | grep my-app
   ```

</details>

---

### 8. Inspect Metadata of a Docker/Podman Image

**Scenario**:
- Inspect the metadata of the `nginx:latest` image and extract details such as the image ID, size, and environment variables.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Pull the image (if not already pulled):
   ```bash
   docker pull nginx:latest
   ```
2. Inspect the image:
   ```bash
   docker inspect nginx:latest
   ```
3. Extract specific metadata (e.g., size):
   ```bash
   docker inspect nginx:latest --format='{{.Size}}'
   ```

#### Steps Using Podman

1. Pull the image (if not already pulled):
   ```bash
   podman pull docker.io/library/nginx:latest
   ```
2. Inspect the image:
   ```bash
   podman inspect docker.io/library/nginx:latest
   ```
3. Extract specific metadata (e.g., size):
   ```bash
   podman inspect docker.io/library/nginx:latest --format='{{.Size}}'
   ```

</details>

---

### 9. Run a Temporary Container for Testing

**Scenario**:
- Run a temporary container using the `alpine` image to execute the `ping` command.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Pull the image (if not already pulled):
   ```bash
   docker pull alpine:latest
   ```
2. Run the container interactively:
   ```bash
   docker run --rm -it alpine:latest ping -c 4 google.com
   ```

#### Steps Using Podman

1. Pull the image (if not already pulled):
   ```bash
   podman pull docker.io/library/alpine:latest
   ```
2. Run the container interactively:
   ```bash
   podman run --rm -it docker.io/library/alpine:latest ping -c 4 google.com
   ```

</details>

---

### 10. Export a Running Container’s Filesystem

**Scenario**:
- Run a container using the `ubuntu` image, then export its filesystem to a `.tar` file.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Run the container:
   ```bash
   docker run -d --name ubuntu-container ubuntu:latest tail -f /dev/null
   ```
2. Export the filesystem:
   ```bash
   docker export ubuntu-container > ubuntu-container.tar
   ```

#### Steps Using Podman

1. Run the container:
   ```bash
   podman run -d --name ubuntu-container docker.io/library/ubuntu:latest tail -f /dev/null
   ```
2. Export the filesystem:
   ```bash
   podman export ubuntu-container > ubuntu-container.tar
   ```

</details>

---

### 11. Import a Filesystem as a New Image

**Scenario**:
- Import the exported `.tar` file as a new image named `custom-ubuntu`.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Import the filesystem:
   ```bash
   cat ubuntu-container.tar | docker import - custom-ubuntu:latest
   ```
2. Verify the image:
   ```bash
   docker images | grep custom-ubuntu
   ```

#### Steps Using Podman

1. Import the filesystem:
   ```bash
   cat ubuntu-container.tar | podman import - custom-ubuntu:latest
   ```
2. Verify the image:
   ```bash
   podman images | grep custom-ubuntu
   ```

</details>

---

### 12. Tag and Push an Image to a Private Registry

**Scenario**:
- Retag the `nginx` image and push it to a private registry running at `myregistry.local`.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Pull the image (if not already pulled):
   ```bash
   docker pull nginx:latest
   ```
2. Tag the image:
   ```bash
   docker tag nginx:latest myregistry.local/nginx:latest
   ```
3. Push the image:
   ```bash
   docker push myregistry.local/nginx:latest
   ```

#### Steps Using Podman

1. Pull the image (if not already pulled):
   ```bash
   podman pull docker.io/library/nginx:latest
   ```
2. Tag the image:
   ```bash
   podman tag docker.io/library/nginx:latest myregistry.local/nginx:latest
   ```
3. Push the image:
   ```bash
   podman push myregistry.local/nginx:latest
   ```

</details>

---

### 13. Prune Unused Images and Containers

**Scenario**:
- Remove all unused images and stopped containers from your environment.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Prune unused images:
   ```bash
   docker image prune -a -f
   ```
2. Prune stopped containers:
   ```bash
   docker container prune -f
   ```

#### Steps Using Podman

1. Prune unused images:
   ```bash
   podman image prune -a -f
   ```
2. Prune stopped containers:
   ```bash
   podman container prune -f
   ```

</details>

---

### 14. Run a Detached Container with Port Forwarding

**Scenario**:
- Run an NGINX container in detached mode and forward port 8080 on the host to port 80 in the container.

<details>
<summary>Details</summary>


#### Steps Using Docker

1. Run the container:
   ```bash
   docker run -d -p 8080:80 nginx:latest
   ```
2. Verify access:
   ```bash
   curl http://localhost:8080
   ```

#### Steps Using Podman

1. Run the container:
   ```bash
   podman run -d -p 8080:80 docker.io/library/nginx:latest
   ```
2. Verify access:
   ```bash
   curl http://localhost:8080
   ```

</details>

---

### 15. Use Build Arguments in a Dockerfile

**Scenario**:
- Create a Dockerfile for an Alpine-based image that accepts a build argument for a custom message.

<details>
<summary>Details</summary>


#### Dockerfile

```dockerfile
FROM alpine:latest
ARG CUSTOM_MSG="Default Message"
RUN echo "$CUSTOM_MSG" > /message.txt
CMD cat /message.txt
```

#### Steps Using Docker

1. Build the image with a custom argument:
   ```bash
   docker build -t custom-alpine:latest --build-arg CUSTOM_MSG="Hello CKAD" .
   ```
2. Run the container:
   ```bash
   docker run --rm custom-alpine:latest
   ```

#### Steps Using Podman

1. Build the image with a custom argument:
   ```bash
   podman build -t custom-alpine:latest --build-arg CUSTOM_MSG="Hello CKAD" .
   ```
2. Run the container:
   ```bash
   podman run --rm custom-alpine:latest
   ```

</details>

---

## **Notes and Tips**
### **Build**
- Use `docker build` or `podman build` to create images.
- Optimize image layers by ordering commands effectively in Dockerfiles.

### **Run**
- Start containers with `docker run` or `podman run`.
- Add flags for configuration:
  - `-d`: Run in detached mode.
  - `-p`: Map ports.
  - `--name`: Assign a name to the container.

### **Pull & Push**
- Pull images with `docker pull <image>` or `podman pull <image>`.
- Push images to a registry using `docker push <image>` or `podman push <image>`.

### **Save**
- Save images locally with `docker save <image> -o <filename>` or `podman save`.

### **Debug**
- Use commands like `docker logs <container>` or `podman logs` for debugging.
- Inspect container metadata with `docker inspect <container>` or `podman inspect`.

### **Remove**
- Delete images or containers:
  - `docker rmi <image>` or `podman rmi` for images.
  - `docker rm <container>` or `podman rm` for containers.

---

## **Resources**
1. [Docker Documentation](https://docs.docker.com/)
2. [Podman Documentation](https://podman.io/)
3. [Kubernetes Official Documentation](https://kubernetes.io/docs/)
