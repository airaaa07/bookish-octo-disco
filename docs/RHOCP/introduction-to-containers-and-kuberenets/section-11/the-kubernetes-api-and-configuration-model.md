---
title: 1.1.5 - The Kubernetes API and Configuration Model
deprecated: false
hidden: false
link:
  new_tab: false
metadata:
  robots: index
---
# The Kubernetes API and Configuration Model

## 1. Introduction

Kubernetes provides a standard way to define, create, manage, and monitor applications running in a cluster.

Instead of manually configuring every application component, administrators define the desired configuration using **Kubernetes resources**.

These resources describe what should run in the cluster, how many replicas are required, which container images to use, and what resources the applications need.

Kubernetes processes these definitions and continuously works to make the actual cluster state match the desired state.

For example, an administrator can define a university portal that requires three replicas, specify its container image, and configure its CPU and memory requirements.

Kubernetes then manages the resources needed to run the application.

## 2. Kubernetes API and Resources

The **Kubernetes API** provides the interface through which users, tools, and cluster components interact with Kubernetes resources.

Everything managed through the Kubernetes API is represented using resource objects.

Examples include:

- **Pod:** Represents one or more containers that run together.
- **Deployment:** Manages replicated application pods and application updates.
- **Service:** Provides a stable network endpoint for accessing application pods.
- **ConfigMap:** Stores non-sensitive application configuration.
- **Secret:** Stores sensitive configuration data, such as credentials.

Tools such as `kubectl` allow administrators to create, view, update, and delete these resources.

For example:

```
kubectl get pods
```

Lists pods in the current namespace.

```
kubectl get deployments
```

Lists Deployments in the current namespace.

```
kubectl get services
```

Lists Services in the current namespace.

Kubernetes also supports custom resources, which allow extensions to introduce additional resource types. These can often be managed using the same API patterns and tools as built-in resources.

**In simple terms:** The Kubernetes API is the interface for managing cluster resources, and resources describe the objects that make up an application.

## 3. Declarative Configuration Model

Kubernetes supports two general approaches to managing resources: imperative and declarative.

### 3.1 Imperative management

In imperative management, administrators issue commands describing the actions to perform.

For example:

```
kubectl create deployment university-portal --image=nginx
kubectl scale deployment university-portal --replicas=3
```

These commands create a Deployment and then set its desired replica count to three.

This approach can be useful for quick tasks, testing, and troubleshooting.

### 3.2 Declarative management

In declarative management, administrators describe the desired state in a configuration file rather than specifying every operation individually.

For example, a YAML manifest can define a Deployment with three replicas and a specific container image.

Kubernetes processes the manifest and works to achieve the requested state.

This approach is particularly useful for applications that need repeatable deployments and long-term maintenance.

### Example manifest

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
```

Save this file as `deployment.yaml`.

Apply the configuration:

```
kubectl apply -f deployment.yaml
```

Check the Deployment:

```
kubectl get deployment university-portal
```

Kubernetes attempts to maintain three replicas of the application.

If the manifest is updated to request five replicas, applying it again tells Kubernetes to change the desired replica count.

```
kubectl apply -f deployment.yaml
```

Kubernetes then works to create the additional replicas.

## 4. Declarative vs. Imperative Management

| Feature         | Imperative                                           | Declarative                                   |
| --------------- | ---------------------------------------------------- | --------------------------------------------- |
| Approach        | Specifies actions to perform                         | Specifies the desired state                   |
| Typical method  | Individual `kubectl` commands                        | YAML or JSON manifests                        |
| Example         | Scale a Deployment using a command                   | Define `replicas: 3` in a manifest            |
| Repeatability   | Depends on the commands executed                     | Configuration can be reapplied consistently   |
| Version control | Possible, but commands need to be tracked separately | Manifests can be stored and reviewed directly |

Declarative management makes it easier to reproduce deployments, review configuration changes, and maintain consistency between environments.

## 5. Resource Definitions and Version Control

Kubernetes resources can be represented as YAML or JSON documents.

These files can be stored in version control systems such as Git, allowing administrators to track and review changes over time.

For example, a repository might contain:

```
kubernetes/
├── deployment.yaml
├── service.yaml
├── configmap.yaml
└── namespace.yaml
```

Each file defines a different part of the application configuration.

Using version control provides several advantages:

- **Change tracking:** Records changes to application configuration.
- **Collaboration:** Allows team members to review and contribute changes.
- **Repeatability:** Helps recreate environments using stored manifests.
- **Rollback support:** Makes it easier to retrieve earlier configuration versions.
- **Maintainability:** Keeps infrastructure configuration organized alongside application code.

Kubernetes supports importing and exporting resource representations through its API and command-line tools. Exported representations may contain system-managed fields, so they should be reviewed before being reused as declarative manifests.

## 6. Controllers and Reconciliation

Kubernetes uses **controllers** to monitor resources and work toward maintaining their desired state.

A controller compares the desired state with the actual state and takes corrective action when necessary. This process is known as **reconciliation**.

### Example: Maintaining replicas

Suppose a Deployment specifies three replicas.

The desired state is three application pods, but only two are currently available.

The Deployment's controllers work to restore the desired replica count by creating a replacement pod.

Once the replacement pod starts and becomes ready, the application can return to its expected operating state.

This reconciliation process continues throughout the application's lifecycle.

### Desired state vs. actual state

- **Desired state:** What the resource configuration specifies.
- **Actual state:** What currently exists in the cluster.
- **Reconciliation:** The process of working to bring actual state in line with desired state.

## 7. How Kubernetes Applies Configuration Changes

Changes to Kubernetes resources are not always effective immediately.

After a configuration update, Kubernetes components process the change and perform the required operations. The time required depends on the type of change, available resources, application startup time, and cluster conditions.

For example, suppose a Deployment is updated to use a container image with higher memory requirements.

Kubernetes does not automatically move a running pod to another node simply because its memory requirements have increased.

Depending on the change, the workload controller may create replacement pods, and the scheduler may place those pods on suitable nodes. If the new pods cannot fit on available nodes, they may remain pending until the problem is resolved.

When a Deployment uses an appropriate rolling update strategy, Kubernetes can wait for new pods to become ready before removing corresponding old replicas, subject to the Deployment configuration.

This helps reduce service disruption during updates.

**Important:** Kubernetes does not guarantee that every configuration change will succeed automatically. Scheduling constraints, insufficient resources, application errors, and storage limitations can prevent the desired state from being reached.

## 8. Kubernetes Namespaces

A **Namespace** is a Kubernetes resource used to organize resources within a cluster.

Namespaces are useful when a cluster hosts multiple applications, teams, or environments.

For example, an organization might create the following namespaces:

- `development` — for development workloads.
- `testing` — for testing and validation.
- `production` — for live applications.

Resources such as Deployments, Pods, Services, ConfigMaps, and Secrets can be organized within these namespaces.

### Example commands

Create a namespace:

```
kubectl create namespace development
```

Create a Deployment inside that namespace:

```
kubectl create deployment university-portal \
  --image=nginx \
  --namespace=development
```

List pods in the namespace:

```
kubectl get pods -n development
```

List namespaces:

```
kubectl get namespaces
```

Namespaces help organize resources, but they do not automatically provide complete security isolation.

Some resources are cluster-scoped rather than namespaced, and access control must be configured separately.

## 9. Namespaces and Access Control

Kubernetes supports **Role-Based Access Control (RBAC)** to control who can access or modify resources.

Administrators can create Roles and RoleBindings to grant specific permissions within a namespace.

For example, a development team might be allowed to view and update Deployments in the `development` namespace without receiving the same permissions in `production`.

This approach helps apply the principle of least privilege, giving users only the permissions they need.

Namespaces, RBAC, and resource quotas can work together to improve the organization and management of shared clusters.

## 10. How the Kubernetes Configuration Model Works

The following Mermaid diagram illustrates the declarative workflow.

```
flowchart TD
    A["Administrator defines desired state"]
    B["YAML or JSON manifest"]
    C["kubectl apply"]
    D["Kubernetes API Server"]
    E[("Cluster state stored in etcd")]
    F["Controllers reconcile resources"]
    G["Scheduler selects suitable nodes"]
    H["Kubelet and container runtime run pods"]
    I["Actual state approaches desired state"]

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I -. "Continuous reconciliation" .-> F
```

![](https://files.readme.io/61bdb5e68285c0f22627fe22fdd4709db32264ce0bd2e98301bba28accda86e4-k8sn.png)

The workflow can be summarized as follows:

1. The administrator defines the desired configuration.
2. The manifest is submitted through `kubectl` to the Kubernetes API Server.
3. Kubernetes processes the requested resource changes and stores cluster state.
4. Controllers work to reconcile the resources.
5. The scheduler selects suitable nodes for pods that require placement.
6. Kubelets work with container runtimes to run the assigned workloads.
7. Kubernetes continues monitoring and reconciling resources as conditions change.

## Summary

The Kubernetes API and configuration model provides a consistent way to manage applications across a cluster.

Its key concepts are:

- **Kubernetes API:** Provides the interface for interacting with cluster resources.
- **Resources:** Represent applications, workloads, networking, configuration, and other cluster objects.
- **Declarative configuration:** Describes the desired state using YAML or JSON manifests.
- **Controllers:** Continuously work to maintain the desired state through reconciliation.
- **Version control:** Helps track, review, and maintain configuration changes.
- **Namespaces:** Organize resources and provide a scope for namespace-level access control.

**In simple terms:** Administrators describe what they want Kubernetes to run, submit that configuration through the API, and rely on Kubernetes controllers to work toward maintaining the requested state.

<br /><br />

<br /><br />

#### **References**<br />**External Resources**

<Anchor target="_blank" href="https://kubernetes.io/docs/concepts/containers/">Containers</Anchor>

<Anchor target="_blank" href="https://kubernetes.io/docs/concepts/overview/">Kubernetes Concepts Overview</Anchor>

<Anchor target="_blank" href="https://kubernetes.io/docs/concepts/overview/#what-kubernetes-is-not">What Kubernetes Is Not</Anchor>

<Anchor target="_blank" href="https://kubernetes.io/docs/setup/best-practices/cluster-large/">Considerations for Large Clusters</Anchor>
