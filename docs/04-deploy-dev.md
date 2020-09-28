# Step 3: Build the Pet Clinic Demo App

Although it is still pretty cool, simply creating a native Azure service from OpenShift does not make for a great demo.  We need an application to consumer this Azrue service to really show the value of this integration!

Even better, how about showing how you can use both a *"containerized"* service as well as the *"native/managed"* equivalent!

The application were are going to build can connect to a MySQL database to save data.  In our **DEV** environment we will use a MySQL container.  In our **PROD** environment we will use an Azure native managed MySQL service.

Why both?  See if you can think of why using a MySQL container in *non-prod* environments might have interesting advantages, and why a managed service in *prod* is probably the way to go.  We'll discuss the reasoning a bit later, but for now let's build this app!

## Create the "CI/CD" Project and the Build

```
oc apply -k overlays/cicd

namespace/cicd created
rolebinding.rbac.authorization.k8s.io/cicd-image-puller created
buildconfig.build.openshift.io/petclinic created
```

This created a new Project/namespace called `cicd`.  We can switch into it:
```
oc project cicd
```

```
oc status -n cicd

In project cicd on server https://api.rn9nygs6.canadacentral.aroapp.io:6443

bc/petclinic source builds https://github.com/pittar/spring-petclinic on openshift/java:8
  -> istag/petclinic:latest
  build #1 running for 37 seconds - 88d2c83: Update application-azure.properties (Andrew Pitt <apitt@redhat.com>)
```

In the OpenShift web console (Developr View), you can use the **Project** drop-down to select the new `cicd` project.

Not much to see here once the build is complete.  A new image is now available in the "CI/CD" project named `petclinic:latest`.  We will refer to this in the next two steps.

Time to deploy to *DEV*!

[Step 4: Deploy Pet Clinic connected to a MySQL Container]
