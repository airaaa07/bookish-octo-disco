---
title: 1.1.2 - Kubernetes Overview
deprecated: false
hidden: false
link:
  new_tab: false
metadata:
  robots: index
---
# Kubernetes Overview

## 1. What Is Kubernetes?

**Kubernetes (K8s)** is an open-source platform used to automate the deployment, scaling, and management of containerized applications.

Applications can be packaged into containers along with the libraries and dependencies they need to run. Kubernetes manages these containers across a cluster of machines, helping applications run reliably and efficiently.

### Example

Consider a university portal that allows students to view their profiles, check examination results, and access course information.

The application might contain:

- **Frontend:** Provides the user interface.
- **Backend:** Processes requests and implements application logic.
- **Database:** Stores student information and examination results.

Kubernetes helps run the application components, maintain the desired number of application instances, distribute workloads across available machines, and recover from certain failures.

In simple terms, **Kubernetes manages containerized applications so that administrators do not have to handle every operational task manually.**

## 2. Running Containers vs. Using Kubernetes

Containers can run directly on a Linux system using tools such as Podman or Docker. However, running containers on a single machine is different from managing an application across multiple machines.

### Without Kubernetes

An administrator may need to:

- Start and stop containers manually.
- Monitor application health.
- Restart failed containers.
- Configure multiple servers.
- Manage application scaling and updates.

As the application grows, these tasks become increasingly difficult.

### With Kubernetes

Kubernetes automates many of these operational tasks, including:

- Scheduling pods onto suitable nodes.
- Replacing failed pods managed by workload controllers.
- Scaling the number of application replicas.
- Performing controlled application updates.
- Providing service discovery and load distribution through Kubernetes Services.

### Practical example

Suppose a university portal runs three application replicas.

If one pod fails, Kubernetes can create a replacement to restore the desired replica count. If traffic increases, an administrator can increase the number of replicas to handle the additional workload.

However, Kubernetes cannot guarantee uninterrupted service by itself. Availability also depends on the application's design, cluster capacity, networking, storage, and configuration.

## 3. Kubernetes Cluster and Nodes

A **Kubernetes cluster** is a group of machines that work together to run and manage containerized applications.

Each machine in the cluster is called a **node**. A node can be a physical server or a virtual machine.

Kubernetes nodes are commonly divided into two categories.

### 3.1 Control Plane Node

The control plane manages the overall state of the cluster and coordinates its operations.

Its main components include:

- **API Server:** Provides the interface through which users and other components interact with Kubernetes.
- **etcd:** Stores the cluster's configuration and state data.
- **Scheduler:** Selects suitable nodes for newly scheduled pods.
- **Controller Manager:** Runs controllers that work to maintain the desired cluster state.

### 3.2 Worker Node

Worker nodes run application workloads.

Their main components include:

- **Kubelet:** Ensures that pods assigned to the node are running as specified.
- **Container Runtime:** Runs containers, such as containerd or CRI-O.
- **Kube-proxy or an alternative networking implementation:** Helps implement Service networking, depending on the cluster configuration.

Worker nodes typically run application pods.

### Example cluster architecture

A cluster might contain one control plane node and three worker nodes.

- The control plane manages the cluster.
- Worker Node 1 runs two application pods.
- Worker Node 2 runs another application pod.
- Worker Node 3 runs additional workloads or provides spare capacity.

In production environments, multiple control plane nodes are commonly used to improve control plane availability.

### What happens if a worker node fails?

Suppose Worker Node 1 becomes unavailable.

1. Kubernetes detects that the node is not responding.
2. The control plane updates the node's status and handles the failure.
3. A workload controller, such as a Deployment, works to restore the desired number of replicas.
4. Replacement pods may be scheduled onto healthy nodes if sufficient resources are available.

Recovery is not necessarily immediate. Pod termination, scheduling, storage attachment, and application startup can all affect recovery time.

**Important:** Kubernetes can recreate application pods, but it does not automatically guarantee that application data stored on failed or unavailable storage will be recovered.

## 4. Declarative Configuration and Desired State

Kubernetes uses a **declarative configuration model**.

Instead of manually describing every step needed to run an application, an administrator specifies the desired state. Kubernetes then continuously works to make the actual state match that configuration.

### Example

Suppose a university portal requires:

- Application image: `university-portal:v1`
- Number of replicas: `3`
- Memory request: `512Mi` per container

The administrator defines these settings in a Kubernetes manifest.

Kubernetes attempts to maintain three application replicas according to the configuration.

If a managed pod fails, Kubernetes works to replace it. If the administrator changes the replica count from three to five, Kubernetes attempts to create two additional replicas.

This ongoing process is called **reconciliation**.

### Desired state vs. actual state

- **Desired state:** What the administrator declares in the configuration.
- **Actual state:** What currently exists and runs in the cluster.
- **Reconciliation:** The process of bringing the actual state closer to the desired state.

For example:

| Desired state | Actual state | Kubernetes response                     |
| ------------- | ------------ | --------------------------------------- |
| 3 replicas    | 3 replicas   | No replica-count correction needed      |
| 3 replicas    | 2 replicas   | Attempts to restore the missing replica |
| 5 replicas    | 3 replicas   | Attempts to create additional replicas  |

Kubernetes continuously reconciles resources, although some failures or configuration problems may prevent it from reaching the desired state.

## 5. Workloads and Resource Requirements

A **workload** is an application or task that Kubernetes runs and manages.

Kubernetes provides several workload resources for different purposes.

| Workload    | Purpose                                                                  | Example                    |
| ----------- | ------------------------------------------------------------------------ | -------------------------- |
| Deployment  | Manages replicated, typically stateless applications and rolling updates | University portal frontend |
| StatefulSet | Manages applications requiring stable identities or persistent storage   | Distributed database       |
| DaemonSet   | Runs a pod on every eligible node or selected nodes                      | Node monitoring agent      |
| Job         | Runs a task until it completes                                           | Database migration         |
| CronJob     | Creates jobs on a schedule                                               | Nightly report generation  |

### CPU and memory requests

Kubernetes allows administrators to define CPU and memory requirements for containers.

- **Requests:** The resources Kubernetes uses when evaluating whether a node can accommodate a pod.
- **Limits:** The maximum resource consumption configured for a container, subject to the resource type and enforcement mechanism.

### Practical example

Suppose the university portal has three replicas, and each replica requests `512Mi` of memory.

The total memory request is:

`3 × 512Mi = 1536Mi`

Therefore, the three replicas request a combined total of `1536Mi` of memory.

This is a scheduling requirement, not necessarily the amount of memory the application will consume at every moment.

Kubernetes considers available node resources and other scheduling constraints when placing pods. If no suitable node has sufficient available capacity, a pod may remain in the `Pending` state.

## 6. What Is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes.

A pod contains one or more containers that are scheduled together and share certain resources.

Containers within the same pod share:

- **Network namespace:** They use the same pod IP address and can communicate through `localhost`.
- **Port space:** Containers share the pod's network environment, so they must avoid conflicting port usage.
- **Configured shared storage:** Containers can access the same mounted volume when it is configured for them.

All containers in a pod run on the same node and share the pod's lifecycle.

### 6.1 Single-container pod

Most pods contain one main application container.

For example, a university frontend pod might contain a single container running a web application.

A Deployment can manage multiple replicas of this pod to support scaling and recovery.

### 6.2 Multi-container pod

Some containers need to work closely together and share resources.

For example, a pod might contain:

- **Application container:** Runs the main application.
- **Helper container:** Collects application logs from a shared volume and forwards them to a logging system.

Both containers run in the same pod and can communicate through the shared network environment.

Multi-container pods are useful when containers have a tightly coupled relationship. Independent application components are generally better placed in separate pods.

### Pod vs. container

A container is a running instance of an application image. A pod is the Kubernetes object that provides the environment in which one or more containers run.

**Remember:** Kubernetes schedules pods, not individual containers independently.

## 7. How Kubernetes Runs an Application

Let's follow the lifecycle of the university portal.

Suppose the administrator wants three replicas of the application running in the cluster.

1. **Define the application:** The administrator creates a Deployment manifest specifying the application image and the desired replica count.
2. **Submit the configuration:** The manifest is submitted to the Kubernetes API Server.
3. **Store the desired state:** Kubernetes records the resource state in the cluster's backing store.
4. **Reconcile the Deployment:** Kubernetes controllers work to create the required ReplicaSet and pods.
5. **Schedule the pods:** The scheduler selects suitable worker nodes for pods that need placement.
6. **Start the containers:** The kubelet on each selected node works with the container runtime to start the containers.
7. **Check application health:** Configured probes can help Kubernetes determine whether containers have started, are ready to receive traffic, or need restarting.
8. **Route application traffic:** A Kubernetes Service provides a stable network endpoint for reaching eligible application pods.
9. **Maintain the desired state:** Controllers continue reconciling resources when pods fail or the desired replica count changes.

If the administrator scales the Deployment from three replicas to five, Kubernetes attempts to create two additional pods.

If a pod fails, Kubernetes can replace it when the workload controller's desired state requires another replica.

## 8. Scaling and High Availability

Kubernetes supports scaling applications and distributing workloads across a cluster.

### Horizontal scaling

Horizontal scaling means increasing or decreasing the number of application replicas.

For example:

- Normal traffic: 3 replicas.
- Increased traffic: 5 replicas.
- Reduced traffic: 2 replicas.

Replica counts can be changed manually or managed automatically using a Horizontal Pod Autoscaler when the required metrics and configuration are available.

### High availability

High availability means designing a system to continue providing service despite certain failures.

For example, running application replicas on multiple worker nodes can reduce the impact of losing a single node.

However, multiple replicas alone do not guarantee high availability. Appropriate scheduling policies, sufficient capacity, redundant control plane components where required, resilient storage, and correctly configured networking are also important.

## 9. Kubernetes Configuration Files

Kubernetes resources are commonly defined using YAML manifest files.

For example, the following manifest defines a Deployment with three replicas of a university portal application.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: university-portal
spec:
  replicas: 3
  selector:
    matchLabels:
      app: university-portal
  template:
    metadata:
      labels:
        app: university-portal
    spec:
      containers:
        - name: portal
          image: nginx:1.28
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

**Note:** The example uses the NGINX image to demonstrate the Deployment structure. Replace it with your actual university portal image when deploying the application.

Save the manifest as `deployment.yaml`.

Apply it to the cluster:

```
kubectl apply -f deployment.yaml
```

Check the Deployment:

```
kubectl get deployments
```

Check the pods:

```
kubectl get pods
```

Inspect a pod in detail:

```
kubectl describe pod <pod-name>
```

View application logs:

```
kubectl logs <pod-name>
```

These commands help administrators create resources, verify the running state, troubleshoot problems, and inspect application behavior.

## 10. Key Kubernetes Concepts at a Glance

| Concept        | Meaning                                                        |
| -------------- | -------------------------------------------------------------- |
| Cluster        | The complete Kubernetes environment                            |
| Node           | A machine that participates in the cluster                     |
| Control plane  | Coordinates and manages cluster operations                     |
| Worker node    | Runs application workloads                                     |
| Container      | Packages and runs an application process                       |
| Pod            | The smallest deployable Kubernetes unit                        |
| Deployment     | Manages replicated application pods and updates                |
| Service        | Provides a stable way to access a set of pods                  |
| Manifest       | A configuration file describing Kubernetes resources           |
| Reconciliation | The process of aligning actual state with desired state        |
| Scaling        | Adjusting application capacity, such as the number of replicas |
| Namespace      | Organizes resources within a cluster                           |

## Summary

Kubernetes simplifies the deployment and management of containerized applications by:

- Running application workloads across a cluster of nodes.
- Scheduling pods based on resource requirements and constraints.
- Maintaining the desired number of replicas through workload controllers.
- Replacing failed pods when possible.
- Supporting application scaling and rolling updates.
- Providing service discovery and networking abstractions.
- Managing resources through declarative configuration.

**In simple terms:**

- **Containers** package and run applications.
- **Pods** provide the units Kubernetes schedules.
- **Nodes** provide the machines that run pods.
- **The control plane** coordinates cluster operations.
- **Kubernetes controllers** work to maintain the desired state.

Kubernetes brings these components together to help applications run consistently, scale when needed, and recover from supported failures.

## What makes this version better?

- Better learning sequence: It progresses from the definition of Kubernetes to clusters, pods, workloads, and application deployment.
- More practical: It includes a working Deployment manifest and useful `kubectl` commands.
- Technically clearer: It distinguishes containers from pods, resource requests from limits, and pod recovery from guaranteed high availability.
- README-friendly: It uses consistent headings, concise explanations, tables, and examples suitable for a GitHub repository.
