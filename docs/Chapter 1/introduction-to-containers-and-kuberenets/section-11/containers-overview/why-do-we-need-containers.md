---
title: Why Do We Need Containers?
excerpt: >-
  Compare host processes with containers to understand isolation, dependency
  management, and deployment.
deprecated: false
hidden: false
metadata:
  robots: index
---
Compare applications running directly on a host with applications packaged in containers to understand when containers help.

Processes run applications; containers package and isolate their running environments.

## 1. Without Containers: Multiple Processes on One Host

Imagine your host machine runs three applications:

- Application A — a web application
- Application B — a database
- Application C — a Python application

You can run all three as normal processes. No problem.

However, suppose each application needs a different version of a dependency.

**One host machine — One operating system**

| Application | Example requirement |
| ----------- | ------------------- |
| App A       | Python 3.9          |
| App B       | PostgreSQL 16       |
| App C       | Python 3.12         |

Applications installed directly on a host can depend on different versions of the same software, as Apps A and C do here. Managing these dependencies and their configurations in a shared host environment can become difficult.

The problem isn't that processes cannot run together. They can! The problem is managing their dependencies, configuration, permissions, and deployment without conflicts.


<Image src="https://files.readme.io/2fd3ba5dd40d29f56930f2d7e45ec9a6a2d78177fdde109e6b26e67b68fced78-applcations.png" align="center" caption="Applications running on one host" border={true} framed={true} />


## 2. With Containers: Each Application Gets Its Own Environment

Instead of installing everything directly on the host, you package each application with the dependencies and configuration it needs.

**One host machine — Shared host operating-system kernel**

| Container   | Contents                      |
| ----------- | ----------------------------- |
| Container A | App + Python 3.9 + libraries  |
| Container B | Database + required files     |
| Container C | App + Python 3.12 + libraries |

Each container has its own isolated application environment, but all share the host kernel.

Now, each application can use its own dependency versions without needing to install all of them into one shared host environment.


<Image src="https://files.readme.io/681774a7d5ee48866ef86763bfa907262a012d3a7b554e1068714db0d4a1d7c6-containers.png" align="center" caption="Applications packaged in separate containers" border={true} framed={true} />


## 3. Why Are Containers Useful?

Here are the main reasons:

1. **Isolation:** Applications are separated from one another. For example, one application's filesystem and processes are isolated from another's, subject to the configured permissions and security controls.
2. **Dependency management:** One application can use Python 3.9 while another uses Python 3.12, without requiring both versions to be managed as one shared application environment on the host.
3. **Portability:** Package an application into a container image and run it in different compatible environments, such as a developer's machine, a test server, or a production cluster.
4. **Easy deployment and replacement:** Instead of manually installing and configuring an application on every server, deploy its image. You can replace the container with a new one when you release an updated version.
5. **Efficient resource usage:** Containers generally use fewer resources than separate virtual machines because they share the host's operating-system kernel.

## 4. Then Why Do We Need OpenShift?

Imagine a university ERP application with three components:

- **Frontend** — the user interface
- **Backend** — business logic and APIs
- **Database** — stores student and fee information

You could run all three as processes on one server. But as the application grows, you might need to deploy new versions, restart failed components, scale the backend, and manage many servers.

OpenShift helps manage the containers running these components across a cluster of machines.

**OpenShift cluster**

| Component | Runs in   |
| --------- | --------- |
| Frontend  | Container |
| Backend   | Container |
| Database  | Container |

OpenShift orchestrates deployment, scaling, networking, and recovery of containerized applications.


<Image src="https://files.readme.io/13f0a00d60af7105145050f694d09c7277178a5261917501f73d224e178a1a41-hostvscontainer.png" align="center" caption="Host processes compared with containers" border={true} framed={true} />


## 5. The Key Takeaway

| Concept         | Main purpose                                                         |
| --------------- | -------------------------------------------------------------------- |
| Process         | Runs a program                                                       |
| Container       | Runs application processes in an isolated, packaged environment      |
| Virtual machine | Runs a separate operating-system instance with its own kernel        |
| OpenShift       | Manages and orchestrates containerized applications across a cluster |

**The important distinction:** We don't use containers because a host cannot run multiple processes. We use them to package, isolate, deploy, and manage applications more consistently. Containers share the host kernel, and their isolation depends on the configured permissions and security controls.