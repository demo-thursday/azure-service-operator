# Step 5: Deploy to PROD with a Managed Azure MySQL Service

Our Pet Clinic app has been tested in DEV and passed with flying colours.  Now it's time to deploy it to PROD!

In our dev/test environments, we decided to use MySQL container images instead of a managed service because:
* A MySQL container starts much faster than a managed Azure MySQL instance, which requires a new VM.
* A MySQL container doesn't incur any extra VM cost.  It runs on your existing OpenShift cluster.
* Dev/Test environments do not have critical data.  Speed and cost management are prioritized over high availability and ease of upgrdes.

Production, however, is a different story!

##  PROD: MySQL container or service?

In production, most teams prioritize stability, high availability and ease of upgrades over speed to provision and cost.  For this reason, using a managed Azure MySQL database instance is the obvious choice.  Thanks to the new Azure Service Operator, you can have the same GitOps experience with native Azure services as you can with a container image.  Let's fire up this environment!

## Create the PROD Environment and Deploy Pet Clinic and MySQL

We will use the same Kustomie process as before to create our PROD environment, but this time aa new Azure MySQL instance will be provisioned instead of a MySQL container instance.

If you decide to dive into the supporting Kubernetes manifests (in the `overlays` and `base` directories), you will see that there is really no copy/paste that goes on between environments.  The Pet Clinic `DeploymentConfig` references the [same file](https://github.com/demo-thursday/azure-service-operator/blob/master/base/petclinic/petclinic-deploymentconfig.yaml) in both envioronments, with the difference being the [environment variables that are patched in](https://github.com/demo-thursday/azure-service-operator/blob/master/overlays/azure-prod/environment-patch.yaml).  No copy/paste here!

Without further ado, we will create our PROD enviornment with a single command:

```
oc apply -k overlays/azure-prod

namespace/petclinic-prod created
service/petclinic created
deploymentconfig.apps.openshift.io/petclinic created
mysqldatabase.azure.microsoft.com/petclinic created
mysqlfirewallrule.azure.microsoft.com/pitt-mysql-aro-fw created
mysqlserver.azure.microsoft.com/pitt-mysql-aro created
route.route.openshift.io/petclinic created
```

This is now creating:
* The `petclinic-prod` namespace.
* The Pet Clinic `DeploymentConfig`, `Service`, and `Route`.
* An Azure `MySQLServer`, `MySQLDatabase`, an `MySQLFirewallRule`

Take a look at the `mysql` custom resource (short for `MySQLServer`) to see what his happening:

```
oc get mysqls

NAME             PROVISIONED   MESSAGE
pitt-mysql-aro                 request submitted to Azure
```

The request for a new `MySQLServer` has been submitted to Azure!

In the OpenShift developer web console, when you switch to the new `petclinic-prod` project, you will see there is only one Pod - Pet Clinic.  This makes sense, since there is no MySQL container running in in this project.

If you login into your Microsoft Azure Portal, take a look at the `databases` *Resource Group* and check the *Activity Log*.   It will take 2-3 minutes for any activity to actually appear after you run the `oc apply` command above.  This is normal.

It is also normal to see a few failures before everything succeeds.  With the command above, the MySQL Server, Database, and Firewall Rule are all created at the same time.  Obviously, a database and a firewall rule can't be created until the server is ready!  This is fine, though.  Both the database and firewall rule creation will retry once the server is ready.  Your activty log will look something like this when everything is ready.

You will also know when your database is ready by looking at the Resource Group Overview.  You should see your new database instance listed.

Overall, it will probably take at least 5 minutes for your database to be provisioned and come online.

## Star Pet Clinic in PROD

Ok, we we have a production Azure MySQL database!  Time to start-up the app!

From the `petclinic-prod` project in the OpenShift developer web console:
* Click on the `petclinic` pod (the big circle) and scale up the app by selecting **Start Rollout** From the **Actions** drop down.


Notice there is reference to a `Secret` named `pitt-mysql-aro`?  This was created by the Azure Service Operator!  As part of the provisioning process, the operator creates a `Secret` in your namespace with the connection details that your application will need.  This includes server URL, username, and password.  Neat, eh?
Notice there is reference to a `Secret` named `pitt-mysql-aro`?  This was created by the Azure Service Operator!  As part of the provisioning process, the operator creates a `Secret` in your namespace with the connection details that your application will need.  This includes server URL, username, and password.  Neat, eh?

[Step 6: Cleanup and Conclusion](06-cleanup-and-conclusion.md)
