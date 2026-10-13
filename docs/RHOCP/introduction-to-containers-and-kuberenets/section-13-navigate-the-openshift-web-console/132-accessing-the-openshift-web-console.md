---
title: 1.3.2 - Accessing the OpenShift Web Console
excerpt: Page 8 Continuation
deprecated: false
hidden: false
metadata:
  robots: index
---
You access the web console by using any modern web browser. The web console URL is generally configurable, and the cluster administrator provides it to you. If the cluster administrator provides the cluster API endpoint instead of the web console URL, then you can discover the web console address by using the oc command-line interface (CLI).

From a terminal, authenticate to the cluster by using the oc login -u <USERNAME> \<API_ENDPOINT>:<PORT> command:

\[user\@host \~]$ oc login -u developer [https://api.lab.example.com:6443](https://api.lab.example.com:6443)
Console URL: [https://api.lab.example.com:6443/console](https://api.lab.example.com:6443/console)
Authentication required for [https://api.lab.example.com:6443](https://api.lab.example.com:6443) (openshift)
Username: developer
Password: developer
Login successful.
...output omitted...
The oc login command output displays the Console URL shortcut. This API-based endpoint automatically redirects your web browser to the web console address that the console operator configured.

Alternatively, retrieve the direct web console URL by running the oc whoami --show-console command:

\[user\@host \~]$ oc whoami --show-console
[https://console-openshift-console.apps.lab.example.com](https://console-openshift-console.apps.lab.example.com)<br /><br />Lastly, use a web browser to go to the URL, which displays the authentication page:
