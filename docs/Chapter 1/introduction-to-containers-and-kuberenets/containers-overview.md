---
title: Containers Overview
excerpt: SubPage 1
deprecated: false
hidden: false
metadata:
  robots: index
---
A container is a process that runs an application independently from other containers on the host.

A process is a program that is currently running on a computer.

Let's understand it step by step.

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

Containers use operating-system features, such as **namespaces** and **control groups (cgroups)**, to provide isolation and manage resources.

### 4. Remember this for OpenShift

- **Program:** Application code or an executable that can be run.
- **Process:** A running instance of a program.
- **Container:** An isolated environment in which application processes run.
- **Pod:** The Kubernetes/OpenShift unit that runs one or more containers.

**One important detail:** A container is not technically a separate computer. Its processes use the host machine's operating-system kernel.
