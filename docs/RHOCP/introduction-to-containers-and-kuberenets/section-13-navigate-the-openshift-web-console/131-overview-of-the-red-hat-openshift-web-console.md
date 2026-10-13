---
title: 1.3.1 - Overview of the Red Hat OpenShift Web Console
deprecated: false
hidden: false
metadata:
  robots: index
---
The Red Hat OpenShift Web Console provides a graphical user interface to perform many administrative tasks for managing a cluster. The web console uses the Kubernetes APIs and OpenShift extension APIs to deliver a robust graphical experience. The menus, tasks, and features within the web console are always available by using the CLI. The web console provides ease of access and management for the complex tasks that cluster administration requires.

Kubernetes provides a web-based dashboard, which is not deployed by default within a cluster. The Kubernetes dashboard provides minimal security permissions, and accepts only token-based authentication. This dashboard also requires a proxy setup that limits access to the web console from only the system terminal that creates the proxy. By contrast with the stated limitations of the Kubernetes web console, OpenShift includes a fuller-featured web console.

The OpenShift web console is not related to the Kubernetes dashboard, but is a separate tool for managing OpenShift clusters. Additionally, operators can extend the web console features and functions to include more menus, views, and forms to aid in cluster administration.
