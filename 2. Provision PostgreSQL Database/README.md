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

![image of IBM Console](/Users/jeremycaine/code/ibm.github.com/crossplane-with-openshift/assets/pg-provision.gif)
*and on the IBM Cloud Console, you can see the provisioning in progress*

kubectl get resourceinstance.resourcecontrollerv2.ibmcloud.crossplane.io/mypostgres-via-crossplane

# Delete the instance of Pg database service
kubectl delete resourceinstance.resourcecontrollerv2.ibmcloud.crossplane.io/postgres-via-crossplane
kubectl delete resourceinstance postgres-via-crossplane

# admin
ibmcloud cdb user-password example-pg admin <newpassword>
ibmcloud cdb cxn postgres-via-crossplane -s

# composite
kubectl apply -f composition.yaml
kubectl get resourceinstance caine-xp-composition
?? kubectl describe secret postgres-via-crossplane-secret -n crossplane-system
```

kubectl describe secret postgres-via-crossplane-secret -n crossplane-system



