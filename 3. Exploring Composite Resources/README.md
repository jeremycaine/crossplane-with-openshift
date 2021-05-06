# 3. Exploring Composite Resources
As platform engineers Crossplane allows us to define custom 'stacks' which are composition of platform services. Those compositions can be defined to only expose the underlying provider specific parameters that the platform engineering teams want to expose.

A DevOps team then provisions an instance of the 'stack' as their base environment accessing the instances of platform services (and other object specific to them e.g. secrets).

There are different ways to compose and offer these resources to teams (see [here](https://crossplane.io/docs/v1.2/concepts/composition.html))

## Create a User-defined Composition of Platform Services
Let's look at a simple composition that could represent a 'stack' that an enteprise platform engineering team would assemble.

1. Definitions

  claimNames:
    kind: ACMEPostgreSQLInstance
    plural: acmepostgresqlinstances


oc new-project demo-apps

Type 1
kubectl apply -f acme-postgres-definition.yaml
kubectl apply -f acme-postgres-composition.yaml
kubectl apply -f acme-postgres-composite.yaml

Tyep 2
kubectl apply -f acme-postgres-definition.yaml
kubectl apply -f acme-postgres-claim.yaml


kubectl apply -f definition.yaml
compositeresourcedefinition.apiextensions.crossplane.io/compositepostgresqlinstances.example.org created

kubectl apply -f composite.yaml
compositepostgresqlinstance.example.org/example-ibm-cloud created

kubectl apply -f composition.yaml
composition.apiextensions.crossplane.io/example-ibm-cloud created

kubectl apply -f composition.yaml
composition.apiextensions.crossplane.io/example-ibm-cloud configured

kubectl apply -f claim.yaml
postgresqlinstance.example.org/example created

kubectl get resourceinstance
NAME                      STATUS   STATE    CLASS   AGE
example-ibm-cloud-sqdm9            active           16h
example-ljp7z-25292                active           16h

kubectl delete resourceinstance/example-ljp7z-25292
kubectl delete resourceinstance/example-ibm-cloud-sqdm9



