# Azure Service Catalog in OpenShift Container Platform

This demo is based on this [OpenShift blog post](https://www.openshift.com/blog/using-the-azure-service-operator-on-openshift).

## Azure Service Operator

When installed in an OpenShift Container Platform cluster that is running on Microsoft Azure (such as Azure Red Hat OpenShift), a number of native Azure services become available to provision directly from the OpenShift console.  In this demo, we will install the Azure Service Operator in an Azure Red Hat OpenShift cluster, then provision a managed PostgreSQL database to use with our trusy Pet Clinic demo app!

## Installing the Operator

The instructions for installing this operator are detailed in the description found in the OpenShift embedded OperatorHub.  However, there are currently a few issues that need fixing with the docs.  As of verion `0.37.0`, there was a formatting issue with the `Secret` you are asked to create, and an RBAC issue that has already been reported (and has an easy work around).  For now, please follow these instructions

### Step 1: Create a Service Principal

For convenience, set your `tenant_id` and `subscription_id` as env vars.  You can find this info with `az account show`:

```
AZURE_TENANT_ID=<your-tenant-id-goes-here>
AZURE_SUBSCRIPTION_ID=<your-subscription-id-goes-here>
```

Next, create a `ServicePrincipal` for the Azure Service Operator to use:

```
az ad sp create-for-rbac -n "azure-service-operator" --role contributor \
    --scopes /subscriptions/$AZURE_SUBSCRIPTION_ID
```

### Step 2: Create a Secret with Service Principal Details

Based on the output of this command, you will then create a `Secret` containing the details of this Service Principal.  The `Secret` should look like this:

**`azure-operator-settings-secret.yaml`**
```
apiVersion: v1
kind: Secret
metadata:
  name: azureoperatorsettings
stringData:
  AZURE_TENANT_ID: <tenant ID>
  AZURE_SUBSCRIPTION_ID: <subscription ID>
  AZURE_CLIENT_ID: <app ID>
  AZURE_CLIENT_SECRET: <client secret / password>
  AZURE_CLOUD_ENV: AzurePublicCloud
```

This secret should be created in the `openshift-operators` namespace:

```
oc apply -f azure-operator-settings-secret.yaml -n openshift-operators
```

Once this is applied, you should be able to login to your OpenShift cluster as an Admin to finish the install.

### Step 3: Install the Azure Serivce Operator

1. From the **Administrator** view, go to the `OperatorHub` view.
2. Find the `Azure Service Operator` and deploy it to all namespaces.

The current version of the Azure Service Operator (0.37) has a Role Based Access Control bug.  You can [track the issue on GitHub](https://github.com/Azure/azure-service-operator/issues/1269).  Until this issue is resolved, the easiest fix is to grant the `azure-service-operator` service account that is in the `openshift-operators` namespace `cluster-admin` rights.

```
oc adm policy add-cluster-role-to-user cluster-admin -z azure-service-operator -n openshift-operators
```

Once the operator completes the install, you will now have a number of Azure services available to be deployed in projects.  They will appear as "Catalog" items that are "Operator Backed".

## Using the Azure Service Operator

You can add these Azure services through the OpenShift UI, but it's even better to simply include them as part of your GitOps configuration.  After all, definining a new MySQL database service is as simple as applying the following yaml to your cluster:

**`mysql-azure.yaml`**
```
apiVersion: azure.microsoft.com/v1alpha2
kind: MySQLServer
metadata:
  name: petclinic-mysql-aro
spec:  
  location: canadacentral
  resourceGroup: databases
  serverVersion: "8.0"
  sslEnforcement: Disabled
  createMode: Default # Possible values include: Default, Replica, PointInTimeRestore (not implemented), GeoRestore (not implemented)
  sku:
    name: B_Gen5_1 # tier + family + cores eg. - B_Gen4_1, GP_Gen5_4
    tier: Basic # possible values - 'Basic', 'GeneralPurpose', 'MemoryOptimized'
    family: Gen5 
    size: "51200"
    capacity: 1
---
apiVersion: azure.microsoft.com/v1alpha1
kind: MySQLDatabase
metadata:
  name: petclinic
spec:
  resourceGroup: databases
  server: petclinic-mysql-aro
---
apiVersion: azure.microsoft.com/v1alpha1
kind: MySQLFirewallRule
metadata:
  name: petclinic-mysql-aro-fw
spec:
  resourceGroup: databases
  server: petclinic-mysql-aro
  startIpAddress: 0.0.0.0
  endIpAddress: 0.0.0.0
```

```
oc apply -f mysql-azure.yaml -n <my project>
```

You can check the status of this request by listing the `mysqls` resources.  If everything is good, your `MySQLServer` instance should have a status of "Request sent to Azure".

It seems to take 2-3 minutes before you will notice the new database instance show up in the Activity Log in your Azure portal, and about 5min total before it  will be active and ready to accept connections.

## Guided Demo

If you would like to try a more in-depth demo that includes building an application conatitner and hypothetical "DEV" and "PROD" environments that use both MySQL container images and native Azure MySQL services, take a look at the [demo](demo.md) in this repository!

[DEMO: Mixing containers and native services in a an application enviornment](demo.md)