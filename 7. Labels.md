## Lab 7: Labels
This is a pretty simple lab, we are going to explore labels. You can use labels to organize, group, or select API objects.

For example, pods are “tagged” with labels, and then services use label selectors to identify the pods they proxy to. This makes it possible for services to reference groups of pods, even treating pods with potentially different Docker containers as related entities.

**Step 1: Labels a on pod**
In a previous lab we added our web app using a template. When we did that, OpenShift labeled our objects for us. Let’s look at the labels on our running pod.

Goto the terminal and try the following:
```
$ oc get pods
$ oc describe pod/time | more
```
You can see the Labels automatically added contain the app, deployment, and deploymentconfig. Let's add a new label to this pod.

**Step 2: Add a label**

```
$ oc label pod/time testdate=27.06.2017 testedby=mylastname
```

**Step 3: Look at the labels**

```
$ oc describe pod/time | more
```

Here's a handy way to search through all objects and look at all the labels:

```
$ oc describe all | grep -i "labels:"
```

**Summary**

That’s it for this lab, now you know that all the objects in OpenShift can be labeled. This is important because those labels can be used as part of your CI/CD process. Advanced labs will cover using labels for Blue/Green deployments and running yours apps on specific nodes (e.g. just on SSD nodes or just on east coast nodes). 
