A/B Deployment
A/B deployments generally imply running two (or more) versions of the application code or application configuration at the same time for testing or experimentation purposes.

The simplest form of an A/B deployment is to divide production traffic between two or more distinct shards — a single group of instances with homogeneous configuration and code.

More complicated A/B deployments may involve a specialized proxy or load balancer that assigns traffic to specific shards based on information about the user or application (all "test" users get sent to the B shard, but regular users get sent to the A shard).

A/B deployments can be considered similar to A/B testing, although an A/B deployment implies multiple versions of code and configuration, where as A/B testing often uses one code base with application specific checks.

When to Use an A/B Deployment
When you want to test multiple versions of code or configuration, but are not planning to roll one out in preference to the other.

When you want to have different configuration in different regions.

An A/B deployment groups different configuration and code — multiple shards — together under a single logical endpoint. Generally, these deployments, if they access persistent data, should properly deal with N-1 compatibility (the more shards you have, the more possible versions you have running). Use this pattern when you need separate internal configuration and code, but end users should not be aware of the changes.

A/B Deployment Example
All A/B deployments are composite deployment types consisting of multiple deployment configurations.

ONE SERVICE, MULTIPLE DEPLOYMENT CONFIGURATIONS

OpenShift Container Platform, through labels and deployment configurations, supports multiple simultaneous shards being exposed through the same service. To the consuming user, the shards are invisible. An example of the simplest possible sharding is described below:

Create the first shard of the application based on the example deployment images:

$ oc new-app openshift/deployment-example --name=ab-example-a --labels=ab-example=true SUBTITLE="shard A"
Edit the newly created shard to set a label ab-example=true that will be common to all shards:

$ oc edit dc/ab-example-a
In the editor, add the line ab-example: "true" underneath spec.selector and spec.template.metadata.labels alongside the existing deploymentconfig=ab-example-a label. Save and exit the editor.

Trigger a re-deployment of the first shard to pick up the new labels:

$ oc deploy ab-example-a --latest
Create a service that uses the common label:

$ oc expose dc/ab-example-a --name=ab-example --selector=ab-example=true
If you have the router installed, make the application available via a route (or use the service IP directly):

$ oc expose svc/ab-example
Browse to the application at ab-example.<project>.<router_domain> to verify you see the v1 image.

Create a second shard based on the same source image as the first shard but different tagged version, and set a unique value:

$ oc new-app openshift/deployment-example:v2 --name=ab-example-b --labels=ab-example=true SUBTITLE="shard B" COLOR="red"
Edit the newly created shard to set a label ab-example=true that will be common to all shards:

$ oc edit dc/ab-example-b
In the editor, add the line ab-example: "true" underneath spec.selector and spec.template.metadata.labels alongside the existing deploymentconfig=ab-example-b label. Save and exit the editor.

Trigger a re-deployment of the second shard to pick up the new labels:

$ oc deploy ab-example-b --latest
At this point, both sets of pods are being served under the route. However, since both browsers (by leaving a connection open) and the router (by default, through a cookie) will attempt to preserve your connection to a back-end server, you may not see both shards being returned to you. To force your browser to one or the other shard, use the scale command:

$ oc scale dc/ab-example-a --replicas=0
Refreshing your browser should show v2 and shard B (in red).

$ oc scale dc/ab-example-a --replicas=1; oc scale dc/ab-example-b --replicas=0
Refreshing your browser should show v1 and shard A (in blue).

If you trigger a deployment on either shard, only the pods in that shard will be affected. You can easily trigger a deployment by changing the SUBTITLE environment variable in either deployment config oc edit dc/ab-example-a or oc edit dc/ab-example-b. You can add additional shards by repeating steps 5-7.
