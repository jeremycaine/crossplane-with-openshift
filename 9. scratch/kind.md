# kind base install of Crossplane v1.0
```
$ kind create cluster --name=crossplanev1
Creating cluster "crossplanev1" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-crossplanev1"
You can now use your cluster with:
kubectl cluster-info --context kind-crossplanev1
Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

$ kubectl create namespace crossplane-system

$ helm repo add crossplane-stable https://charts.crossplane.io/stable

$ helm repo update

$ helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
namespace/crossplane-system created

NAME: crossplane
LAST DEPLOYED: Thu Jan 14 20:28:36 2021
NAMESPACE: crossplane-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Release: crossplane
Chart Name: crossplane
Chart Description: Crossplane is an open source Kubernetes add-on that extends any cluster with the ability to provision and manage cloud infrastructure, services, and applications using kubectl, GitOps, or any tool that works with the Kubernetes API.
Chart Version: 1.0.0
Chart Application Version: 1.0.0
Kube Version: v1.19.1

$ kubens crossplane-system
Context "kind-crossplanev1" modified.
Active namespace is "crossplane-system".

$ kubectl get all -n crossplane-system
NAME            NAMESPACE               REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
crossplane      crossplane-system       1               2021-01-14 20:28:36.31492 -0500 EST     deployed        crossplane-1.0.0        1.0.0      
NAME                                           READY   STATUS    RESTARTS   AGE
pod/crossplane-558bd75f5-q7zj9                 1/1     Running   0          100s
pod/crossplane-rbac-manager-65cd75b79d-f8dhv   1/1     Running   0          100s
NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/crossplane                1/1     1            1           100s
deployment.apps/crossplane-rbac-manager   1/1     1            1           100s
NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/crossplane-558bd75f5                 1         1         1       100s
replicaset.apps/crossplane-rbac-manager-65cd75b79d   1         1         1       100s

$ kubectl crossplane install provider crossplane/provider-ibm-cloud:alpha
provider.pkg.crossplane.io/crossplane-provider-ibm-cloud created

$ k get provider
NAME                            INSTALLED   HEALTHY   PACKAGE                               AGE
crossplane-provider-ibm-cloud   True        Unknown   crossplane/provider-ibm-cloud:alpha   9s
$ k get provider
NAME                            INSTALLED   HEALTHY   PACKAGE                               AGE
crossplane-provider-ibm-cloud   True        True      crossplane/provider-ibm-cloud:alpha   54s

$ kubectl get crds | grep ibm-cloud
providerconfigs.ibm-cloud.crossplane.io                          2021-01-15T01:39:35Z
providerconfigusages.ibm-cloud.crossplane.io                     2021-01-15T01:39:35Z
resourceinstances.resourcecontrollerv2.ibm-cloud.crossplane.io   2021-01-15T01:39:35Z
resourcekeys.resourcecontrollerv2.ibm-cloud.crossplane.io        2021-01-15T01:39:35Z

$ kubectl create secret generic provider-ibm-cloud-secret --from-literal=credentials=${IBMCLOUD_API_KEY} -n crossplane-system
secret/provider-ibm-cloud-secret created

$ cat <<EOF | kubectl apply -f -
apiVersion: ibm-cloud.crossplane.io/v1beta1
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

providerconfig.ibm-cloud.crossplane.io/ibm-cloud created

kubectl get all -n crossplane-system
```