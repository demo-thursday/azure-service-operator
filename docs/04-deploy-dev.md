# Step 4: Deploy to DEV with a MySQL Container

The Pet Clinic application container that we built in the last step needs to connect to MySQL to store data.

Now that we have the **Azure Service Operator** installed in our cluster, we have some options for MySQL!  The obvious options are:
* Create a native Azure managed MySQL service, or
* Use a MySQL container from the OpenShift container catalog

From an application perspective, do we need to change anything in order to use a MySQL container vs an Azure MySQL instance? So... what should we use for our DEV environment?

Most applications (especially [Quarkus](https://quarkus.io/) or Spring Boot) have connection information configured as properties that can be overridden at run time with environment variables.  This is also how our Pet Clinic application work!  You can see the [Azure configuration properties here](https://github.com/pittar/spring-petclinic/blob/master/src/main/resources/application-azure.properties).

This means as long as we use a MySQL 8 server with at `petclinic` database, we should be good to go.

So, what option should we use in our DEV environment?

##  DEV: MySQL container or service?

Although there is really no "wrong" answer here, in DEV **using a container** for our MySQL server has a number of advantages:

1. **It's fast.**  A new MySQL container can be up and running in seconds.  This can be important in CI/CD pipelines, or for running end-to-end test suites.
2. **There's no added cost.** A MySQL container runs in the OpenShift cluster you have already provisioned.  It doesn't require a new VM.  This can help save cost, especially in dev/test environments.
3. **It can be ephemeral.** You decide if the database needs persistence or not.  A database that might only exist for as long as it takes for a test suite to run may not require persistence at all, again, saving a little more money in a dev/test environment.
4. **It can be persistent.** This might seem obvious, but it's worth noting that you can use a "persistent volume claim" so that your data will survive rerstarts of the MySQL container/pod.

With this in mind, we will use a MySQL container in our DEV environment.

## Create the DEV Environment and Deploy Pet Clinic and MySQL

Once more we have [Kustomize](https://github.com/kubernetes-sigs/kustomize) come to the rescue to make it super easy to consistently deploy our environment in a GitOps way.

The following command will:
* Create a new *namespace* called `petclinic-dev`
* Create a *DeploymentConfig*, *Service*, and *Route* for the Pet Clinic app.
* Create a *DeploymentConfig*, *Service*, *Secret*, and *PersistentVolumeClaim* for MySQL

```
oc apply -k overlays/azure-dev

namespace/petclinic-dev created
secret/petclinicdb created
service/petclinicdb created
service/petclinic created
deploymentconfig.apps.openshift.io/petclinicdb created
deploymentconfig.apps.openshift.io/petclinic created
route.route.openshift.io/petclinic created
persistentvolumeclaim/petclinicdb created
```

In OpenShift developer web console, switch to the new `petclinic-dev` project (it probably be near the bottom of the project list)