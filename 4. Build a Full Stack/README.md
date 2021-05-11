# 4. Build a Full Stack

explain the template

BuildConfig and DeploymentConfig pick up env vars from secret
build a NextJS app, populated a schema with tables and two

```
oc project demo-apps

oc new-app -f ./reverse-app-acme-stack-oc-template.yaml \
    -p NAME=reverse-app-pg -p SERVER_PORT=3000 -p CONN_NAME=acme-postgresqlinstance-conn
```

Output
```
--> Deploying template "demo-apps/reverse-app-with-pg" for "./reverse-app-acme-stack-oc-template.yaml" to project demo-apps

     Node.js
     ---------
     Simple NodeJS app interacting with PostgreSQL database

     The following service(s) have been created in your project: reverse-app-pg.


     * With parameters:
        * Name=reverse-app-pg
        * Namespace=openshift
        * Version of NodeJS Image=12
        * Memory Limit=512Mi
        * Git Repository URL=https://github.com/jeremycaine/reverse-app-with-pg.git
        * Git Reference=
        * Context Directory=
        * Application Hostname=
        * GitHub Webhook Secret=lDhbhe48RIkXosmT6auTCOC6MMqsnXT0tD0JpVlR # generated
        * Generic Webhook Secret=whloaeetiq1N8wbWErwNOBuA2pgQ008MpiWHMSo7 # generated
        * Custom NPM Mirror URL=
        * Server Listen Port=3000
        * Postgres Connection Secret=acme-postgresqlinstance-conn

--> Creating resources ...
    service "reverse-app-pg" created
    route.route.openshift.io "reverse-app-pg-route" created
    imagestream.image.openshift.io "reverse-app-pg" created
    buildconfig.build.openshift.io "reverse-app-pg" created
    deploymentconfig.apps.openshift.io "reverse-app-pg" created
--> Success
    Access your application via route 'reverse-app-pg-route-demo-apps.caine-roks-dev-a01ee4194ed985a1e32b1d96fd4ae346-0000.us-south.containers.appdomain.cloud'
    Build scheduled, use 'oc logs -f bc/reverse-app-pg' to track its progress.
    Run 'oc status' to view your app.
```

## Delete
The build uses the git repo name as the app selector e.g. `reverse-app-with-pg`

To delete all the components of the app deployment:
```
oc delete all -l app=reverse-app-with-pg
```