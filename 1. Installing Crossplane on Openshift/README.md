# 1. Installing Crossplane on OpenShift
We will install Crossplane control plane system in an OpenShift cluster. This walkthrough will use Red Hat OpenShift on IBM Cloud, but any OpenShift cluster will be equivalent. Additional we will install the IBM Cloud Crossplane provider so that we can provision and lifecycle managed IBM Cloud services using Crossplane.

This walkthrough was tested with Crossplane v1.2 on an OpenShift 4.6 cluster.

The latest installation steps for Crossplane are [here](https://crossplane.io/docs/v1.2/getting-started/install-configure.html) and the IBM Cloud provider [here](https://github.com/crossplane-contrib/provider-ibm-cloud).

## Pre-requisites

1. Provision a Crossplane namespace project in my OpenShift cluster
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
kubectl apply -f providerconfig-ibm-cloud.yaml
```

This will create all the necessary cluster objects. The CRDs of Crossplane are intended to be usable cluster-wide.
```
# check the resources are all running ok
oc get all
```

