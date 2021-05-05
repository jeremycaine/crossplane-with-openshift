# 1. Installing Crossplane on OpenShift
We will install Crossplane control plane system in an OpenShift cluster. This walkthrough will use Red Hat OpenShift on IBM Cloud, but any OpenShift cluster will be equivalent. Additional we will install the IBM Cloud Crossplane provider so that we can provision and lifecycle managed IBM Cloud services using Crossplane.

This walkthrough was tested with Crossplane v1.2 on an OpenShift 4.6 cluster.

The latest installation steps for Crossplane are [here](https://crossplane.io/docs/v1.2/getting-started/install-configure.html) and the IBM Cloud provider [here](https://github.com/crossplane-contrib/provider-ibm-cloud).

## Pre-requisites

1. provision a Crossplane namespace project in my OpenShift cluster
```
ic login --sso
oc login ...

oc new-project crossplane-system
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
```
2. install Crossplane
```
# in OpenShift this creates the Service Account 'crossplane' during the chart install
# change version number
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane --set securityContextCrossplane.runAsUser=null --set securityContextCrossplane.runAsGroup=null --version 1.1.0 --set alpha.oam.enabled=true

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane --set securityContextCrossplane.runAsUser=null --set securityContextCrossplane.runAsGroup=null --set securityContextRBACManager.runAsUser=null --set securityContextRBACManager.runAsGroup=null --version 1.2.0 --set alpha.oam.enabled=true

curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
sudo mv kubectl-crossplane /usr/local/bin
kubectl crossplane --help
```

3. install providers e.g. IBM Cloud
```
kubectl apply -f openshift-config.yaml
kubectl apply -f provider-ibm-cloud.yaml
kubectl crossplane install provider crossplane/provider-ibm-cloud:alpha

# Using an IBM Cloud account with permissions to manage IBM Cloud Services:
if [[ -z "${IBMCLOUD_API_KEY}" ]]; then
  echo "*** Generating new APIKey"
  IBMCLOUD_API_KEY=$(ibmcloud iam api-key-create provider-ibm-cloud-key -d "Key for Crossplane Provider IBM Cloud" | grep "API Key" | awk '{ print $3 }')
fi

kubectl create secret generic provider-ibm-cloud-secret --from-literal=credentials=${IBMCLOUD_API_KEY} -n crossplane-system

# We will create the following ProviderConfig object to configure credentials for IBM Cloud Provider:
cat <<EOF | kubectl apply -f -
apiVersion: ibmcloud.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: ibm-cloud
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: provider-ibm-cloud-secret
      key: credentials
  region: us-south
EOF

oc get all
```

## Create Managed Resource direct to the Provider
Create a resource of Kind supported by the IBM Cloud provider
```
apiVersion: resourcecontrollerv2.ibmcloud.crossplane.io/v1alpha1
kind: ResourceInstance
metadata:
  name: postgres-via-crossplane
spec:
  forProvider:
    name: postgres-via-crossplane
    target: us-south
    resourceGroupName: default
    serviceName: databases-for-postgresql
    resourcePlanName: standard
    userPassword:
    
    tags:
      - dev
    parameters:
      members_disk_allocation_mb: 32768

  providerConfigRef:
    name: ibm-cloud
```
Create an instance of Pg datbase service ``

```
# Create an instance of Pg database service
kubectl apply -f resource-instance-postgres.yaml
kubectl get resourceinstance postgres-via-crossplane
kubectl describe secret postgres-via-crossplane-secret -n crossplane-system

kubectl get resourceinstance.resourcecontrollerv2.ibmcloud.crossplane.io/postgres-via-crossplane

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