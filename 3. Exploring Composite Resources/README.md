# 3. Exploring Composite Resources
As platform engineers Crossplane allows us to define custom 'stacks' which are composition of platform services. Those compositions can be defined to only expose the underlying provider specific parameters that the platform engineering teams want to expose.

A DevOps team then provisions an instance of the 'stack' as their base environment accessing the instances of platform services (and other object specific to them e.g. secrets).

There are different ways to compose and offer these resources to teams (see [here](https://crossplane.io/docs/v1.2/concepts/composition.html))

## Create a User-defined Composition of Platform Services
Let's look at a simple composition that could represent a 'stack' that an enteprise platform engineering team would assemble.

1. User Namespace
The consuming DevOps team will have their namespaces where they will build and deploy their application that will use the composite platform services.

This namespace is referenced in their composite resource definition so that secrets can be created there and visible to the application.

```
oc new-project demo-apps
```

2. Definitions
A `CompositeResourceDefinition` kind object is created that sets out the name of the composite object that will be created, the parameters that can be set (and values), and parameters returned from connection secrets.

It also references the `Composition` resource that wraps the underlying provider service.

```
kubectl apply -f acme-postgres-definition.yaml
```

3. Composition
This resource "assembles" the composite platform service. It specifies the provider and services from that provider. It describes how parameters set by the composite specification, are translated or mapped into the provider specific variables.

```
kubectl apply -f acme-postgres-composition.yaml
```

4. Composite
When this resource object is created or updated it triggers the provisioning and update of the underlying services. It specifies the input parameters and the connection secret that will be created in the user's namespace.
```
kubectl apply -f acme-postgres-composite.yaml
```
At this point you can see the provisioning process begin in the cluster and on the IBM Cloud console.
```
kubectl get resourceinstance
NAME                            STATUS   STATE          CLASS   AGE
acme-postgres-composite-6pn7d            provisioning           36s
```

In the user namespace you can see the namespaced secret of type `connection.crossplane.io/v1alpha1`
```
oc project demo-apps
oc get secrets
```

When the database has completed provisioning, the secret's data is populated
```
oc describe secret acme-postgresqlinstance-conn

Name:         acme-postgresqlinstance-conn
Namespace:    demo-apps
Labels:       <none>
Annotations:  <none>

Type:  connection.crossplane.io/v1alpha1

Data
====
host:      83 bytes
password:  64 bytes
port:      5 bytes
username:  46 bytes
endpoint:  118 bytes
```

5. Deletion
In order to delete the composite service you need to delete the composite resource object which will in turn delete the underlying resource instances as specified in the composition - and this will trigger the delete action of the provider's services.
```
oc delete project demo-apps
kubectl delete ACMECompositePostgreSQLInstance/acme-postgres-composite
kubectl delete Composition/acme-postgres-composition-ref
kubectl delete xrd acmecompositepostgresqlinstances.acme.org
```


