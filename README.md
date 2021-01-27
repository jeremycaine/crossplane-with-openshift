# Crossplane
Exploration of Crossplane open source project and the Crossplane provider for IBM Cloud

## Project Setup
```
oc new-project crossplane-system
# which creates kubernetes namespace crossplane-system, so don't need
# kubectl create namespace crossplane-system
```

## OpenShift RBAC for Crossplane deployment
Allow the IAM user to be able to instatiate the crossplane deployment
```
# workaround
# -z (service account) crossplane must have been created as part of helm install
oc adm policy add-scc-to-user privileged -z crossplane -n crossplane-system
oc adm policy remove-scc-from-user privileged -z crossplane -n crossplane-system
```
or re-apply Provider for Crossplane, but add ControllerConfig
```
apiVersion: pkg.crossplane.io/v1alpha1
kind: ControllerConfig
metadata:
  name: openshift-config
  namespace: crossplane-system
spec:
  securityContext: {}
  podSecurityContext: {}
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Provider
metadata:
  name: provider-ibm-cloud
spec:
  package: crossplane/provider-ibm-cloud:alpha
  controllerConfigRef:
    name: openshift-config
```

## Install Crossplane
Create an OpenShift cluster on IBM Cloud
```
ic login -sso
oc login ...
```

Install Crossplane taking instructions from [here](https://crossplane.io/docs/v1.0/getting-started/install-configure.html)
```
# if needed
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

# install crossplane
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
kubectl get all -n crossplane-system
```

## Install Crossplane CLI
Check Crossplane status
```
helm list -n crossplane-system
kubectl get all -n crossplane-system
```

Install the Crossplane CLI
```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/release-1.0/install.sh | sh
sudo mv kubectl-crossplane /usr/local/bin
kubectl crossplane --help
```

## Install IBM Cloud Provider
Install [IBM Cloud Provider](https://github.com/crossplane-contrib/provider-ibm-cloud)
```
kubectl crossplane install provider crossplane/provider-ibm-cloud:alpha
```

Generate IBM Cloud API Key
```
if [[ -z "${IBMCLOUD_API_KEY}" ]]; then
  echo "*** Generating new APIKey"
  IBMCLOUD_API_KEY=$(ibmcloud iam api-key-create provider-ibm-cloud-key -d "Key for Crossplane Provider IBM Cloud" | grep "API Key" | awk '{ print $3 }')
fi

# *** Generating new APIKey
# Please preserve the API key! It cannot be retrieved after it's created.
```
Create a Provider secret
```
kubectl create secret generic provider-ibm-cloud-secret --from-literal=credentials=${IBMCLOUD_API_KEY} -n crossplane-system
```

Create a Provider Config object to configure credentials for the IBM Cloud Provider
```
kubectl apply -f providerconfig-ibm-cloud.yaml
kubectl get all -n crossplane-system
```

### OpenShift RBAC for IBM Cloud Provider deployment
Allow the IAM user to be able to instatiate the crossplane deployment
```
# -z (service account) crossplane must have been created as part of helm install
oc adm policy add-scc-to-user privileged -z crossplane-provider-ibm-cloud-12f6b13091ae -n crossplane-system
```

## References
[ISSUE 19](https://github.com/crossplane-contrib/provider-ibm-cloud/issues/19) - CLOSED