---
title: The Kubernetes API and Configuration Model
deprecated: false
hidden: false
metadata:
  robots: index
---
Kubernetes provides a model to define the workloads to run on a cluster.

Administrators define workloads in terms of resources.

All resource types work in the same way, with a uniform API. Tools such as the kubectl command can manage resources of all types, even custom resource types.

Kubernetes can import and export resources as text files. By working with resource definitions in text formats, administrators can describe their workloads instead of performing the right sequence of operations to create them. This approach is called declarative resource management.<br /><br />

![](https://files.readme.io/61bdb5e68285c0f22627fe22fdd4709db32264ce0bd2e98301bba28accda86e4-k8sn.png)

Declarative resource management can reduce the work to create workloads and improve maintainability. Additionally, when using text files to describe resources, administrators can use generic tools such as version control systems to gain further benefits such as change tracking.

To support declarative resource management, Kubernetes uses controllers that continuously track the state of the cluster and perform the necessary steps to keep the cluster in the intended state. This process means that changes to resources often require some time to be effective. However, Kubernetes can automatically apply complex changes. For example, if you increase the RAM requirements of an application that cannot be satisfied on the current node, then Kubernetes can move the application to a node with sufficient RAM. Kubernetes redirects traffic to the new instance only when the movement is complete.

Kubernetes also supports namespaces. Administrators can create namespaces, and most resources must be created inside a namespace. Besides helping organize large amounts of resources, namespaces provide the foundations for features such as resource access. Administrators can define permissions for namespaces, to allow specific users to view or modify resources.<br /><br />

References<br />External Resources

<Anchor target="_blank" href="https://kubernetes.io/docs/concepts/containers/">Containers</Anchor>

[Kubernetes Concepts Overview]()

What Kubernetes Is Not

Considerations for Large Clusters
