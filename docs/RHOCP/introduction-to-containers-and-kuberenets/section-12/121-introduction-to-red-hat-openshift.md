---
title: 1.2.1 - Introduction to Red Hat OpenShift
deprecated: false
hidden: false
metadata:
  robots: index
---
Kubernetes provides many features to run container workloads on clusters. However, for some features, Kubernetes provides only the building blocks to implement them, because different environments might need different solutions. Kubernetes administrators can select an existing solution or implement their own solution to fit their specific requirements.

OpenShift uses the extensibility of Kubernetes to build a complete solution by adding the following features to a Kubernetes cluster:

Integrated developer workflow
When running applications on Kubernetes, you need to build and store container images for the applications. OpenShift integrates a built-in container registry, CI/CD pipelines, and S2I, a tool to build artifacts from source repositories to container images.

Observability
To achieve the intended reliability, performance, and availability of applications, cluster administrators might need additional tools to prevent and solve issues. OpenShift includes monitoring and logging services for both your applications and the cluster.

Server management
Kubernetes requires an operating system to run on that must be installed, configured, and maintained.

OpenShift provides installation and update procedures for many scenarios.

Additionally, hosts in a cluster use Red Hat Enterprise Linux CoreOS (RHEL CoreOS) as the underlying operating system. RHEL CoreOS is an immutable operating system that is optimized for running containerized applications. RHEL CoreOS uses the same kernel and packages as Red Hat Enterprise Linux. OpenShift also provides features to manage RHEL CoreOS following the Kubernetes configuration model.

OpenShift also brings unified tools and a graphical web console to manage all the different capabilities, and additional enhancements such as improved security measures.

This diagram shows many of the projects that provide the various functions within each OpenShift cluster:<br /><br />

![](https://files.readme.io/80f1e3bab121637d297d12e9855cf320bd17633a513bbe631122a89c398c5269-editions.png)
