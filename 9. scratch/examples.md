# Examples of Provisioning IBM Cloud resources via Crossplane

## Example deploy
Create an instance of PostgreSQL on IBM Cloud
```
apiVersion: resourcecontrollerv2.ibm-cloud.crossplane.io/v1alpha1
kind: ResourceInstance
metadata:
  name: sample-postgresql
spec:
  forProvider:
    name: mypostgres
    target: us-south
    resourceGroupName: default
    serviceName: databases-for-postgresql
    resourcePlanName: standard
    tags:
      - dev
    parameters:
      members_disk_allocation_mb: 32768
  providerConfigRef:
    name: ibm-cloud
```
```
kubectl apply -f caine-postgres.yaml
```

## Import Resources
Get information about already provisioned IBM Cloud resources
e.g. an existing cloud database
```
ibmcloud resource service-instance sample-postgresql
```
Returns
```
Name:                  sample-postgresql   
ID:                    crn:v1:bluemix:public:databases-for-postgresql:us-south:a/b71ac2564ef0b98f1032d189795994dc:664c5fea-7d53-43e4-bd45-d3735afde725::   
GUID:                  664c5fea-7d53-43e4-bd45-d3735afde725   
Location:              us-south   
Service Name:          databases-for-postgresql   
Service Plan Name:     standard   
Resource Group Name:   default   
State:                 active   
Type:                  service_instance   
Sub Type:              Public   
Created at:            2021-01-15T08:14:59Z   
Created by:            jeremycaine@ibm.com   
Updated at:            2021-01-15T08:25:28Z   
Last Operation:                        
                       Status    create succeeded      
                       Message   Provisioning postgresql with version 12 (100%)  
```

Then get env vars:
```
NAME=mypostgres

INFO="$(ibmcloud resource service-instance "${NAME}")"
ID=$(echo "$INFO" | grep ^ID | awk '{print $2}')
TARGET=$(echo "$INFO" | grep ^Location | awk '{print $2}')
SERVICE_NAME=$(echo "$INFO" | grep "^Service Name" | awk '{print $3}')
RG_NAME=$(echo "$INFO" | grep "^Resource Group Name" | awk '{print $4}')
RP_NAME=$(echo "$INFO" | grep "^Service Plan Name" | awk '{print $4}')
META_NAME=$(echo "$NAME" | awk '{print tolower($0)}' | tr " " - | tr "." -)
```


```
cat <<EOF | kubectl apply -f -
apiVersion: resourcecontrollerv2.ibm-cloud.crossplane.io/v1alpha1
kind: ResourceInstance
metadata:
  name: $META_NAME
  annotations:
    crossplane.io/external-name: "$ID"
spec:
  forProvider:
    name: $NAME
    target: $TARGET
    resourceGroupName: $RG_NAME
    serviceName: $SERVICE_NAME
    resourcePlanName: $RP_NAME
  providerConfigRef:
    name: ibm-cloud
EOF
```

## IBM Cloud Database commands

Install IBM Cloud Database plugin for CLI
```
ibmcloud plugin install cloud-databases
```

Set admin password on a database
```
ibmcloud cdb user-password sample-postgresql admin <newpassword>
```

Install `pgadmin` utility from [here](https://www.pgadmin.org/download/)
```
# drag pgAdmin 4 into Applications on macos
# launches in browser
http://127.0.0.1:53652/browser/
```

Install `psql` on Mac with 
```
brew install libpq
brew link --force libpq
```

Connect to the database via command line
```
ibmcloud cdb deployment-connections sample-postgresql --start
```