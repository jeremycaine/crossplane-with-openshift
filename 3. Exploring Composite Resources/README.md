## Create Composite Resource for provisining via Crossplane
Create a 'stack' for OpenShift.

3. Create a composition
Using ...
4. kubectl describe compositepostgresqlinstance.example.org
5. create Composite Resource Claim - a claim that use compositeSelector triggers a dynamic provision
6. kubectl describe mysqlinstanceclaim.example.org example


1. Define the composite resource of kind: CompositeResourceDefinition (XRD)
kubectl apply -f postgres-xrd.yaml
> compositeresourcedefinition.apiextensions.crossplane.io/compositepostgresqlinstances.example.org created
kubectl describe xrd compositepostgresqlinstances.example.org

2. Create a Composition
kubectl apply -f postgres-composition.yaml
> composition.apiextensions.crossplane.io/example-ibm-cloud created

3. Platform builder creates a Composite so that claims can be made by the Dev platform consumer
kubectl apply -f postgres-composite.yaml
> compositepostgresqlinstance.example.org/example-ibm-cloud created
kubectl describe compositepostgresqlinstance.example.org

4. Create a omposite Resource Claim - a claim that use compositeSelector triggers a dynamic provision
kubectl apply -f postgres-claim.yaml
> postgresqlinstance.example.org/example created

kubectl describe postgresqlinstance.example.org

No resources found in crossplane-system namespace

kubectl get resourceinstance 
kubectl describe secret db-conn -n crossplane-system

```

## Finding Services Names
```
ic catalog service-marketplace | grep databases-for-postgresql | awk '{print $2}'
ic catalog service-marketplace | grep satellite | awk '{print $2}'
ic catalog service-marketplace | grep openshift | awk '{print $2}'
ic catalog service-marketplace  | awk '{print $2}'

curl --request GET --url 'https://globalcatalog.cloud.ibm.com/api/v1/databases-for-postgresql?complete=true&depth=100' --header 'accept: application/json' | jq '.children[].children[] | .id, .metadata.service.parameters'

curl --request GET --url 'https://containers.cloud.ibm.com/global/api/v2/satellite?complete=true&depth=100' --header 'accept: application/json' | jq '.children[].children[] | .id, .metadata.service.parameters'

curl --request GET --url 'https://globalcatalog.cloud.ibm.com/api/v1/databases-for-redis?complete=true&depth=100' --header 'accept: application/json'

https://github.com/IBM/platform-services-go-sdk