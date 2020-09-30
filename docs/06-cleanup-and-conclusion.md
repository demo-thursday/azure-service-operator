# Step 6: Cleanup and Conclusion

[Back to step 5: Deploy Pet Clinic connected to a Managed Azure MySQL Instance](05-deploy-prod.md)

That was pretty easy, right?

Now all that's left to do is clean up our resources!

## Deleting the Azure MySQL Database

Since the Azure MySQL instance we are using was created by the Azure Service Operator, we will also let the Azure Service Operator delete our database.

Although it sounds almost too simple, all you need to do to delete you Azure MySQL instance is to delete the associated `MySQLServer` custom resource.

You could delete just the database with the following command:

```
oc delete mysqlserver prod-mysql-aro -n petclinic-prod
```

Or... clean up everything all at once by deleting our demo Projects/namespaces.

```
oc delete project petclinic-dev
oc delete project petclinic-prod
oc delete project cicd
```

When the Azure Service Operator is notified that a resource is deleted, it will take care of deleting it from your Azure subscription as well.  Once you delete your projects in OpenShift, take a look at your Azure Portal to see if your database and Resource Group are gone.  It may take a moment for the Azure Portal to reflect these changes.

## Conclusion

[Azure Red Hat OpenShift](https://azure.microsoft.com/en-us/services/openshift/) brings the full Enterprise Kubernetes power of [OpenShift Container Platform](https://www.openshift.com) to Microsoft Azure as a fully managed service that is jointly engineered and supported by Microsoft and Red Hat.

The [Azure Service Operator](https://github.com/Azure/azure-service-operator) brings tight integration between OpenShift and Azure, allowing Devlopers and Operators to provision both containers and native Azure services in a Kubernetes-native way that feels perfectly natural, allowing for smart decisions to be made when it comes to the supporting infrastructure of your applications.

The use cases will only continue to increase as more services get added to the Azure Service Operaator.  You can [track the official list here](https://github.com/Azure/azure-service-operator#supported-azure-services).

