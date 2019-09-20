# catalog-service
1.1. Deploy Catalog Service to OpenShift
In this lab, you create the cart service. This service depends on a supporting service: the catalog service.
The solution code for the catalog service is provided. All you have to do is deploy the catalog service.
The catalog service is responsible for retrieving products from and adding products to the catalog. This functionality is exposed over a REST API. Product entities are stored in MongoDB.
1.	Git clone https://github.com/shaileshnamjoshi/catalog-service.git
2.	Go to your terminal and do cd catalog-service and then mvn clean package
3.	Log in to OpenShift, using your login credentials when prompted:
oc login --insecure-skip-tls-verify \ --server=<ocp cluster url>
4.	Create a new project:
export CATALOG_PRJ=<yourname>-coolstore-catalog  oc new-project $CATALOG_PRJ
5.	Deploy an instance of MongoDB on OpenShift using the coolstore-catalog-mongodb-persistent.yaml template in the etc directory:
oc process -f etc/coolstore-catalog-mongodb-persistent.yaml -p CATALOG_DB_USERNAME=mongo -p CATALOG_DB_PASSWORD=mongo -n $CATALOG_PRJ | oc create -f - -n $CATALOG_PRJ
o	The coolstore-catalog-mongodb-persistent.yaml file specifies the deployment directives for MongoDB, including the name of the base image, port numbers, and database name.
6.	Add the view role to the default service account:
oc adm policy add-role-to-group view system:serviceaccounts:$CATALOG_PRJ -n $CATALOG_PRJ
o	The Coolstore catalog microservice calls the Kubernetes API to retrieve the ConfigMap, which requires view access.
7.	Create the ConfigMap with the configuration for the Coolstore catalog microservice:
oc create configmap app-config --from-file=etc/app-config.yaml -n $CATALOG_PRJ
o	The etc folder of the application source project contains an example of a configuration file.
o	The configuration file defines the catalog.http.port parameter (port for the HTTP server) and the connection_string, db_name, username, and password MongoDB connection parameters, which are used by the Vert.x MongoDB client.
8.	Verify that the ConfigMap is deployed:
oc get configmap app-config -o yaml -n $CATALOG_PRJ
Sample Output
apiVersion: v1 data:   app-config.yaml: |-     catalog.http.port: 8080     connection_string: mongodb://catalog-mongodb:27017     db_name: catalogdb     username: mongo     password: mongo kind: ConfigMap metadata:   creationTimestamp: 2019-01-03T08:50:19Z   name: app-config   namespace: coolstore-catalog   resourceVersion: "19499"   selfLink: /api/v1/namespaces/coolstore-catalog/configmaps/app-config   uid: 2b3d7672-4a95-11e7-8788-507b9d27afbf
9.	Deploy the Coolstore catalog microservice on OpenShift:
mvn clean fabric8:deploy -Popenshift -Dfabric8.namespace=$CATALOG_PRJ
10.	While the app is deploying, log in to the OpenShift web console using your credentials when prompted.
11.	Select your project and then monitor the status of the deployment:

#The deployment may take 5 to 10 minutes depending on your network connection.
o	Alternatively, use the CLI:
oc get pods -n $CATALOG_PRJ
Sample Output
NAME                          READY     STATUS      RESTARTS   AGE catalog-mongodb-1-w132w       1/1       Running     0          1h catalog-service-1-p1wx1       1/1       Running     0          5m catalog-service-s2i-1-build   0/1       Completed   0          5m
	If you need to redeploy your application, use the following fabric8:build command:
mvn clean package fabric8:build -Popenshift -Dfabric8.namespace=$CATALOG_PRJ
12.	Wait for the deployment to complete, then proceed to the next section.
1.2. Test Catalog Microservice
In this section, you test the Coolstore catalog microservice using curl.
1.	From the command-line window, retrieve the URL of the Coolstore catalog microservice:
export CATALOG_URL=http://$(oc get route catalog-service -n $CATALOG_PRJ -o template --template='{{.spec.host}}')
2.	Invoke the readiness health check probe:
curl -X GET "$CATALOG_URL/health/readiness"
Sample Output
OK
3.	Invoke the liveness health check probe:
curl -X GET "$CATALOG_URL/health/liveness"
Sample Output
{"checks":[{"id":"health","status":"UP"}],"outcome":"UP"}
4.	Retrieve a list of products in the catalog:
curl -X GET "$CATALOG_URL/products"
Sample Output
[ {   "itemId" : "329299",   "name" : "Red Fedora",   "desc" : "Official Red Hat Fedora",   "price" : 34.99 }, {   "itemId" : "329299",   "name" : "Forge Laptop Sticker",   "desc" : "JBoss Community Forge Project Sticker",   "price" : 8.5 }, {   "itemId" : "165613",   "name" : "Solid Performance Polo",   "desc" : "Moisture-wicking, antimicrobial 100% polyester design wicks for life of garment. No-curl, rib-knit collar; special collar band maintains crisp fold; three-button placket with dyed-to-match buttons; hemmed sleeves; even bottom with side vents; Import. Embroidery. Red Pepper.",   "price" : 17.8 }, {   "itemId" : "165614",   "name" : "Ogio Caliber Polo",   "desc" : "Moisture-wicking 100% polyester. Rib-knit collar and cuffs; Ogio jacquard tape inside neck; bar-tacked three-button placket with Ogio dyed-to-match buttons; side vents; tagless; Ogio badge on left sleeve. Import. Embroidery. Black.",   "price" : 28.75 }, {   "itemId" : "165954",   "name" : "16 oz. Vortex Tumbler",   "desc" : "Double-wall insulated, BPA-free, acrylic cup. Push-on lid with thumb-slide closure; for hot and cold beverages. Holds 16 oz. Hand wash only. Imprint. Clear.",   "price" : 6.0 }, {   "itemId" : "444434",   "name" : "Pebble Smart Watch",   "desc" : "Smart glasses and smart watches are perhaps two of the most exciting developments in recent years. ",   "price" : 24.0 }, {   "itemId" : "444435",   "name" : "Oculus Rift",   "desc" : "The world of gaming has also undergone some very unique and compelling tech advances in recent years. Virtual reality, the concept of complete immersion into a digital universe through a special headset, has been the white whale of gaming and digital technology ever since Nintendo marketed its Virtual Boy gaming system in 1995.",   "price" : 106.0 }, {   "itemId" : "444436",   "name" : "Lytro Camera",   "desc" : "Consumers who want to up their photography game are looking at newfangled cameras like the Lytro Field camera, designed to take photos with infinite focus, so you can decide later exactly where you want the focus of each image to be.",   "price" : 44.3 } ]
5.	Retrieve an individual product from the catalog:
curl -X GET "$CATALOG_URL/product/444435"
Sample Output
{   "itemId" : "444435",   "name" : "Oculus Rift",   "desc" : "The world of gaming has also undergone some very unique and compelling tech advances in recent years. Virtual reality, the concept of complete immersion into a digital universe through a special headset, has been the white whale of gaming and digital technology ever since Nintendo marketed its Virtual Boy gaming system in 1995.",   "price" : 106.0 }


