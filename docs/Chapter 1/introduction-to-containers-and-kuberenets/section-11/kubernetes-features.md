---
title: 1.1.3 - Kubernetes Features
deprecated: false
hidden: false
link:
  new_tab: false
metadata:
  robots: index
---
Kubernetes Features
Kubernetes offers the following features on top of a container infrastructure:

Service discovery and load balancing
Distributing applications over multiple nodes can complicate communication between applications.

Kubernetes automatically configures networking and provides a DNS service for pods. With these features, pods can communicate with services from other pods transparently across nodes by using only hostnames instead of IP addresses. Multiple pods can back a service for performance and reliability. For example, Kubernetes can evenly split incoming requests to an NGINX web server by considering the availability of the NGINX pods.

Horizontal scaling
Kubernetes can monitor the load on a service, and create or delete pods to adapt to the load. With some configurations, Kubernetes can also provision nodes dynamically to accommodate cluster load.

Vertical scaling
Starting with Kubernetes 1.35 and OpenShift Container Platform 4.22, Kubernetes can resize the CPU and memory that are allocated to running pods without restarting them. This feature is known as in-place pod resize.

Self-healing
If applications declare health check procedures, then Kubernetes can monitor, restart, and reschedule failing or unavailable applications. Self-healing protects applications both from internal failure (the application stops unexpectedly) or external failure (the node that runs the application becomes unavailable).

Automated rollout
Kubernetes can gradually roll out updates to your application's containers. If something goes wrong during the rollout, then Kubernetes can roll back to the previous version of the deployment. Kubernetes routes requests to the rolled out version of the application, and deletes pods from the previous version when the rollout completes.

Secrets and configuration management
You can manage the configuration settings and secrets of your applications without requiring changes to containers. Application secrets can be usernames, passwords, and service endpoints, or any configuration setting that must be kept private.

Note
Kubernetes does not encrypt secrets.

Operators
Operators are packaged Kubernetes applications that can manage Kubernetes workloads in a declarative manner. For example, an operator can define a database server resource. If the user creates a database resource, then the operator creates the necessary workloads to deploy the database, configure replication, and back up automatically.