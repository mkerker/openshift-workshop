### Rolling Deployments (with Canary checks)

A rolling deployment slowly replaces instances of the previous version of an application (in OpenShift and Kubernetes, pods) with instances of the new version of the application. A rolling deployment typically waits for new pods to become *ready* via a *readiness check* before scaling down the old components. If a significant issue occurs, the rolling deployment can be aborted.

All rolling deployments in OpenShift are *canary* deployments - a new version is tested (the canary) before all of the old instances are replaced. If the readiness check never succeeds, the deployment will be automatically rolled back. The readiness check is part of the application code, and may be as sophisticated as necessary to ensure the new instance is ready to be used. If you need to implement more complex checks of the application (such as sending real user workloads to the new instance), consider implementing a custom deployment or a using a blue-green deployment strategy.


#### When to use a rolling deployment?

* When you want to take no downtime during an application update
* When your application supports having old code and new code running at the same time

A rolling deployment means you to have both old and new versions of your code running at the same time. This typically requires that your application handle **N-1** compatibility - that data stored by the new version can be read and handled (or gracefully ignored) by the old version of the code. This can take many forms - data stored on disk, in a database, in a temporary cache, or that is part of a user's browser session. While most web applications can support rolling deployments, it's important to test and design your application to handle it.


#### Example

Rolling deployments are the default in OpenShift. To see a rolling update, follow these steps:

1.  Create an application based on the example deployment images:

        $ oc new-app openshift/deployment-example

    If you have the router installed, make the application available via a route (or use the service IP directly)

        $ oc expose svc/deployment-example

    Browse to the application at `deployment-example.<project>.<router_domain>` to verify you see the 'v1' image.

2.  Scale the deployment config up to three instances:

        $ oc scale dc/deployment-example --replicas=3

3.  Trigger a new deployment automatically by tagging a newer version of the example image as the `latest` tag:

        $ oc tag --source=docker openshift/deployment-example:v2 deployment-example:latest

4.  In your browser, refresh the page until you see the 'v2' image.

5.  If you are using the CLI, the `oc deploy deployment-example` command will show you how many pods are on version 1 and how many are on version 2. In the web console, you should see the pods slowly being added to v2 and removed from v1.

During the deployment process, the new replication controller is incrementally scaled up. Once the new pods are marked as *ready* (because they pass their readiness check), the deployment process will continue. If the pods do not become ready, the process will abort, and the deployment config will be rolled back to its previous version.


#### Rolling deployment variants

Coming soon!


### Recreate Deployment

A recreate deployment removes all pods from the previous deployment before creating new pods in the new deployment.


#### When to use a recreate deployment?

* When you must run migrations or other data transformations before your new code starts
* When you don't support having new and old versions of your application code running at the same time

A recreate deployment incurs downtime, because for a brief period no instances of your application are running. However, your old code and new code do not run at the same time.


#### Example

You can configure a recreate deployment by updating a deployment config. The `recreate-example.yaml` file in this directory contains the same scenario we tried above, but configured to recreate. The `strategy` `type` field is set to `Recreate`.

1.  Create the example:

        $ oc create -f examples/deployment/recreate-example.yaml

    Browse to the application at `recreate-example.<project>.<router_domain>` to verify you see the 'v1' image.

2.  Trigger a new deployment automatically by tagging a new version of the example as the `latest` tag:

        $ oc tag recreate-example:v2 recreate-example:latest

3.  In your browser, refresh the page until you see the 'v2' image.

4.  If you are using the CLI, the `oc deploy recreate-example` command will show you how many pods are on version 1 and how many are on version 2. In the web console, you should see all old pods removed, and then all new pods created.
