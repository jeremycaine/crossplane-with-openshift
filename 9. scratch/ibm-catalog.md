## Finding Services Names
```
ic catalog service-marketplace | grep databases-for-postgresql | awk '{print $2}'
ic catalog service-marketplace | grep satellite | awk '{print $2}'
ic catalog service-marketplace | grep openshift | awk '{print $2}'
ic catalog service-marketplace  | awk '{print $2}'

curl --request GET --url 'https://globalcatalog.cloud.ibm.com/api/v1/databases-for-postgresql?complete=true&depth=100' --header 'accept: application/json' | jq '.children[].children[] | .id, .metadata.service.parameters'

curl --request GET --url 'https://containers.cloud.ibm.com/global/api/v2/satellite?complete=true&depth=100' --header 'accept: application/json' | jq '.children[].children[] | .id, .metadata.service.parameters'

curl --request GET --url 'https://globalcatalog.cloud.ibm.com/api/v1/databases-for-redis?complete=true&depth=100' --header 'accept: application/json'
```

https://github.com/IBM/platform-services-go-sdk

## Working with your new managed resource

ibmcloud cdb user-password mypostgres-via-crossplane admin <newpassword>
ibmcloud cdb cxn pmypostgres-via-crossplane -s