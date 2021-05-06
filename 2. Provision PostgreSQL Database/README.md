# 2. Provision PostgreSQL Database
With Crossplane installed in our OpenShift cluster we can now use it to provision and lifecycle manage IBM Cloud services through the provider.

The power of Crossplane is its ability to define an environment of composites of a provider's platform services for a DevOps team with enterprise guardrails around their consumption.

To start with let's see how to provision a basic cloud service resource and manage it through Crossplane.

Examples on how to create different IBM Cloud resources with the provider are [here](https://github.com/crossplane-contrib/provider-ibm-cloud/tree/master/examples).

## Create Managed Resource as direct provision of a service from the Provider
Create a resource of Kind supported by the IBM Cloud provider.

This example creates a ResourceInstance (IBM Cloud provider CRD) for a [IBM Cloud Databases for PostgreSQL](https://cloud.ibm.com/docs/databases-for-postgresql?topic=databases-for-postgresql-getting-started) service. The provider is 

```
apiVersion: resourcecontrollerv2.ibmcloud.crossplane.io/v1alpha1
kind: ResourceInstance
metadata:
  name: mypostgres-via-crossplane
spec:
  forProvider:
    name: mypostgres
    target: us-south
    resourceGroupName: default
    serviceName: databases-for-postgresql
    resourcePlanName: standard
    tags:
      - crossplane
      - caine
    parameters:
      members_disk_allocation_mb: 32768
  providerConfigRef:
    name: ibm-cloud
```

Create an instance of Pg database service giving the name `mypostgres-via-crossplane`. 

Its identifier key in the IBM Cloud catalogue is `databases-for-postgresql`. In the spec you can set provider specific parametes associated with this service.

```
# Create an instance of Pg database service
kubectl apply -f resource-instance-mypostgres.yaml

# check it has been created
kubectl get resourceinstance mypostgres-via-crossplane

# shows the provisioning in progress
NAME                        STATUS   STATE          CLASS   AGE
mypostgres-via-crossplane            provisioning           111s
```

![image of IBM Console](../assets/pg-provision.gif)
*and on the IBM Cloud Console, you can see the provisioning in progress*

and then provisioning is complete
```
kubectl get resourceinstance mypostgres-via-crossplane

NAME                        STATUS   STATE    CLASS   AGE
mypostgres-via-crossplane            active           42m

```

## Test the Control Loop
Now you can see the power of the control loop that Crossplane manages.

We have defined a resource - a PostgreSQL database - that the control plane will ensure is always in place. 

This is a database as a service on IBM Cloud. 

In normal circumstances if we delete this service then all of its data is lost - deletion of the service includes the storage allocation, database data within it, and the backups associated with it. Production enterprise solutions should implement external backup/restore management.

So, within the lifecycle management scope of IBM Cloud Databases, let's see how it interacts with Crossplane's control plane. 

1. See the Managed Resource as active under the Crossplane control plane.
```
kubectl get resourceinstance mypostgres-via-crossplane

NAME                        STATUS   STATE    CLASS   AGE
mypostgres-via-crossplane            active           42m
```

2. Go to the IBM Cloud console and DELETE the database `mypostgres-via-crossplane`
What you find is that you cannot. You confirm you want to delete, and status briefly changes to "deleting" but then an error comes back.

*An error occurred during an attempt to complete the operation. Try fixing the issue or try the operation again later.

Description:
The instance state cannot be saved in the delete operation.*

Crossplane is managing the state of the resource and enforcing it - not allowing an external force to delete it. Pretty neat.

## Take the Database out of Loop
In order to delete the instance of PostgreSQL database service you have to delete the managed resource that Crossplane is maintaining through the control loop.
```
kubectl delete resourceinstance mypostgres-via-crossplane

resourceinstance.resourcecontrollerv2.ibmcloud.crossplane.io "mypostgres-via-crossplane" deleted
```
Now in IBM Cloud console you will see the database service deleted from the resource list.





