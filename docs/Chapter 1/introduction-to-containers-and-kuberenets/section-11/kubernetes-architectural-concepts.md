---
title: Kubernetes Architectural Concepts
deprecated: false
hidden: false
metadata:
  robots: index
---
Kubernetes uses several servers (also called nodes) to ensure the resilience and scalability of the applications that it manages. These nodes can be physical or virtual machines. The nodes come in two types, each of which provides a different aspect of the cluster operations.

Control plane nodes provide global cluster coordination for scheduling the deployed workloads. These nodes also manage the state of the cluster configuration in response to cluster events and requests.

Compute plane nodes run the user workloads.<br />

![](https://files.readme.io/f9b0d9b748c51f27668b2af2ec08404c123a5c68ccd3f3cc91983a8f9c414495-k8s.png)

Although a server can act as both a control plane node and a compute node, the two roles are usually separated for increased stability, security, and manageability. Kubernetes clusters can range from small single-node clusters, to large clusters with up to 5,000 nodes.

Although standard clusters deploy the control plane directly on dedicated nodes, advanced architectures can use hosted control planes. A hosted control plane runs the control plane as isolated pods on a separate hosting cluster. This approach is popular for sovereign clouds, which require strict multitenant boundaries.
