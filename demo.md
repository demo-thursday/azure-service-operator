# Azure Service Operator Demo

The [Azure Service Operator](https://github.com/Azure/azure-service-operator) is an open source [Kubernetes Operator](https://operatorhub.io/operator/azure-service-operator) that can be installed and run on most current Kubernetes distributions, including [OpenShift Container Platform](https://www.openshift.com), and the fully managed [Azure Red Hat OpenShift](https://azure.microsoft.com/en-us/services/openshift/) service.

For this demo, it doesn't matter if you use an instance of *Azure Red Hat OpenShift*, or if you install your own self-managed [OpenShift](https://www.openshift.com/try) cluster on Azure, the instructions and results are the same.  After all, Azure Red Hat OpenShift is simply a fully managed instance of OpenShift that comes with a [99.9% financially-backed SLA](https://azure.microsoft.com/en-au/support/legal/sla/openshift/v1_0/).

In this demo we will:
* Install the Azure Service Operator in our OpenShift cluster.
* Build a container image containing a Spring Boot app that connects to a MySQL database.
    * Demonstrate a simple "source-to-image" build process to build a container image simply by referencing a git repository with the application source code.
* Deploy this application in a "dev" namespace that uses a MySQL container image.  This will demonstrate:
    * How to run the container image that we just built.
    * How quick and easy it is to spin-up a MySQL container image to use in a dev/test environment.
    * How the MySQL connection information (including url, username, and password) are injected into the container at run-time to properly externalize environment-specific configuration.
* Deploy this application in a "prod" namespace that uses an [Azure Database for MySQL](https://azure.microsoft.com/en-ca/services/mysql/) instance that is provisioned by OpenShfit!  This will demonstrate:
    * How to provision a native Azure service (in this case, a managed MySQL instance) directly from OpenShift.
    * The benefits of "configuration as code", and how it can be applied with GitOps practices.
    * How the MySQL connection information (including url, username, and password) are injected into the container at run-time to properly externalize environment-specific configuration.  This time, this information will come from a Kubernetes **Secret** that was created by the Azure Service Operator once the database was provisioned.

Let's get started!

[Step 1: Install the Azure Service Operator](docs/01-install-operator.md)



