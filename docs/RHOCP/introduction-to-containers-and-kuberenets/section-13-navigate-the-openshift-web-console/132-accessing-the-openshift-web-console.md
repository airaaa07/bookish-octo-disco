---
title: 1.3.2 - Accessing the OpenShift Web Console
excerpt: Page 8 Continuation
deprecated: false
hidden: false
metadata:
  robots: index
---
You access the web console by any modern web browser. The web console URL is generally configurable, and you can discover the address for your cluster web console by using the `oc` command-line interface (CLI). From a terminal, you must first authenticate to the cluster via the CLI by using the `oc login -u <USERNAME> -p <PASSWORD> <API_ENDPOINT>:<PORT>` command:

```
[user@host ~]$ oc login -u developer -p developer https://api.ocp4.example.com:6443
Login successful.

...output omitted...
```

Then, you execute the `oc whoami --show-console` command to retrieve the web console URL:

```
[user@host ~]$ oc whoami --show-console
https://console-openshift-console.apps.ocp4.example.com
```

Lastly, use a web browser to go to the URL, which displays the authentication page<br /><br /><br />//pending page 8<br /><br /><br />

Figure 1.6: The OpenShift authentication page
Using the credentials for your cluster access brings you to the home page for the web console.

Figure 1.7: The OpenShift home page
Web Console Perspectives
The OpenShift web console provides the Administrator and Developer console perspectives. The sidebar menu layout and the features that it displays differ between using the two console perspectives. The first item on the console sidebar menu is the perspective switcher, to switch between the Administrator and the Developer perspectives.

Figure 1.8: The OpenShift web console perspective switcher
Each perspective presents the user with different menu categories and pages that cater to the needs of the two separate personas. The Administrator perspective focuses on cluster configuration, deployments, and operations of the cluster and running workloads. The Developer perspective pages focus on creating and running applications.

Note
In clusters with the Red Hat OpenShift Virtualization operator deployed, a Virtualization perspective is available. The Virtualization perspective pages focus on creating and managing virtual machines.

OpenShift Web Console Layout
From the web console home page, the primary navigation method is through the sidebar. The sidebar organizes cluster functions and administration into several major categories. By selecting a category from the sidebar, the category expands to reveal the various areas that each provide specific cluster information, configuration, or functionality.

Note
An initial log in to the web console presents the option for a short informational tour. Click Skip Tour if you prefer to dismiss the tour option at this time.

By default, the console displays the Home → Overview page, which provides a quick glimpse of initial cluster configurations, documentation, and general cluster status. Go to Home → Projects to list all projects in the cluster that are available to the credentials in use.

You might initially peruse the Operators → OperatorHub page, which provides access to the collection of operators that are available for your cluster.

Figure 1.10: The OpenShift OperatorHub
By adding operators to the cluster, you can extend the features and functions that your OpenShift cluster provides. Use the search filter to find the available operators to enhance the cluster and to supply the OpenShift aspects that you require.

By clicking the link on the Operator Hub page, you can peruse the Developer Catalog.

Select any project, or use the search filter to find a specific project, to visit the Developer Catalog for that project, where shared applications, services, event sources, or source-to-image builders are available.

Figure 1.12: The Developer Catalog
After finding the preferred additions for a project, a cluster administrator can further customize the content that the catalog provides. By adding the necessary features to a project from this approach, developers can customize features to provide an ideal application deployment.

Red Hat OpenShift Key Concepts
When you navigate the OpenShift web console, it is useful to know some introductory OpenShift, Kubernetes, and container terminology. The following list includes some basic concepts that can help you to navigate the OpenShift web console.

Pods: The smallest unit of a Kubernetes-managed containerized application. A pod consists of one or more containers.

Deployments: The operational unit that provides granular management of a running application.

Projects: A Kubernetes namespace with additional annotations that provide multitenancy scoping for applications.

Routes: Networking configuration to expose your applications and services to resources outside the cluster.

Operators: Packaged Kubernetes applications that extend cluster functions.

These concepts are covered in more detail throughout the course. You can find these concepts throughout the web console as you explore the features of an OpenShift cluster from the graphical environment.

References
For more information about the OpenShift web console, refer to Red Hat OpenShift Container Platform Web Console documentation at [https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/web_console/index](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/web_console/index)
