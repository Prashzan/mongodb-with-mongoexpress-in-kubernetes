### Deploying mongodb with mongoexpress in kubernetes

#### It demonstrates really well a typical simple setup of web appliation and its database. You can apply this to any similar setup you have

#### First we will create a mongodb deployment. Inorder to talk to the pod (deployment), we gonna need a service. We gonna create a internal service which basically means that no external requests are allowed to the mongodb pod, only components inside the same cluster can talk to it.

#### Then we gonna create mongo-express deployment

**1.** we gonna need database_url of mongodb so that mongo-express can connect to it. <br>
**2.** Second this we need is credentials i.e. username and password of the database sot that it can authenticate

#### The way we can pass this information like database_url, database username and database password to mongo-express deployment is through its deployment configuration file through environment variables. This is how application is configured.

#### a. We gonna create a ConfigMap that contains database_url

#### b. We gonna create a secret that contains the credentials

#### We gonna refrence both ConfigMap and secret inside of that deployment files and once we have that setup, we gonna need mongo-express to be accessible through browser. In order to do that we gonna create a external service that will allow external request to talk to the pod.
### URL: IPaddressOfNode : PortOfExternalService

#### Browser Request Flow Through
    -> Request comes from the browser
    -> It goes to external service of mongo-express which will forward it to mongo-express pod
    -> the mongo-express pod will then connects to the internal service of mongodb (basically the database_url here) and it will forward it to mongodb pod

#### mongo-db will authenticate the request using credentials

#### 1. Creating a secret and deployment for mongodb. Secret configured gonna live in kubernetes and nobody has access to it and we have to create secret before the deployment, if we gonna reference the secret inside the deployment, order of creation matters.
``` kubectl apply -f mongo-secret.yaml ```

``` kubectl apply -f mongo.yaml ```

``` kubectl get all ``` (to get all deployment and services)

``` kubectl describe pod pod-name ``` (to get pod specifically)

#### 2. Second step is we gonna create a internal service, so that other components or pods can talk to this monogdb pod
``` kubectl apply -f mongo.yaml ``` (because deployment and service both are in mongo.yaml file)

``` kubectl get service service-name ``` (to validata service is attached to the correct pod)

#### 3. Next step, we gonna create mongo-express deployment and service and also an external configuration called ConfigMap where we gonna put database_url
    We need to tell 3 things like which database it should connect to, we need to tell database address (mongodb address) the internal service to connect to. and we need credentials so that mongodb can authenticate that connection, the env variables to do that ME_CONFIG_MONGODB_ADMINUSERNAME, ME_CONFIG_MONGODB_ADMINPASSWORD, ME_CONFIG_MONGODB_URL

#### 4. We can put database_url in configMap which is an external configuration so that it is centralized and also other components can use it
``` kubectl apply -f mongo-configmap.yaml ``` (apply configmap configuration)

``` kubectl apply -f mongo-express.yaml ``` (created deployment)

``` kubectl get pod ``` (to get list of pods, you should see both mongodb and mongoexpress now)

#### 5. Final step is to access mongo-express from the browser, inorder to do that we gonna need external service for mongo-express
  we can make service external by defining type:LoadBalancer (accepts external request by assigning the service an external IP address) 
  another things we need to make service external is we need to define nodePort. This is the port where external IP address will be open. Port we need to put in browser to access these service
  nodePort range needs to be between (30000 - 32767)

``` kubectl apply -f mongo-express.yaml ``` (created service)

#### 6. Internal service default is Cluster IP (clusterIP will give the service internal IP address, but LoadBalancer will give service internal and external IP address where the external request will be coming from

#### 7. Command to assign external service a public IP address
``` minikube service mongo-express-service ``` (minikube creates a tunnel and give a url to access the mongo-express application)




    
