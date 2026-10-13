---
title: 1.1.3 - Kubernetes Features
deprecated: false
hidden: false
link:
  new_tab: false
metadata:
  robots: index
---
Here is a polished, beginner-friendly, README-ready version of Kubernetes Features, with simple explanations, practical examples, and commands where useful.

Writing

# Kubernetes Features

Kubernetes provides several powerful features that simplify the deployment, scaling, networking, and management of containerized applications.

These features help applications run reliably, adapt to changing workloads, recover from failures, and receive updates with minimal disruption.

## 1. Service Discovery and Load Balancing

### What is it?

When an application runs across multiple pods and nodes, those components need a reliable way to communicate with each other.

Kubernetes provides **Service Discovery** and **Load Balancing** to simplify application communication.

- **Service Discovery:** Allows applications to find other applications using stable names instead of tracking changing pod IP addresses.
- **Load Balancing:** Distributes network traffic across the eligible pods associated with a Service.

Kubernetes also provides DNS-based service discovery, allowing applications to communicate with Services using DNS names.

### Practical example

Suppose a university portal runs three NGINX web server pods:

- NGINX Pod 1 runs on Worker Node 1.
- NGINX Pod 2 runs on Worker Node 2.
- NGINX Pod 3 runs on Worker Node 3.

Instead of connecting directly to each pod's IP address, users or other applications access the web servers through a Kubernetes Service.

The Service provides a stable network endpoint and distributes connections across eligible pods.

If pods are recreated and their IP addresses change, applications can continue using the Service name.

**Benefits:**

- Simplifies communication between applications.
- Provides a stable endpoint for accessing pods.
- Distributes traffic across multiple pod replicas.
- Helps applications communicate across different nodes.

## 2. Horizontal Scaling

### What is it?

**Horizontal scaling** means increasing or decreasing the number of application instances, usually by adding or removing pods.

Kubernetes supports horizontal scaling through workload controllers such as Deployments. The number of replicas can be adjusted manually or automatically using a Horizontal Pod Autoscaler (HPA).

### Practical example

Suppose a university portal normally runs three application pods.

During examination results day, thousands of students may access the portal simultaneously.

The application may need more replicas to handle the increased traffic.

- Normal traffic: 3 replicas.
- High traffic: 8 replicas.
- Low traffic: 2 replicas.

An administrator can change the replica count manually, or configure an HPA to adjust it based on supported metrics such as CPU utilization.

Example command:

```
kubectl scale deployment university-portal --replicas=5
```

This command sets the desired replica count to five.

Kubernetes attempts to create or remove pods to match the requested count.

**Benefits:**

- Helps applications handle changing workloads.
- Improves capacity during periods of high demand.
- Allows resources to be reduced when demand decreases.
- Supports automated scaling when appropriate metrics and configuration are available.

**Note:** Kubernetes can also support cluster autoscaling through compatible infrastructure and autoscaler components. Adding pods does not automatically mean new worker nodes will be created.

## 3. Vertical Scaling

### What is it?

**Vertical scaling** means increasing or decreasing the CPU and memory resources allocated to an application.

For example, an application container might initially request 256 MiB of memory. If the application requires more resources, its memory allocation may need to increase.

Kubernetes supports resource requests and limits to help define how much CPU and memory containers require or are allowed to consume.

Modern Kubernetes versions also support **in-place pod resizing**, which can change the CPU and memory resources allocated to a running pod without recreating the pod, when the feature and environment support it.

### Practical example

Suppose a university reporting application initially uses:

- CPU request: `250m`
- Memory request: `256Mi`

As the workload grows, the administrator may increase the allocation to:

- CPU request: `500m`
- Memory request: `512Mi`

Depending on the Kubernetes version, resource configuration, container runtime, and node capacity, these changes may be applied in place or may require a pod restart or replacement.

**Benefits:**

- Allows applications to receive more CPU or memory when needed.
- Can reduce unnecessary resource allocation.
- May avoid pod restarts when in-place resizing is supported.
- Helps administrators manage cluster resources more efficiently.

**Important:** In-place pod resizing is version- and configuration-dependent. Verify support in the target cluster before relying on it in production.

## 4. Self-Healing

### What is it?

**Self-healing** is Kubernetes' ability to respond to certain application and infrastructure failures.

Kubernetes uses health checks and workload controllers to help keep applications running according to their desired configuration.

Important mechanisms include:

- **Liveness probes:** Check whether a container should be restarted because it is unhealthy.
- **Readiness probes:** Determine whether a container is ready to receive application traffic.
- **Startup probes:** Allow applications additional time to start before other health checks take effect.
- **Workload controllers:** Create replacement pods when the actual number of replicas falls below the desired count.

### Practical example

Suppose a university portal runs three replicas.

One application container stops responding.

Depending on the failure and probe configuration, Kubernetes may restart the container. If an entire pod becomes unavailable, the Deployment's controllers work to restore the desired replica count.

If a worker node fails, Kubernetes may eventually arrange replacement pods on healthy nodes, provided the cluster has sufficient capacity and other scheduling requirements can be satisfied.

**Benefits:**

- Restarts containers when configured health checks indicate a failure.
- Replaces failed pods managed by workload controllers.
- Helps restore the desired number of application replicas.
- Can reduce the impact of certain node and application failures.

**Note:** Self-healing does not guarantee zero downtime. Recovery depends on health checks, scheduling, available resources, storage, networking, and application design.

## 5. Automated Rollouts and Rollbacks

### What is it?

Kubernetes supports controlled application updates through workload resources such as Deployments.

A **rollout** gradually replaces application replicas running an older version with replicas running a newer version.

A **rollback** returns a Deployment to a previous revision when a new rollout needs to be reversed.

### Practical example

Suppose the university portal currently runs version `v1`.

The development team releases version `v2`, containing new features and bug fixes.

Kubernetes can gradually replace the old pods with new pods according to the Deployment's update strategy.

A typical process is:

1. The administrator updates the container image.
2. Kubernetes creates new pods using the updated image.
3. The Deployment gradually scales down old replicas and scales up new replicas.
4. Readiness checks help determine when new pods are ready.
5. The rollout completes when the desired updated replicas are available and the rollout conditions are satisfied.

If the new version causes problems, an administrator can initiate a rollback:

```
kubectl rollout undo deployment/university-portal
```

Check rollout status:

```
kubectl rollout status deployment/university-portal
```

**Benefits:**

- Supports controlled application updates.
- Can reduce disruption during deployments.
- Provides a mechanism for reverting to an earlier Deployment revision.
- Helps teams release application changes more consistently.

**Important:** Rollback does not automatically happen for every application error. Readiness probes and Deployment progress settings help Kubernetes assess rollout progress, but application-level monitoring and release procedures are still important.

## 6. Secrets and Configuration Management

### What is it?

Applications require configuration settings to operate correctly. These settings may include API endpoints, database connection details, feature flags, and credentials.

Kubernetes provides two important resources for managing this information:

- **ConfigMap:** Stores non-sensitive configuration data.
- **Secret:** Stores sensitive information such as passwords, tokens, and credentials.

These resources allow configuration to be managed separately from the container image.

### Practical example

Suppose the university portal connects to a database.

Its configuration might include:

- Database hostname: `university-db`
- Database port: `5432`
- Database username and password

The hostname and port can be provided through a ConfigMap, while credentials can be provided through a Secret.

This allows configuration to be changed without rebuilding the application container image.

**Benefits:**

- Separates application configuration from application code.
- Reduces the need to embed credentials in container images.
- Supports different settings for development, testing, and production.
- Makes configuration reusable across application deployments.

**Security note:** Kubernetes Secrets are not automatically encrypted in etcd by default. Base64 encoding is not encryption. Production environments should consider encryption at rest, least-privilege access through RBAC, and external secret-management systems where appropriate.

## 7. Operators

### What is it?

An **Operator** is a Kubernetes extension that uses custom resources and controllers to automate the management of an application.

Operators are especially useful for complex applications that require ongoing operational tasks, not just initial deployment.

They extend Kubernetes' declarative model by encoding application-specific operational knowledge into software.

### Practical example

Suppose a university needs to run a database cluster.

Manually managing the database might require an administrator to:

- Deploy database instances.
- Configure replication.
- Monitor database health.
- Handle certain failures.
- Manage backups and recovery procedures.

A database Operator can automate some or all of these tasks, depending on its implementation.

For example, an administrator might create a custom resource specifying the desired database configuration. The Operator observes that resource and creates or updates the Kubernetes resources needed to achieve the requested state.

It may also manage database replication, backups, and recovery when those capabilities are implemented.

**Benefits:**

- Automates application-specific operational tasks.
- Simplifies the deployment and management of complex systems.
- Extends Kubernetes' declarative configuration model.
- Can reduce repetitive manual administration.

**Note:** Operators do not automatically provide every management feature. Their capabilities depend on the specific Operator, its configuration, and the application it manages.

## Summary

Kubernetes provides several features that help teams manage containerized applications efficiently.

| Feature                              | Main purpose                                                    | Example                                                            |
| ------------------------------------ | --------------------------------------------------------------- | ------------------------------------------------------------------ |
| Service discovery and load balancing | Provides stable access to applications and distributes traffic  | Route connections to NGINX pods                                    |
| Horizontal scaling                   | Adds or removes application replicas                            | Increase portal replicas during examination results day            |
| Vertical scaling                     | Adjusts CPU and memory resources                                | Increase memory allocation for a reporting application             |
| Self-healing                         | Responds to supported application and infrastructure failures   | Restart an unhealthy container or replace a failed pod             |
| Automated rollouts and rollbacks     | Manages application updates and supports reverting revisions    | Deploy version `v2` and roll back if necessary                     |
| Secrets and configuration management | Separates configuration and credentials from application images | Store database settings in a ConfigMap and credentials in a Secret |
| Operators                            | Automates application-specific operations                       | Manage database replication and backups                            |

### Final takeaway

Kubernetes is more than a container orchestration platform. It provides mechanisms for networking, scaling, recovery, deployment, configuration, and application lifecycle management.

These features help teams operate applications more consistently and efficiently, although reliable production operation still depends on appropriate configuration, security controls, monitoring, and infrastructure.
