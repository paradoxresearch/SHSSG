# Getting Started with Podman

## Contents

- [Getting Started with Podman](#getting-started-with-podman)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Containerization and Podman](#understanding-containerization-and-podman)
      - [Containerization](#containerization)
      - [Podman and Daemonless Architecture](#podman-and-daemonless-architecture)
      - [The Concept of Pods in Podman](#the-concept-of-pods-in-podman)
  - [Installing Podman](#installing-podman)
  - [Basic Podman Commands](#basic-podman-commands)
    - [Image Management](#image-management)
      - [Searching for Container Images (`podman search`)](#searching-for-container-images-podman-search)
      - [Pulling a Container Image (`podman pull`)](#pulling-a-container-image-podman-pull)
      - [Listing Available Images (`podman images`)](#listing-available-images-podman-images)
      - [Removing Images (`podman rmi`)](#removing-images-podman-rmi)
    - [Container Lifecycle Management](#container-lifecycle-management)
      - [Running a Basic Container (`podman run`)](#running-a-basic-container-podman-run)
        - [Example: Running an interactive Ubuntu container](#example-running-an-interactive-ubuntu-container)
      - [List Running Containers (`podman ps` and `podman ps -a`)](#list-running-containers-podman-ps-and-podman-ps--a)
      - [Stopping and Starting Containers (`podman stop` and `podman start`)](#stopping-and-starting-containers-podman-stop-and-podman-start)
      - [Removing Containers (`podman rm`)](#removing-containers-podman-rm)
      - [Detached Mode (`-d`) and Port Mapping (`-p`)](#detached-mode--d-and-port-mapping--p)
  - [Working with Podman](#working-with-podman)
    - [Pods and Grouping Containers](#pods-and-grouping-containers)
    - [Creating a Pod (`podman pod create`)](#creating-a-pod-podman-pod-create)
    - [Adding Containers to a Pod (`--pod` options with `podman run`)](#adding-containers-to-a-pod---pod-options-with-podman-run)
      - [Example: Running a Web Server and a Database in a Pod](#example-running-a-web-server-and-a-database-in-a-pod)
    - [Listing Pods and Their Associated Containers](#listing-pods-and-their-associated-containers)
    - [Stopping and Removing Pods](#stopping-and-removing-pods)
  - [Practical Use Cases](#practical-use-cases)
    - [Common Scenarios for Podman](#common-scenarios-for-podman)

## Introduction

### Understanding Containerization and Podman

#### Containerization

At its core, **containerization** is a method of packaging an application and all its dependencies—such as libraries, frameworks, and configuration files—into a single, isolated unit called a container. Think of it like this: if you were to move your entire office to a new location, instead of packing individual desks, chairs, and computers separately, you'd put each complete workstation into its own self-contained, portable box. This box would have everything needed for that workstation to function immediately upon arrival.

The key benefits of containerization are:

- **Consistency**: Applications run the same way regardless of the environment (your laptop, a development server, a production server). This eliminates the classic "it works on my machine!" problem.
- **Isolation**: Containers isolate applications from each other and from the host system, preventing conflicts and improving security.
- **Portability**: A container can be easily moved and run on any compatible system.

Now, how does this differ from **Virtual Machines (VMs)**? A VM virtualizes an entire operating system, including its kernel, on top of your host system. It's like having a separate, complete computer running inside your computer. Containers, on the other hand, share the host system's kernel. This makes containers much lighter, faster to start, and more efficient in resource usage compared to VMs. They only package the application and its specific dependencies, not an entire OS.

The **OCI (Open Container Initiative)** container is essentially a standardized format for these isolated application packages. It ensures that containers created with one tool (like Podman) can be run by another OCI-compliant tool. It's the "standard box" for our analogy.

#### Podman and Daemonless Architecture

Now that we've covered containers, let's look at **Podman**. While Docker is perhaps the most well-known containerization tool, it traditionally relies on a long-running background service called a "daemon" (the `dockerd` process). This daemon manages all container operations.

**Podman's key differentiator is its "daemonless" architecture**. This means Podman does not require a central, continuously running background process to manage containers. When you execute a Podman command, it directly interacts with the Linux kernel and container runtimes (like runc or crun) to perform the requested action. Once the command finishes, Podman exits.

This daemonless design offers several significant advantages, especially for system administrators managing servers:

1. **Enhanced Security**: Without a single root daemon controlling all operations, the attack surface is reduced. Containers can be run as regular, unprivileged users, limiting potential damage if a container were to be compromised. This is a huge benefit for server security.
2. **Simplified Management**: There's no daemon to troubleshoot, restart, or monitor. Commands are executed directly, making the behavior more predictable and often easier to debug.
3. **Resource Efficiency**: No persistent daemon means fewer background processes consuming system resources when containers are not actively running.
4. **Integration with Systemd**: Podman integrates very well with `systemd` (the init system used by many Linux distributions). You can generate `systemd` unit files directly from Podman containers or pods, allowing for robust and standard service management.

In essence, Podman provides all the functionality you need to build, run, and manage containers, but in a more lightweight, secure, and flexible way, perfectly suited for server environments where stability and security are paramount. It feels very much like a native Linux command-line tool.

#### The Concept of Pods in Podman

While individual containers are powerful, sometimes you have applications that are composed of several tightly coupled services. For instance, a web application might consist of a front-end web server, a back-end API service, and a database. These components often need to share resources or communicate directly.

This is where the concept of a **"pod"** comes in. In Podman, a pod is a group of one or more containers that share resources, such as network namespaces, storage volumes, and process IDs. This means that all containers within a pod can communicate with each other via `localhost`, share storage, and are managed as a single unit.

The idea of pods originated from **Kubernetes**, an open-source system for automating deployment, scaling, and management of containerized applications. Podman's inclusion of pod functionality makes it an excellent tool for local development and testing of multi-container applications that will eventually be deployed to a Kubernetes cluster. It provides a consistent environment from your local machine (or server) to the production orchestrator.

For a system administrator, managing services within pods offers several advantages:

- **Simplified Management**: You can start, stop, or remove an entire application stack with a single command on the pod, rather than managing each individual container.
- **Resource Sharing**: Containers within a pod can efficiently share resources, which can lead to better performance and reduced overhead for tightly integrated services.
- **Logical Grouping**: It provides a clear logical grouping for related components of an application, making it easier to understand and operate complex deployments.

So, when you think of a pod in Podman, envision a self-contained environment for a set of co-located, co-managed containers that need to work closely together.

## Installing Podman

Since we're assuming an Debian-based Linux server environment with command-line tools only, the installation process for Podman is straightforward using the apt package manager.

First, it's always good practice to ensure your system's package list is up-to-date and that you have any necessary dependencies. You can achieve this with the following commands:

```Bash
sudo apt update
sudo apt -y upgrade
```

- `sudo apt update`: This command downloads the package lists from the repositories and updates them to get information on the newest versions of packages and their dependencies.
- `sudo apt -y upgrade`: This command installs the newest versions of all packages currently installed on the system. The `-y` flag automatically answers "yes" to any prompts, which is useful for automation or when you're confident in the upgrade.

Once your system is updated, you can install Podman using the `apt install` command:

```Bash
sudo apt install -y podman
```

This command will download and install the Podman package along with any required dependencies. The `-y` flag again automates the confirmation.

Keep in mind that while Podman is becoming more common in repositories, you might need to add a specific repository (like a PPA) to get the latest version.

After the installation completes, you can verify that Podman is correctly installed and accessible by running a simple command to display its version.

Execute the following command in your terminal:

```Bash
podman --version
```

Upon successful installation, this command should output the installed Podman version (e.g., `podman version 4.3.1`). If you see an error message like "command not found," it indicates an issue with the installation or your system's PATH configuration.

## Basic Podman Commands

### Image Management

#### Searching for Container Images (`podman search`)

Before you can run a container, you need a **container image**. Think of an image as a read-only template that contains the application, its libraries, and configuration. It's like a blueprint for your container.

Podman allows you to search for these images in various public and private **registries**. A registry is essentially a centralized repository for container images, much like a software repository for packages (e.g., Debians's `apt` repositories). The most well-known public registry is Docker Hub, but there are many others, including Quay.io and those run by cloud providers.

To search for images, you use the `podman search` command. It's quite intuitive. For example, if you wanted to find images related to the `nginx` web server, you would type:

```Bash
podman search nginx
```

This command will query configured registries and return a list of images whose names or descriptions match "nginx." The output typically includes columns like `INDEX`, `NAME`, `DESCRIPTION`, `STARS` (a popularity metric), `OFFICIAL` (indicating official images from software vendors), and `AUTOMATED` (indicating automated builds).

The `INDEX` column will often show where the image resides (e.g., `docker.io` for Docker Hub, `quay.io` for Quay.io). Pay attention to the `OFFICIA`L and `STARS` columns when choosing images, as these can be indicators of trustworthiness and quality.

#### Pulling a Container Image (`podman pull`)

Once you've identified an image you want to use from a registry, you need to download it to your local machine. This process is called "pulling" an image. It's akin to downloading a software package before installing it.

You use the `podman pull` command for this purpose. The most common format is `podman pull <registry>/<image_name>:<tag>`.

- `<registry>`: This specifies the registry where the image is located (e.g., `docker.io`, `quay.io`). If you omit this, Podman defaults to docker.io.
- `<image_name>`: This is the name of the image (e.g., `nginx`, `ubuntu`).
- `<tag>`: This refers to a specific version or variant of the image (e.g., `latest`, `1.25`, `focal`). If you omit the tag, latest is usually assumed, but it's good practice to specify a tag for reproducibility.

Example: To pull the official Nginx web server image with the `stable` tag from Docker Hub, you would execute:

```Bash
podman pull docker.io/library/nginx:stable
```

Or, more commonly, if `docker.io/library/` is implied (which it often is for official images on Docker Hub):

```Bash
podman pull nginx:stable
```

When you run this command, Podman will download all the necessary layers that make up the Nginx image. You'll see progress indicators in your terminal as the download proceeds. Once complete, the image will be stored locally on your system, ready to be used to create containers.

It's important to pull the exact image you intend to use to ensure consistency. Specifying tags helps avoid unexpected behavior if the "latest" version changes over time.

#### Listing Available Images (`podman images`)

After pulling images or building your own, you'll want to see what's currently stored locally. The `podman images` command serves this purpose. It provides a concise list of all container images that Podman has downloaded or created on your system.

To see your local images, simply type:

```Bash
podman images
```

The output will typically show several columns:

- **REPOSITORY**: The name of the image (e.g., `nginx`, `ubuntu`).
- **TAG**: The specific version or variant of the image (e.g., `stable`, `latest`, `22.04`).
- **IMAGE ID**: A unique identifier for the image.
- **CREATED**: When the image was built or downloaded.
- **SIZE**: The size of the image on disk.

This command is incredibly useful for keeping track of your image inventory, checking disk space usage, and ensuring you have the correct image versions available for your operations.

#### Removing Images (`podman rmi`)

Just as you pull images, you'll also need to remove them from your local system when they are no longer required. This helps free up disk space and keeps your image inventory tidy. The command for this is `podman rmi` (which stands for "remove image").

To remove an image, you typically specify its name and tag, or its Image ID. It's generally safer and clearer to use the name and tag.

**Example**: If you previously pulled the `nginx:stable` image and no longer need it, you would execute:

```Bash
podman rmi nginx:stable
```

Alternatively, if you want to remove an image by its unique **Image ID** (which you can get from podman images), you would use:

```Bash
podman rmi <Image ID>
```

For instance, if `podman images` showed an Image ID of `a1b2c3d4e5f6` for the Nginx image, you could run:

```Bash
podman rmi a1b2c3d4e5f6
```

**Important Note**: Podman will prevent you from removing an image if there are any containers (even stopped ones) that were created from it. You must remove those containers first before you can remove the image. This is a safety mechanism to prevent accidental data loss or disruption to existing containers.

### Container Lifecycle Management

#### Running a Basic Container (`podman run`)

The `podman run` command is arguably the most fundamental command you'll use. It creates a new container from a specified image and then executes a command within that container. Think of it as launching an instance of the blueprint you've pulled.

The basic syntax is `podman run <image_name>:<tag> <command>`.

Let's look at some common and essential options you'll use with `podman run`:

- `-it` **(Interactive Terminal)**: This is a combination of two flags:
  - `-i` **(or** `--interactive`**)**: Keeps `STDIN` open even if not attached, allowing you to provide input to the container.
  - `-t` **(or** `--tty`**)**: Allocates a pseudo-TTY, essentially giving you an interactive shell inside the container.
  - When you want to interact directly with the container (e.g., run shell commands), you'll almost always use `-it`.

- `--name <container_name>`: This option allows you to assign a human-readable name to your container. If you don't specify a name, Podman will generate a random, sometimes whimsical, name for it. Naming containers makes them much easier to identify and manage later.

##### Example: Running an interactive Ubuntu container

Let's say you want to quickly spin up an Ubuntu container to run some commands. You would use:
Bash

podman run -it --name my-ubuntu-test ubuntu:latest /bin/bash

Let's break this down:

- `podman run`: The command to create and run a container.
- `-it`: We want an interactive terminal inside the container.
- `--name my-ubuntu-test`: We're naming this specific container instance `my-ubuntu-test`.
- `ubuntu:latest`: This specifies the image we want to use. We assume you've already pulled `ubuntu:latest` or Podman will attempt to pull it automatically.
- `/bin/bash`: This is the command that will be executed inside the container. In this case, it starts a Bash shell.

When you run this command, you'll find yourself inside the container's shell prompt. From there, you can execute commands as if you were on a fresh Ubuntu system, completely isolated from your host. To exit the container, you can type `exit` at the shell prompt.

#### List Running Containers (`podman ps` and `podman ps -a`)

After you run containers, you'll need a way to monitor their status. The `podman ps` command (short for "process status," similar to the Linux `ps` command) is your primary tool for this.

- **Listing Running Containers**:

    To see only the containers that are currently active and running, simply use:

    ```Bash
     podman ps
    ```

    The output will typically display columns such as:

  - `CONTAINER ID`: A unique identifier for the container.
  - `IMAGE`: The image the container was created from.
  - `COMMAND`: The command executed inside the container.
  - `CREATED`: When the container was created.
  - `STATUS`: The current state of the container (e.g., `Up X seconds`, `Exited (0) X seconds ago`).
  - `PORTS`: Any port mappings (we'll cover this soon).
  - `NAMES`: The name you assigned with `--name` or a random one Podman generated.

- **Listing All Containers (Running and Stopped)**:

    Often, you'll want to see containers that have exited or stopped, not just the currently running ones. To include all containers, regardless of their status, you add the `-a` (or `--all`) flag:

    ```Bash
    podman ps -a
    ```

    This command is incredibly useful for reviewing past container activity, identifying containers that might need to be restarted, or finding containers that need to be removed to free up resources.

#### Stopping and Starting Containers (`podman stop` and `podman start`)

Containers, much like traditional services, often need to be stopped and started. This is essential for maintenance, troubleshooting, or simply temporarily pausing a service.

- **Stopping a Container (**`podman stop`**)**

    To gracefully shut down a running container, you use the `podman stop` command. It signals the main process inside the container to terminate.

    You'll need the **Container ID** or the **Name** of the container, which you can get from `podman ps` or `podman ps -a`.

    **Example**: If you have a container named `my-ubuntu-test` (from our previous example), you would stop it with:

    ```Bash
    podman stop my-ubuntu-test
    ```

    Podman will send a `SIGTERM` signal to the container's main process, allowing it a grace period to shut down cleanly. If it doesn't stop within that period, Podman will send a `SIGKILL` to force termination.

- **Starting a Container (**`podman start`**)**

    Once a container has been stopped (or if it exited on its own), you can restart it using the `podman start` command. This will resume the container from its last state, executing its primary command again.

    Again, you'll use the **Container ID** or the **Name**.

    **Example**: To restart our my-ubuntu-test container:

    ```Bash
    podman start my-ubuntu-test
    ```

    After starting, you can use `podman ps` to confirm that the container is now running. This allows you to pause and resume services without having to destroy and recreate the container, preserving its internal state and any changes made within it.

#### Removing Containers (`podman rm`)

While `podman stop` pauses a container, `podman rm` permanently deletes it. Removing a container means deleting its writable layer, associated storage, and its configuration. This action cannot be undone, so it's important to use it with care.

You typically remove a container using its **Container ID** or its **Name**.

**Example**: If you have a stopped container named `my-ubuntu-test` (from our earlier example), you would remove it with:

```Bash
podman rm my-ubuntu-test
```

Alternatively, using its Container ID (e.g., `a1b2c3d4e5f6`):

```Bash
podman rm a1b2c3d4e5f6
```

**Important Considerations**:

- **Stopped Containers Only**: You can only remove containers that are in a `Exited` or `Stopped` state. Podman will prevent you from removing a running container to avoid accidental disruption. If you need to remove a running container, you must first stop it using `podman stop`.
- **Force Removal (**`-f` **or** `--force`**)**: While not recommended for general use, if you absolutely need to remove a running container immediately without stopping it first (e.g., if it's unresponsive), you can use the `-f` flag. However, this can lead to data loss and is typically used only in emergency situations.

    ```Bash
    podman rm -f my-running-container
    ```

- **Removing All Stopped Containers (**`podman container prune`**)**: For cleaning up multiple stopped containers, Podman offers a convenient command:

    ```Bash
    podman container prune
    ```

    This command will prompt you before removing all stopped containers, which is very useful for system maintenance.

Using `podman rm` is crucial for keeping your server environment clean and managing disk space effectively.

#### Detached Mode (`-d`) and Port Mapping (`-p`)

When you run a container in an interactive terminal (using `-it`), it takes over your current shell. This is fine for quick tests, but for server applications like web servers or databases, you need them to run continuously in the background without tying up your terminal. This is where detached mode comes in.

- **Detached Mode (**`-d` **or** `--detach`**)**

    When you add the `-d` flag to `podman run`, the container starts in the background, and Podman prints the container ID and immediately returns control of your terminal to you. The container continues to run as a background process.

    **Example**: To run an Nginx web server container in the background:

    ```Bash
    podman run -d --name my-nginx nginx:latest
    ```

    After executing this, you can use `podman ps` to confirm that `my-nginx` is running in the background.

- **Port Mapping (**`-p` **or** `--publish`**)**

    By default, containers are isolated from the host machine's network. If an application inside your container is listening on a specific port (e.g., a web server on port 80), you won't be able to access it from your host machine or from the internet without explicitly mapping a port from the container to a port on your host. This is called port mapping.

    The `-p` flag allows you to define this mapping. The syntax is typically `host_port:container_port`.

    **Example**: To run an Nginx web server (which listens on port 80 inside the container) and make it accessible on port 8080 of your host machine, you would use:

    ```Bash
    podman run -d --name my-nginx-web -p 8080:80 nginx:latest
    ```

    In this example:

  - `-d`: Runs the container in detached mode.
  - `--name my-nginx-web`: Assigns a readable name.
  - `-p 8080:80`: This is the port mapping. It means traffic coming to `host_ip:8080` will be forwarded to `container_ip:80`.
  - `nginx:latest`: The image to use.

Now, if Nginx is running correctly inside the container, you could access it from your host's web browser or via `curl` at `http://localhost:8080` (assuming `localhost` for testing on the server itself, or the server's IP address if accessing remotely).

## Working with Podman

### Pods and Grouping Containers

We briefly touched upon pods earlier, but let's dive deeper into their purpose. In the world of container orchestration (like Kubernetes, which Podman aligns with), a **pod** is the smallest deployable unit. For Podman, it represents a logical grouping of one or more containers that are deployed and managed together.

Think of a pod as a small, isolated "virtual machine" that hosts a tightly coupled set of containers. These containers within a pod share crucial resources:

1. **Network Namespace**: All containers within a pod share the same IP address and network ports. This means they can communicate with each other using `localhost`, just as if they were different processes running on the same physical machine. This simplifies inter-container communication immensely.
2. **IPC Namespace**: They share Inter-Process Communication (IPC) mechanisms, allowing them to communicate using shared memory or message queues.
3. **Storage Volumes (optional but common)**: Containers in a pod can easily share persistent storage volumes, enabling them to read from and write to the same data locations. This is vital for applications that need to persist data or share configuration files.

**Why is this useful for a system administrator?**

- **Atomic Deployments**: Instead of managing individual containers that make up an application, you manage the entire application as a single unit (the pod). If one component fails, you can restart or replace the entire pod to ensure the application's integrity.
- **Co-location and Co-scheduling**: Pods ensure that related containers are always run together on the same host, minimizing network latency between them.
- **Simplified Resource Sharing**: Sharing resources like network and storage within a pod is much simpler than trying to link disparate individual containers.
- **Kubernetes Compatibility**: As mentioned, Podman's pod concept is directly inspired by Kubernetes. This means that if you design your multi-container applications using Podman pods, they will be much easier to migrate and manage in a Kubernetes cluster later on, providing a consistent development-to-production workflow.

Imagine a web application where the main web server needs to read configuration from a sidecar container that fetches dynamic content. Grouping them in a pod ensures they can effortlessly communicate and share files.

### Creating a Pod (`podman pod create`)

Creating a pod in Podman is a straightforward process. You use the `podman pod create` command. This command sets up the necessary infrastructure for the pod, such as its unique network namespace, but it doesn't run any containers within it yet. Think of it as preparing the shared "house" before moving in the "roommates" (containers).

The basic syntax is simple:

```Bash
podman pod create [options]
```

You'll almost always want to give your pod a meaningful name using the `--name` option, just like with individual containers.

**Example**: To create a new pod named `my-web-app-pod`, you would execute:

```Bash
podman pod create --name my-web-app-pod
```

Upon successful creation, Podman will output the unique **Pod ID** of the newly created pod. This ID is similar to a Container ID and can be used to reference the pod in future commands.

It's important to remember that `podman pod create` only creates the pod infrastructure. At this stage, no application containers are running inside it.

### Adding Containers to a Pod (`--pod` options with `podman run`)

You don't "add" containers to a pod in a separate step after they are created. Instead, you specify which pod a container should belong to when you run the container using the `podman run` command. This is achieved with the `--pod` option.

When you use `--pod <pod_name_or_id>` with `podman run`, Podman will create and start the new container within the network and IPC namespaces of the specified pod.

#### Example: Running a Web Server and a Database in a Pod

Let's assume you've already created a pod named `my-web-app-pod`. Now, let's add an Nginx web server and a simple `redis` database to it.

First, the Nginx container:

```Bash
podman run -d --pod my-web-app-pod --name nginx-in-pod nginx:latest
```

Let's break this down:

- `podman run`: Standard command to run a container.
- `-d`: Runs the container in detached mode (background).
- `--pod my-web-app-pod`: This is the key! It tells Podman to attach this new container to the `my-web-app-pod` pod.
- `--name nginx-in-pod`: A specific name for this Nginx container instance.
- `nginx:latest`: The image for the Nginx web server.

Now, let's add a Redis database container to the same pod:

```Bash
podman run -d --pod my-web-app-pod --name redis-in-pod redis:latest
```

Notice the `--pod my-web-app-pod` option is identical. This ensures both `nginx-in-pod` and `redis-in-pod` are part of the same pod.

**Key advantage**: Since both containers are in the same pod, `nginx-in-pod` can access `redis-in-pod` simply by connecting to `localhost:<redis_port>` (default Redis port is 6379) because they share the same network namespace. You don't need to map complex networks or link them explicitly.

### Listing Pods and Their Associated Containers

Podman provides specific commands to help you inspect your pods and the containers running within them.

1. **Listing all Pods (**`podman pod ps`**)**

    To get an overview of all the pods on your system, whether they are running or not, use:

    ```Bash
    podman pod ps
    ```

    This command will show you basic information about each pod, including its ID, Name, Status, and the number of containers associated with it.

2. **Inspecting a Specific Pod (**`podman pod inspect`**)**

    For a more detailed view of a particular pod, including its full configuration and a list of all containers inside it, use `podman pod inspect` followed by the pod's name or ID. This command provides a JSON output with extensive information.

    ```Bash
    podman pod inspect my-web-app-pod
    ```

    You would look for the `"Containers"` section within the JSON output to see details about each container within that pod.

3. **Listing Containers and Their Pods (**`podman ps --pod`**)**

    The podman ps command (which we used earlier for individual containers) also has an excellent option to show container-to-pod relationships. By adding the --pod flag, you can see which pod each container belongs to:

    ```Bash
    podman ps -a --pod
    ```

    This output will add a `POD` column, clearly indicating the name of the pod each container is associated with. This is incredibly helpful for visualizing your multi-container deployments.

Using these commands, you can effectively monitor the health and structure of your pod-based applications.

### Stopping and Removing Pods

Managing pods as a single unit is one of their core benefits. Podman provides commands to stop and remove entire pods, which in turn affects all containers running within them.

- **Stopping a Pod (**`podman pod stop`**)**

    To gracefully shut down all running containers within a specific pod, you use the podman pod stop command.

    ```Bash
    podman pod stop <pod_name_or_id>
    ```

    **Example**: To stop our my-web-app-pod:

    ```Bash
    podman pod stop my-web-app-pod
    ```

    When this command is executed, Podman will send termination signals to all containers inside `my-web-app-pod`, allowing them to shut down cleanly. After the command completes, `podman pod ps` will show the pod as `Exited`, and `p`odman ps -a --pod` will show the individual containers as `Exited`.

- **Removing a Pod (**`podman pod rm`**)**

    To permanently delete a pod and all its associated containers (even if they are stopped), you use the `podman pod rm` command. Be cautious with this command, as it is destructive.

    ```Bash
    podman pod rm <pod_name_or_id>
    ```

    **Example**: To remove `my-web-app-pod` and all its containers:

    ```Bash
    podman pod rm my-web-app-pod
    ```

    Podman will first attempt to stop any running containers within the pod before removing them and the pod infrastructure itself. If you want to force the removal of a running pod without first stopping it, you can use the `-f` or `--force` flag, but this is generally not recommended as it bypasses graceful shutdowns.

    ```Bash
    podman pod rm -f my-running-pod
    ```

- **Removing All Stopped Pods (**`podman pod prune`**)**

    Similar to `podman container prune`, there's a convenient command to clean up all exited (stopped) pods:

    ```Bash
    podman pod prune
    ```

    This command will prompt you for confirmation before proceeding, which is useful for cleaning up your environment.

These commands enable you to manage multi-container applications as cohesive units, simplifying deployment and teardown processes.

## Practical Use Cases

### Common Scenarios for Podman

Podman offers several compelling advantages and practical use cases, especially given its daemonless nature and strong integration with Linux systems:

1. **Running Services Securely and Efficiently**:
   - You can containerize various services (e.g., web servers like Nginx or Apache, databases like PostgreSQL or MySQL, caching layers like Redis) and run them on your Ubuntu server using Podman. This provides isolation, ensures consistent environments, and simplifies dependency management.
   - Since Podman allows running containers as unprivileged users (without root), it significantly enhances the security posture of your deployed services compared to traditional daemon-based solutions.

2. **Isolated Development and Testing Environments**:
   - Need to test a new version of an application or a specific software configuration without impacting your host system? Podman lets you quickly spin up isolated environments. You can run different versions of Python, Node.js, or specific library versions without worrying about conflicts with your host's packages.
   - This is invaluable for testing patches, new software, or reproducing issues reported by developers in a clean, disposable environment.

3. **Building and Distributing Custom Applications**:
   - Podman can be used to build your own custom container images (`podman build`) for applications developed in-house. These images can then be easily deployed across multiple servers, ensuring that every deployment is identical.
   - This streamlines the deployment pipeline and reduces configuration drift between environments.

4. **Managing System Services with Systemd**:
   - A significant advantage of Podman for server administrators is its seamless integration with `systemd`. You can generate `systemd` unit files directly from running Podman containers or pods (`podman generate systemd`).
   - This allows you to manage your containerized applications just like any other system service, enabling automatic startup on boot, dependency management, logging, and other standard `systemd` features. This is a very powerful capability for production environments.

5. **Offline or Air-Gapped Environments**:
   - Because Podman doesn't require a persistent daemon, it can be particularly useful in environments with strict security policies or limited network connectivity (e.g., air-gapped networks). Once images are pulled, containers can be run without an active internet connection or a running daemon.

In essence, Podman empowers you to manage applications and services on your servers with greater agility, security, and integration with native Linux tools.