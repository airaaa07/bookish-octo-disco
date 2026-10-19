---
title: 1.1.1 - Containers Overview
excerpt: Page 1
deprecated: false
hidden: false
link:
  new_tab: false
metadata:
  robots: index
---
A container is a process that runs an application independently from other containers on the host.<br />**What is a process?**<br />A process is a program that is currently running on a computer.

### 1. Program vs. Process

Imagine you have a calculator application on your computer.

- **Program:** The calculator application stored on your computer, waiting to be opened.
- **Process:** When you open the calculator, the operating system starts running it. That running instance is called a process.

In simple words, a program is something you can run; a process is that program while it is running.

### 2. What does this mean in a container?

Suppose you run a web application inside a container.

**Host Machine**

The physical or virtual machine running containers.

```
Host Machine
│
├── Container 1
│   └── Web Application Process
│
└── Container 2
    └── Database Process
```

Both containers run on the same host, but their processes are isolated from each other.

For example:

- Container 1 runs a web application.
- Container 2 runs a database.
- Each container runs its own application process, or processes, in an isolated environment.

If the web application crashes, the database process can continue running independently, provided it is not affected by the same underlying failure.

### 3. What does "independently" mean?

It means each container has an isolated environment, so its processes do not normally interfere directly with processes in other containers.

Containers use Linux kernel features, such as **namespaces** and **control groups (cgroups)**, to provide process isolation and resource management.

- **Namespaces:** Isolate a container's processes and resources from those of other containers and the host system. They provide separate views of resources such as process IDs, network interfaces, mount points, and hostnames, allowing containers to operate independently.
- **Control groups (cgroups):** Manage and limit the resources that container processes can consume, such as CPU time and system memory. They help prevent one container from consuming excessive resources and affecting other containers running on the same host.

The **Open Container Initiative (OCI)** maintains open standards for container images, container runtimes, and container-related technologies. These standards define common formats and behaviors that promote compatibility across different container tools and platforms.

### 4. Remember this for OpenShift

- **Program:** Application code or an executable that can be run.
- **Process:** A running instance of a program.
- **Container:** An isolated environment in which application processes run.
- **Pod:** The Kubernetes/OpenShift unit that runs one or more containers.

**One important detail:** A container is not technically a separate computer. Its processes use the host machine's operating-system kernel.

### <br />References:

[https://github.com/RHEPDS/bootcamp/blob/main/1_OpenShift%20for%20Developers.pdf](https://github.com/RHEPDS/bootcamp/blob/main/1_OpenShift%20for%20Developers.pdf "https://github.com/RHEPDS/bootcamp/blob/main/1_OpenShift%20for%20Developers.pdf")

Check Page: 16 to 32
