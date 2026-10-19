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

Lastly, use a web browser to go to the URL, which displays the authentication page<br /><br /><br />//pending page 8
