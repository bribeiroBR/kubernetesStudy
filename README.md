# kubernetesStudy
It is using Alura.com examples

This project will take the basic information about kubernetes until build an app using it

01 - Kubernetes OVERVIEW

Container Manager just like the Docker Swarm


For example:
- Imagine we have a CLUSTER with a Master and 2 nodes, 1 for the web app and the other to the DB
- the master receives the YML (docker-compose) configuration saying we always need to have 1 container for DB and another one container to WEB
- In case the DB node stops for some reason the Kubernetes will recognise it and will, somehow manage to deploy the DB container in another node, machine.


Some important things to know are:

- CLUSTER
- Group of independent servers (usually in close proximity to one another) interconnected through a dedicated network to work as one centralised data processing resource


- POD OBJECT
- A Pod is the basic building block of Kubernetes
- –the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents a running process on your cluster.
- A Pod encapsulates an application container (or, in some cases, multiple containers), storage resources, a unique network IP, and options that govern how the container(s) should run


- SERVICE OBJECT
- Kubernetes PODs are mortal. They are born and when they die, they are not resurrected. We can create / destroy PODs easily, for instance using the scale option.
- The SERVICE object is an abstraction which defines a logical set of PODs and a policy by which to access them - sometimes called a micro-service


- NODE
- A node is a worker machine in Kubernetes, previously known as a minion . A node may be a VM or physical machine, depending on the cluster. Each node has the services necessary to run PODs and is managed by the master components


- DEPLOYMENT OBJECT
- It will encapsulate your PODs object that contains our containers which has your apps.
- Kubernetes will use this object to check the app status and make it consistence.
- For example if you specify in this Deployment object it should has a POD with some app and for some reason this app is down, at some point, the kubernetes will check this status and will put it up again.


- MINIKUBE
- it is a project that allow us to work with Kubernetes in our local machine
- INSTALLING MINIKUBE
- To use the minikube we need to:
- install http://download.virtualbox.org/virtualbox/ (latest version)
- install curl library
- change the environment variables
- sudo curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.22.3/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/


- KUBECTL
- kubectl is a tool that allow us to communicate with the cluster managed by the kubernets.
- INSTALLING KUBECTL
- sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/darwin/amd64/kubectl
- change the permission to be executable
- chmod +x ./kubectl
- move the kubectl to the environment variables
- sudo mv ./kubectl /usr/local/bin/kubectl

02 - Kubernetes Using Minikube for Management Actions


ALL ACTIONS YOU WANT TO DO IN THE CLUSTER MANAGEMENT YOU WILL USE THE “minikube” COMMANDS


HOW TO CREATE AND STARTS YOUR KUBERNETES MANAGEMER
- minikube start
- it will start the kubernetes and the cluster

- minikube stop
- it will stop the kubernetes and the cluster


HOW TO CHECK YOUR CLUSTER MANAGER STATUS AND LOGS
- minikube ip
- will return the ip of your running app
- minikube dashboard
- will open an web interface from of the kubernetes that is managing the clusters
- minikube logs
- will show up all the logs

03 - Kubernetes and kubectl cluster actions


ALL THE ACTIONS RELATED TO THE CLUSTER YOU WILL USE THE “kubectl” COMMAND.


HOW TO CREATE OUR POD OBJECT FROM THE CONFIGURATION FILE
- kubectl create -f $YAML_FILE_NAME


HOW TO DELETE OUR POD OBJECT
- kubectl delete $POD_NAME
or
- kubectl delete service,deployment $pod_service_name


HOW TO CHECK IF THE APP IS RUNNING IN THE CLUSTER
- kubectl get pods
- it will list all running PODS objects with their names, status…

04 - Kubernetes - Minikube and Kubectl Getting Start

In order to validate our deployment and check that everything went well, let’s start the local Kubernetes cluster:



minikube start
- starting our kubernetes’ cluster

output =
Starting local Kubernetes v1.7.5 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.



kubectl get pods --all-namespaces
- inspecting our cluster and getting the pods’ info

output =
NAMESPACE     NAME                          READY     STATUS    RESTARTS   AGE
kube-system   kube-addon-manager-minikube   1/1       Running   0          4m
kube-system   kube-dns-1326421443-9w5zp     3/3       Running   0          4m
kube-system   kubernetes-dashboard-4hqw8    1/1       Running   0          4m



kubectl get nodes
- inspecting our nodes

output =
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    7m        v1.7.5



kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
- Installing a Minikube Hello World. It is just a small project from kubernetes to getting start with it

output =
deployment "hello-minikube" created



kubectl get pods
- inspecting the pods

output =
NAME                             READY     STATUS    RESTARTS   AGE
hello-minikube-180744149-319vb   1/1       Running   0          2m



kubectl get deployments
- inspecting the deployment

output =
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1         1         1            1           4m



kubectl expose deployment hello-minikube --type=NodePort
- exposing the deployment to an external IP, in order to access the hello-minikube service
- P.S.: we must use the type=NodePort because minikube doesn’t support the LoadBalancer service

output =
service "hello-minikube" exposed



kubectl get services
- check if the service was exposed by listing services

output =
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort    10.0.0.160   <none>        8080:31243/TCP   2m
kubernetes       ClusterIP   10.0.0.1     <none>        443/TCP          19m



minikube service hello-minikube --url
- checking the external port to access it via browser

output =
http://192.168.99.100:31243



Now just access this URL via browser



kubectl delete service,deployment hello-minikube
- it will delete the service and the deployment called hello-minikube
- P.S.: In case of sdoubt about the service or deployment name just use the commands mentioned above to get the service and deployment names

output =
service "hello-minikube" deleted
deployment "hello-minikube" deleted


In order to double check if the service and the deployment were deleted we can run the
- kubectl get pods
- kubectl get services

05 - Kubernetes Build and Install a node service with Docker

Now we will create a small node server, build it into a Docker image with a Dockerfile, and run it in Kubernetes.

Let’s do it

KubernetesHelloDocker && cd KubernetesHelloDocker && touch Dockerfile server.js
- create a folder, enter into it and create a server.js file



vi server.js
var http = require('http');
var handleRequest = function(request, response) {
response.writeHead(200);
response.end('Hello World!');
};
var helloServer = http.createServer(handleRequest);
helloServer.listen(8080);

- editing the server.js file and adding the information to return a Hello Word message



vi Dockerfile
FROM node:4.4
EXPOSE 8080
COPY server.js .
CMD node server.js

- editing the Dockerfile and create this image based on the node:4.4 and the container will run the service based on the server.js



eval $(minikube docker-env)
- it will set up the docker environment variables. It is for the qinikube runtime usage



docker build -t kuberneteshellodocker:v1 .
- it will build our docker image based on our last configuration used in the Dockerfile



docker images
- it will display the built images



kubectl run kuberneteshellodocker --image=kuberneteshellodocker:v1 --port=8080
- deploying the kuberneteshellodocker pod to our local Kubernetes cluster
- P.S.: the pod status can be checked via kubectl get pods/deployments



kubectl expose deployment kuberneteshellodocker --type=NodePort
- exposing the deployment to an external IP



minikube service kuberneteshellodocker --url
- checking the external address to access the service



just get the URL displayed as a result of the above command and access it via browser



kubectl delete service,deployment kuberneteshellodocker
- deleting the pods and services created



minikube stop
- it will stop the kubernetes and the cluster

06 - Kubernetes and the App.YML configuration file

The app.yml file will describe all the configuration to build your app in the cluster

- Kubernetes Configuration file
- we have to create a configuration file with the YAML extension.
- usually this file has the following fields:
- apiVersion: (kubernetes version)
- kind: (object type to create in the cluster)
- metadata:
-   name: (app name)
- spec:
-   containers:
-     - name: (container name)
-       image: (image to based your container)
-       ports:
-         - containerPort: (port to be used)
- Example:
apiVersion: v1
kind: Pod
metadata:
name: app
spec:
containers:
- name: container-app-store
image: rafanercessian/aplicacao-loja:v1
ports:
- containerPort: 80


P.S.: This file will give instructions to create a POD object with your containers for your app, NOTE that it will not give instruction how to manage the cluster so far.



kubectl create -f app.yml
- it will create our pod based on the YAML file configuration
- P.S.: The -f is just to set the YAML file name



kubectl get pods
- will check if the pod was created or not
- P.S.: We also can check detailed information about our pods using:
- kubectl describe pods



kubectl expose deployment $POD_NAME --type=NodePort
- this will open an external IP connection for our service



kubectl get services
- will check if the service was exposed



minikube service $SERVICE_NAME --url
- it will display the URL we can access our app via browser

07 - Kubernetes and the Deployment.YML configuration file

The deployment.yml file will describe all the configuration to manage your POD object described in the app.YML file created before.

- Kubernetes Deployment file
- we have to create a deployment file with the YAML extension.
- usually this file has the following fields:
- apiVersion: (version)
- kind: (should be Deployment)
- metadata:
-   name: (name of your app)
- spec:
-   template:
-     metadata:
-       labels:
-         name: (name of your app defined in the app.yml file)
-     spec:
-       containers:
-         - name: (same from the app.yml file)
-           image: (same from the app.yml file)
-           ports:
-             - containerPort: (same from the app.yml file)
- Example:
apiVersion: apps/v1beta1
kind: Deployment
metadata:
name: app-deployment
spec:
template:
metadata:
labels:
name: app-pod
spec:
containers:
- name: container-app-store
image: rafanercessian/aplicacao-loja:v1
ports:
- containerPort: 80


P.S.: This file will give instructions to kubernetes how to manage the deployment that is encapsulating the PODs that contains your containers apps which has your app.



kubectl create -f deployment.yml
- it will create our pod based on the YAML file configuration
- P.S.: The -f is just to set the YAML file name



kubectl get pods
- will check if the pod was created or not
- P.S.: We also can check detailed information about our pods using:
- kubectl describe pods



kubectl expose deployment $POD_NAME --type=NodePort
- this will open an external IP connection for our service



kubectl get services
- will check if the service was exposed



kubectl get deployments
- will display all the deployments created



minikube service $SERVICE_NAME --url
- it will display the URL we can access our app via browser

08 - Kubernetes Scaling and Service Object


So, considering we need to scale our app, due to the increase of users accessing our app. Kubernetes will help us out with that.



1st let’s put our app up again:

kubectl create -f deployment.yml

kubectl expose deployment $POD_NAME --type=NodePort



Now let’s check if the pod was created and if the service is accessible from an external port

kubectl get pods

kubectl get services

kubectl describe pods | grep IP

minikube service $SERVICE_NAME --url



Now we can start working in the scale thing. To make our life easy we can use an UI to configure and change things in our kubernetes

minikube dashboard
- it will open the dashboard interface of our kubernetes, where we can manager all the clusters, pods, services…
- P.S.: Usually it opens the following address
http://192.168.99.100:30000/#!/overview?namespace=default


From now on, we have 2 ways. Either use the dashboard interface or use command line.


SCALE

kubectl scale --current-replicas=1 --replicas=3 deployment/$POD_NAME
- this command will scale from 01 to 03 pods

We can also go in the UI, in the left side we have a menu named DEPLOYMENTS, then we can click on it and in the right side of the listed pod we can click on SCALE and change


if we use the above commands to check the pods we will see 03 pods now and not 01 anymore, also each pod has it IP


P.S.: To decrease the number of pods we can use the same command used above, just changing the values.



As we can see working with POD, specially using the scale feature is ok, but it is also unstable, because they can be created or deleted any time. Due to work in a more stable environment we have to abstract this pods inside another object.


SERVICE OBJECT our Load Balancer

just remembering, The SERVICE object is an abstraction which defines a logical set of PODs and a policy by which to access them - sometimes called a micro-service.

Creating a Service Object

apiVersion: v1
kind: Service
metadata:
name: ANY NAME
spec:
type: LoadBalancer
ports:
- port: 80
selector:
name: NAME DESCRIBED IN THE DEPLOYMENT FILE IN THE LABELS NAME FIELD


EXAMPLE:
apiVersion: v1
kind: Service
metadata:
name: service-app
spec:
type: LoadBalancer
ports:
- port: 80
selector:
name: app-pod

P.S.: Important to note this configuration will already expose our app and save us some configuration lines



Now we will create this object in our cluster

kubectl create -f $SERVICE_FILENAME.YML
- creates our service object in our kubernetes



minikube service $SERVICE_NAME --url
- will show the URL to access our app

09 - Kubernetes POD DB, StatefulSets Object, Volumes, Permissions and Service Object Abstraction

Just as a reminder, the minimum object we are working with Kubernetes is the POD object.

1st - We will abstract our DB container to this object. For that we will create another YAML file, due to create this object in our cluster.

2nd - As we already know the POD is not a stable object, so we will need to abstract it, due to store all information and not loose them. But here the better object is not the Service Object, but the StatefulSets object.

3rd - We will need to create a volume in order to persist the information we want;

4th - Create a file to setup all necessary permissions

5th - Abstract the POD DB in the Service Object to make it stable

6th - Start Minukube and create all objects

7th - Create all necessary tables in our DB


StatefulSets Object
- is the workload API object used to manage stateful applications. It means:
- Stable, unique network identifiers and persistent storage.
- Ordered, deployment and scaling + deletion and termination + automated rolling updates.


CREATING THE DB POD OBJECT

apiVersion: v1
kind: Pod
metadata:
name: mysql
spec:
containers:
- name: $CONTAINER_NAME_YOU_WANT
image: mysql (from which image it will be based on)
ports:
- containerPort: 3306 (default DB port)
env: (those information should be the same information from our App DB configuration file)
- name: MYSQL_DATABASE
value: "loja"
- name: MYSQL_USER
value: "root"
- name: MYSQL_ALLOW_EMPTY_PASSWORD
value: "1"


Example:
apiVersion: v1
kind: Pod
metadata:
name: mysql
spec:
containers:
- name: container-mysql
image: mysql
ports:
- containerPort: 3306
env:
- name: MYSQL_DATABASE
value: "loja"
- name: MYSQL_USER
value: "root"
- name: MYSQL_ALLOW_EMPTY_PASSWORD
value: "1"



CREATING THE STATEFULSET OBJECT AND DEFINING THE VOLUME

apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
name: statefulset-mysql
spec:
serviceName: db (metadata name from the Service-db.yml file)
template: (inside this we have to put all the information from our pod-db.yml)
metadata:
labels: (we need to add this LABEL field before the information)
name: mysql
spec:
containers:
- name: (POD name)
image: mysql (default image for mysql DB)
ports:
- containerPort: 3306 (default port for DB)
env: (all below data should be exactly the same from the DB config file from the app)
- name: MYSQL_DATABASE
value: "loja"
- name: MYSQL_USER
value: "root"
- name: MYSQL_ALLOW_EMPTY_PASSWORD (allow empty password)
value: "1"
volumeMounts:
- name: volume-mysql
mountPath: /var/lib/mysql (path to persist the information)
volumes:
- name: volume-mysql (same name we used above)
persistentVolumeClaim: (calling the permission file setup)
claimName: config-mysql (same name used for the permission file in the metadata name variable)


Example:

apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
name: statefulset-mysql
spec:
serviceName: db
template:
metadata:
labels:
name: mysql
spec:
containers:
- name: container-mysql
image: mysql
ports:
- containerPort: 3306
env:
- name: MYSQL_DATABASE
value: "loja"
- name: MYSQL_USER
value: "root"
- name: MYSQL_ALLOW_EMPTY_PASSWORD
value: "1"
volumeMounts:
- name: volume-mysql
mountPath: /var/lib/mysql
volumes:
- name: volume-mysql
persistentVolumeClaim:
claimName: config-mysql



CREATING THE PERMISSION FILE

apiVersion: v1 (kubernetes api version)
kind: PersistentVolumeClaim (type used for permissions)
metadata:
name: config-mysql (name)
spec:
accessModes: (permission access)
- ReadWriteOnce
resources:
requests:
storage: 3Gi (using 3Gb for persistence)


Example

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: config-mysql
spec:
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 3Gi



CREATING THE SERVICE OBJECT FOR THE DB IN ORDER TO ABSTRACT THE POD DB OBJECT

apiVersion: v1
kind: Service
metadata:
name: db (the name configured in the db file configuration from the app)
spec:
type: ClusterIP (it means only the cluster inside this DB object can access this service)
ports:
- port: 3306 (default port for the DB)
selector:
name: mysql (same name specified in the POD DB object)

Example:

apiVersion: v1
kind: Service
metadata:
name: db
spec:
type: ClusterIP
ports:
- port: 3306
selector:
name: mysql



CREATING ALL OBJECTS IN OUR CLUSTER

1st - Start Minikube and Check its status
- minikube start
- minikube status

2nd - Create our service app object and check its status
- kubectl create -f app.yml (app YAML file)
- kubectl get pods
- kubectl get services

3rd - Getting the URL to access our app via browser
- minikube service service-app —url (service name, listed down in the previous command)

4 - Create the statefulset object and check its status
- kubectl create -f statefulset.yml
- kubectl get pods

5 - Create the DB service object that abstracts the POD db object and check its status
- kubectl create -f service-db.yml
- kubectl get services

6 - Creating the object with all the permissions
- kubectl create -f permissions.yml

7  - Access the DB
- kubectl exec -it statefulset-mysql-0 sh (DB POD name)
- mysql -u root

8 - Using the “loja” DB and Creating all tables
create table produtos (id integer auto_increment primary key, nome varchar(255), preco decimal(10,2));
alter table produtos add column usado boolean default false;
alter table produtos add column descricao varchar(255);
create table categorias (id integer auto_increment primary key, nome varchar(255));
insert into categorias (nome) values ("Futebol"), ("Volei"), ("Tenis");
alter table produtos add column categoria_id integer;
update produtos set categoria_id = 1;

9 - Accessing the app and adding some data
- minikube service service-app —url (service name, listed down in the previous command)

10 - Kubernetes final Considerations

SERVICE OBJECT
- it is an abstraction which defines a logical set of PODs and a policy (Controllers) by which to access them - sometimes called a micro-service;
- example: Service Object contains POD + Controllers (Deployment + StatefulSets + CronJob…);


PERSISTENTVOLUMECLAIM STORAGE OBJECT
- it is an object create to configure the storage of our app
- Example: it defines which path, the storage size…


STATEFULSETS CONTROLLER OBJECT
- it is a controller used like a workload to manage stateful applications. For example used to persist information.
- Example: Stable, unique network identifiers and persistent storage, order, deployment and scaling + deletion and termination + automated rolling updates.


DEPLOYMENT CONTROLLER OBJECT
- It will encapsulate PODs objects.
- Kubernetes will use this object to check the app status and make it consistence.
- For example if you specify in this Deployment object it should has a POD with some app and for some reason this app is down, at some point, the kubernetes will check this status and will put it up again.


POD OBJECT:
- A POD represents a running process on your cluster. A Pod encapsulates an application container;
- example: Containers + Images + App + DB…


CLUSTER
- Group of independent servers (usually in close proximity to one another) interconnected through a dedicated network to work as one centralised data processing resource


NODE
- A node is a worker machine in Kubernetes, previously known as a minion . A node may be a VM or physical machine, depending on the cluster. Each node has the services necessary to run PODs and is managed by the master components



HIGHLIGHTING
- StateFulSets Controller makes reference to
- Service Object
- PersistentVolumeClaim
- PODs Object

- Deployment Object makes reference to
- PODs object

- Service Object makes reference to
- PODs object

- POD Object makes reference to
- containers
- container images
- apps



COMMANDS OVERVIEW
Kubernetes Cluster Management
- minikube start
- minikube status
- minikube stop
- minikube dashboard
- minikube service $SERVICE_NAME --url


Kubernetes Resource Management
- kubectl creates -f $FILE_NAME.YAML
- kubectl delete (pods / services / deployments / statefulsets …) $OBJECT_NAME
- kubectl delete (pods / services / deployments / statefulsets …)--all
- kubectl get (pods / services / deployments / statefulsets …)
- kubectl exec -it $POD_DB_NAME sh
- kubectl expose deployment $POD_NAME --type=NodePort
- kubectl describe (pods / services / deployments / statefulsets …)
- kubectl scale --current-replicas=$CURRENT_SIZE --replicas=$DESIRED_SIZE deployment/$POD_NAME



BUILDING THE PROJECT

1 - Putting it up (working in the root path)
- minikube start
- it will start our kubernetes
- minikube status
- kubectl create -f deployment.yml
- it will deploy our app, create the pod and ensure that we always will have PODs running for the app
- kubectl get deployment
- kubectl get pods
- kubectl create -f service.yml
- it will abstract our POD inside this service and this service will also works as a load balancer
- kubectl get services
- minikube service app-service --url
- grab the URL and access it via browser
P.S.: Here our app is up, working with a load balancer and also our service ensures it will be always running

2 - Checking that our app is our running
- kubectl get pods
- kubectl delete pods --all
- it will delete all pods, but the service will ensure the pods will be up again
- kubectl get pods

3 - Scaling our app
- kubectl get services
- kubectl scale --current-replicas=1 --replicas=5 deployment/app-deployment
- it will scale our app from 1 to 5 pods
- kubectl get pods

4 - Creating our DB and linking it with our app (entering in the DB folder)
- kubectl create -f service-db.yml
- kubectl create -f statefulset.yml
- kubectl get pods
- kubectl get services
- kubectl get statefulsets
