# Various install steps
Have a cluster ready

#kind create cluster --name=crossplane1

## full install
```
kubectl create namespace crossplane-system

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
helm list -n crossplane-system
kubectl get all -n crossplane-system

curl -sL https://raw.githubusercontent.com/crossplane/crossplane/release-1.0/install.sh | sh
sudo mv kubectl-crossplane /usr/local/bin
kubectl crossplane --help

kubectl apply -f provider-ibm-cloud.yaml
kubectl get provider
kubectl get crds | grep ibm-cloud

if [[ -z "${IBMCLOUD_API_KEY}" ]]; then
  echo "*** Generating new APIKey"
  IBMCLOUD_API_KEY=$(ibmcloud iam api-key-create provider-ibm-cloud-key -d "Key for Crossplane Provider IBM Cloud" | grep "API Key" | awk '{ print $3 }')
fi

kubectl create secret generic provider-ibm-cloud-secret --from-literal=credentials=${IBMCLOUD_API_KEY} -n crossplane-system
kubectl get all -n crossplane-system

kubectl apply -f providerconfig-ibm-cloud.yaml
kubectl get all -n crossplane-system
```

## fix privilege
```
oc adm policy add-scc-to-user privileged -z crossplane -n crossplane-system
> securitycontextconstraints.security.openshift.io/privileged added to: ["system:serviceaccount:crossplane-system:crossplane"]

# then back to ...
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane

```

## basic steps
With helm, crossplane kubectl and API_KEY already done
```
kubectl create namespace crossplane-system
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
kubectl apply -f provider-ibm-cloud.yaml

kubectl get provider # does not return anything

kubectl get crds | grep ibm-cloud

if [[ -z "${IBMCLOUD_API_KEY}" ]]; then
  echo "*** Generating new APIKey"
  IBMCLOUD_API_KEY=$(ibmcloud iam api-key-create provider-ibm-cloud-key -d "Key for Crossplane Provider IBM Cloud" | grep "API Key" | awk '{ print $3 }')
fi

kubectl create secret generic provider-ibm-cloud-secret --from-literal=credentials=${IBMCLOUD_API_KEY} -n crossplane-system

kubectl apply -f providerconfig-ibm-cloud.yaml

kubectl get all -n crossplane-system
```

