# 1. Installing Crossplane on OpenShift
We will install Crossplane control plane system in an OpenShift cluster. This walkthrough will use Red Hat OpenShift on IBM Cloud, but any OpenShift cluster will be equivalent. Additional we will install the IBM Cloud Crossplane provider so that we can provision and lifecycle managed IBM Cloud services using Crossplane.

This walkthrough was tested with Crossplane v1.2 on an OpenShift 4.6 cluster.

The latest installation steps for Crossplane are [here](https://crossplane.io/docs/v1.2/getting-started/install-configure.html) and the IBM Cloud provider [here](https://github.com/crossplane-contrib/provider-ibm-cloud).

## Create a Crossplane project and namespace in the OpenShift cluster
```
# log into your cloud infrastructure and your OpenShift cluster
ic login ...
oc login ...

# Create a project and namespace into where the Crossplane system will be installed
oc new-project crossplane-system
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
```
## Install Crossplane
Due to the nature of OpenShift's enterprise strength security model when we install Crossplane we need to set various security contexts. By setting to `null` we pass through the user credentials authority of the logged in OpenShift user through to the Crossplane components.
```
# in OpenShift this creates the Service Account 'crossplane' during the chart install
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane --set securityContextCrossplane.runAsUser=null --set securityContextCrossplane.runAsGroup=null --set securityContextRBACManager.runAsUser=null --set securityContextRBACManager.runAsGroup=null --version 1.2.0 --set alpha.oam.enabled=true

# install the Crossplane CLI
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
sudo mv kubectl-crossplane /usr/local/bin

# checl the Crossplane CLI installed and working
kubectl crossplane --help
```

## Install IBM Cloud Providers
The provider is how Crossplane interacts with the API of the cloud service provider and manage the resources through its control plane. Crossplane CLI does not yet support `ControllerConfig` which we need so we can set an empty security context to ensure OpenShift has the correct `runAsUser` and `runAsGroup` privileges.
```
# OpenShift ControllerConfig
kubectl apply -f openshift-config.yaml

# Create the provder and install into the Crossplane system
kubectl apply -f provider-ibm-cloud.yaml
kubectl crossplane install provider crossplane/provider-ibm-cloud:alpha

# We need to create a key to use an IBM Cloud account with permissions to manage IBM Cloud Services
if [[ -z "${IBMCLOUD_API_KEY}" ]]; then
  echo "*** Generating new APIKey"
  IBMCLOUD_API_KEY=$(ibmcloud iam api-key-create provider-ibm-cloud-key -d "Key for Crossplane Provider IBM Cloud" | grep "API Key" | awk '{ print $3 }')
fi

# put the IBM Cloud credentials in a secret for Crossplane
kubectl create secret generic provider-ibm-cloud-secret --from-literal=credentials=${IBMCLOUD_API_KEY} -n crossplane-system

# We will create the following ProviderConfig object to configure credentials for IBM Cloud Provider
kubectl apply -f providerconfig-ibm-cloud.yaml
```

This will create all the necessary cluster objects. The CRDs of Crossplane are intended to be usable cluster-wide.

## Confirm Installation
```
# check the resources are all running ok
oc get all

# with results like this
NAME                                                     READY   STATUS    RESTARTS   AGE
pod/crossplane-9798b7c4c-75jmj                           1/1     Running   0          4m15s
pod/crossplane-rbac-manager-5ffd879c55-295dk             1/1     Running   0          4m14s
pod/oam-kubernetes-runtime-crossplane-59bd79c76c-vxtzz   1/1     Running   0          4m15s
pod/provider-ibm-cloud-e0a75509c392-6dbf84c6d-w58jf      1/1     Running   0          3m57s

NAME                                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/crossplane                          1/1     1            1           4m16s
deployment.apps/crossplane-rbac-manager             1/1     1            1           4m15s
deployment.apps/oam-kubernetes-runtime-crossplane   1/1     1            1           4m16s
deployment.apps/provider-ibm-cloud-e0a75509c392     1/1     1            1           3m58s

NAME                                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/crossplane-9798b7c4c                           1         1         1       4m16s
replicaset.apps/crossplane-rbac-manager-5ffd879c55             1         1         1       4m15s
replicaset.apps/oam-kubernetes-runtime-crossplane-59bd79c76c   1         1         1       4m16s
replicaset.apps/provider-ibm-cloud-e0a75509c392-6dbf84c6d      1         1         1       3m58s
```

