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
