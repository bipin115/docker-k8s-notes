# ---------------------------------------------------------
# ############ < Docker > #############
# ---------------------------------------------------------

`Defination Image` : Images are template /blueprints for containers, multiple containers can be based on same Image.
Images contains multiple layers (1 instruction = 1 layer ) to optimize build speed (Caching !) and reusability


`docker start <CONATINER_NAME>`               (Note: docker start by default runs this container in detach mode)
`docker start -a <CONATINER_NAME>`            (Note: -a <attached mode> docker start conatiner in attached mode)
`docker stop <CONATINER_NAME>`                (Note: stops the running container)

`docker run`                                  (Note: docker run start this container in attached mode.)
`docker run -p 3000:80 -d --rm <IMAGE_ID>`    (Note: --rm is for automatically removing the container when stopped.)

`docker log <CONATINER-NAME>`                 (Note: Gives log for the running container)

`docker log -f <CONATINER-NAME>`              (Note: -f <follow mode> helps to keep listening to the future outputs generated on the terminal)

#### INSPECTING IMAGES ####
`docker image inspect <IMAGE-ID>`            (Note: Gives you the details of the Image and the layers its built on)

#### COPY File to Containers and from container to local ####
# USECASE: To collect logs generated from a running container. OR to modify any server configuration file for test.

# COPY FROM LOCAL TO RUNNING CONTAINER
`docker cp <LOCAL_FOLDER>/<File_Name> <CONATINER_NAME>:/<FOLDER_TO_COPY>`

(Note: This above command will copy files from local to the running container. Instead of <File_Name> we can use . to copy all files to the container. If <FOLDER_TO_COPY> doesn't exist in container Docker will create the folder to copy.)

# COPY FROM RUNNING CONTAINER TO LOCAL
`docker cp <CONATINER_NAME>:/<FOLDER_TO_COPY> <LOCAL_FOLDER>`

#### NAMING & TAGGING A CONTAINER IMAGE ####

# IN CONTAINER
Use: --name
e.g `docker run -p 3000:80 -d --rm --name goalsapp <IMAGE_ID OR IAMGE_TAG>`
# IN IMAGE
Use: -t
e.g `docker build -t goals:latest .`


### TO NOTE ###
By default, if you run a Container without -d, you run in "attached mode".

If you started a container in detached mode (i.e. with -d), you can still attach to it afterwards without restarting the Container with the following command:

`docker attach <CONTAINER-ID>`

attaches you to a running Container with an ID or name of CONTAINER.

#### INCREASE RAM SIZE ####
This will set the maximum limit docker consume while running containers. Now run your image in new container with -m=4g flag for 4 gigs ram or more. e.g.

`docker run -m=4g {imageID}`

#### Sharing Images & Containers ####
1. Share the Dockerfile
2. Share the finished Image and run the container

Sharing via Dockerhub OR Private repository

# Docker Hub
`docker push <IMAGE-NAME>`
`docker pull <IMAGE-NAME>`

# Private Repository

`docker push <HOST>: <IMAGE_NAME>`
`docker pull <HOST>: <IMAGE_NAME>`

# Renaming an existing Image 
`docker tag <OLD_IMAGE_NAME>:tag <NEW_IAMGE_NAME>:tag` (Note: tag is not necessary. This command will create a clone of old Image with the new name )


#### External Data Storage ####

There are two types of external Data storage
 1. Volumes (Managed by Docker)
     a. Anonyomous Volume (Note: Volume would be created and destroyed on removing the container)
        e.g `docker run -d -p 3000:80 --rm --name feedback-app -v /app/feedback feedback-node:volumes`

     b. Named Volume (Limitation: can not be created from the Dockerfile. It can be created from cmd line while creating a container)   (Note: Volume persist even after removeable of Container)
        e.g `docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback feedback-node:volumes` 
        
 2. Bind Mounts (Managed by you) (Usecase: When you need to provide `live data` to the container no rebuilding of Image is required basically Development)

    e.g `docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "<PATH TO YOUR SOURCE CODE NODE APP FOLDER >:/app" -v /app/node_modules feedback-node:volumes` 
    
    (Note: Here the souces code from local will be overwritten onto container /app folder. It will be prone to give error on run as node_modules will not be found within the container for that issue to get resolved we have mapped  the node_modules with anonyomous volumes to keep node_modules intact which got created from Dockerfile during Image creation)

    macOS / Linux: -v $(pwd):/app
    Windows: -v "%cd%":/app

Now you just start piling up a bunch of unused anonymous volumes - you can clear them via docker volume rm VOL_NAME or docker volume prune

<IMPORTANT:>
 By Default Docker volumes are `read` and `Write`. In case of Bind mount Docker doesn't update anything on the local host but to be on safer side if we want to declare a volume read only. we can do that by:

`docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "<PATH TO YOUR SOURCE CODE NODE APP FOLDER >:/app:ro" -v /app/node_modules feedback-node:volumes`

ro --> READ ONLY

#### ARGUMENTS & ENV Variables ####
# USECASE : Both can be used to make Image and Containers more Dynamic/ Configurable

Arguments OR ARG   (Note: Avialable in Dockerfile, not accessible in cmd or any other application code. Set on Image builder using cmd via `--bild-arg`). Helps to build Image in flexible way, Instaed of modifying Dockerfile each time.

Environmanr Variables OR ENV (Note: Avialabl inside Dockerfile and in application code. Set via `ENV` in Dockerfile or via `--env` on docker run ). In case of multiple environment variables `--env or --e`

e.g `docker run -d -p 3000:80 --env PORT=80 --rm --name feedback-app -v feedback:/app/feedback -v "<PATH TO YOUR SOURCE CODE NODE APP FOLDER >:/app:ro" -v /app/node_modules feedback-node:volumes`

# Setting ENV in Dockerfile

 ENV PORT 8080
 EXPOSE $PORT

# In case you can to point to .env file
e.g `docker run -d -p 3000:80 --env-file ./.env --rm --name feedback-app -v feedback:/app/feedback -v "<PATH TO YOUR SOURCE CODE NODE APP FOLDER >:/app:ro" -v /app/node_modules feedback-node:volumes`

# Environment Variables & Security
One important note about environment variables and security: Depending on which kind of data you're storing in your environment variables, you might not want to include the secure data directly in your Dockerfile.

Instead, go for a separate environment variables file which is then only used at runtime (i.e. when you run your container with docker run).

Otherwise, the values are "baked into the image" and everyone can read these values via docker history <image>.

For some values, this might not matter but for credentials, private keys etc. you definitely want to avoid that!

#### NETWORKING - Cross Container Communication ####
`https://localhost:27017/swfavourites` ---> MongoDb URL

# If we need to interact with the mongoDb installed on our local host within the container we can modify the URL like

## <Container> to <Host System> communication
`https://host.docker.internal:27017/swfavourites` ---> Host MongoDb URL

## NOTE <host.docker.internal> == <System Host IP>

## <Container> to <Container> communication

`mongodb://<IPV4 adress container mongodb>:27017/swfavourites`  # mongodb is the container created from mongo Image.

Note: Ipv4 adress can be found using command `docker inspect <Container name OR ID>`

## <Container> to <Container> communication using Docker Network cmd
#  If both containers are part of same network the url can auto resolve the IP just by conatiner name

`mongodb://<Conatiner Name>:27017/swfavourites`  # mongodb is the container created from mongo 
e.g `mongodb://mongodb:27017/swfavourites`       # mongodb is the container created from mongo


# ---------------------------------------------------------
# ############ < Docker Compose > #############
# ---------------------------------------------------------
https://www.w3cschool.cn/doc_docker_1_13/docker_1_13-engine-reference-commandline-service_create-index.html

docker-compose up
docker-compose down

docker-compose up -d
docker-compose down --rmi all
docker-compose up --build


Deploying a Registry:
docker service create --name registry --publish published=5000,target=5000 registry:2
docker service ps --format 'table {{.ID}}\t{{.Name}}\t{{.Node}}\t{{.CurrentState}}' registry
curl localhost:5000/v2/_catalog   ----> O/P : {"repositories":[]}
curl -X GET localhost:5000/v2/_catalog

$ for i in rng hasher worker webui; do docker image tag $i:v1 localhost:5000/$i:v1; done
$ docker image ls --format 'table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.Size}}'

$ for i in rng hasher worker webui; do docker image push localhost:5000/$i:v1; done
$ curl localhost:5000/v2/_catalog
{"repositories":["hasher","rng","webui","worker"]}

$ docker stack deploy -c docker-compose.yml dockercoins
$ docker stack services --format 'table {{.Name}}\t{{.Mode}}\t{{.Replicas}}\t{{.Image}}\t{{.Ports}}' dockercoins
$ docker stack ps --format 'table {{.Name}}\t{{.Node}}\t{{.CurrentState}}' dockercoins

# $ docker stack rm dockercoins



# ---------------------------------------------------------
# ############ < Docker Swarm > #############
# ---------------------------------------------------------
https://www.youtube.com/watch?v=KC4Ad1DS8xU

# What is Docker Swarm?
Docker Swarm is an orchestration management tool that runs on Docker applications. It helps end-users in creating and deploying a cluster of Docker nodes.

Each node of a Docker Swarm is a Docker daemon, and all Docker daemons interact using the Docker API. Each container within the Swarm can be deployed and accessed by nodes of the same cluster. 

Benifits:
    can help in acieveing zero downtime deployment.
    scalable

# `docker swarm init`                                         (Initializes Docker Swarm and make things reday to deploy applications on Swarm)
# `docker swarm join <IP-Adress-Host>:2377`                   (To make Node joined swarm as Worker)
# `docker stack deploy -c docker-compose.yaml swarmpyapp`     (Deploy compose on swarm)
# `docker service ls`                                         (To check ports and replicas)
# `docker stack ls`                                           (To check stacks deployed on swarm)
# `docker service scale <swarm-servicename>=20`               (Scale appliation to 20 replicas)
# `docker stack rm <service-name>`                            (To remove stack deployed application on Swarm)

# `docker swarm init --listen-addr <IP-Adress-Host>:2377`      (Creates a Manager Node)
# `docker node ls`                                             (Status of Docker swarm nodes)

# Note: Each node should have one Master Manager
# To create a swarm add other VM's running docker daemon

`docker swarm join <IP-Adress-Host>:2377`  (Node joined swarm as Worker)

`docker service create --name website --publish 80:80 sixeyed/docker-swarm-walkthrough`
`docker service inspect website --pretty`
`docker service tasks website`

`docker service update --replicas=5` (Spin up new Instances)
SSS
`docker service create --replicas 5 nginx`

# Docker Swarm: Create and Scale a Service
In this lab we will create a new docker swarm cluster: one manger node and three worker nodes, then create a service and try to scale it.

# Create a Swarm Cluster
Based on the lab Swarm Mode: Create a Docker Swarm Cluster, create four docker machines and init a swarm cluster.

➜  ~ docker-machine ls
NAME            ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
swarm-manager   -        virtualbox   Running   tcp://192.168.99.103:2376           v1.12.5
swarm-worker1   -        virtualbox   Running   tcp://192.168.99.104:2376           v1.12.5
swarm-worker2   -        virtualbox   Running   tcp://192.168.99.105:2376           v1.12.5
swarm-worker3   -        virtualbox   Running   tcp://192.168.99.106:2376           v1.12.5
➜  ~

docker@swarm-manager:~$ docker node ls
ID                           HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
0skz2g68hb76efq4xknhwsjt9    swarm-worker2  Ready   Active
2q015a61bl879o6adtlb7kxkl    swarm-worker3  Ready   Active
2sph1ezrnr5q9vy0683ah3b90 *  swarm-manager  Ready   Active        Leader
59rzjt0kqbcgw4cz7zsfflk8z    swarm-worker1  Ready   Active
docker@swarm-manager:~$

# Create a Service
# Use docker service create command on manager node to create a service

docker@swarm-manager:~$ docker service create --name myapp --publish 80:80/tcp nginx
7bb8pgwjky3pg1nfpu44aoyti
docker@swarm-manager:~$ docker service inspect myapp --pretty
ID:           7bb8pgwjky3pg1nfpu44aoyti
Name:         myapp
Mode:         Replicated
 Replicas:    1
Placement:
UpdateConfig:
 Parallelism: 1
 On failure:  pause
ContainerSpec:
 Image:               nginx
Resources:
Ports:
 Protocol = tcp
 TargetPort = 80
 PublishedPort = 80
docker@swarm-manager:~$
Open the web browser, you will see the nginx page http://192.168.99.103/

# Scale a Service
We can use docker service scale to scale a service.

docker@swarm-manager:~$ docker service scale myapp=2
myapp scaled to 2
docker@swarm-manager:~$ docker service inspect myapp --pretty
ID:           7bb8pgwjky3pg1nfpu44aoyti
Name:         myapp
Mode:         Replicated
 Replicas:    2
Placement:
UpdateConfig:
 Parallelism: 1
 On failure:  pause
ContainerSpec:
 Image:               nginx
Resources:
Ports:
 Protocol = tcp
 TargetPort = 80
 PublishedPort = 80
In this example, we scale the service to 2 replicas.




# ---------------------------------------------------------
# ############ < KUBERNATES NOTES > #############
# ---------------------------------------------------------
Defination: Open Source Conatiner Orchestration tool

Kubelet ---> On each Master and Slave
API Server
Scheduler
Controll Manager
etcd

# --------------------------------------
`Main Kubernates Components ?`
# ---------------------------------------
Volumes
Pods
Service
Ingress
Deployment
Statefullset
ConfigMap
Secret

`Pods`:
      1. Smallest Unit of K8s 
      2. Abstraction Over Container
      3. Usually 1 application per pod
      4. Each Pods gets its own IP address (Internal IP)

      (Note: ISSUE: In case the POD dies then a new IP gets assigned to the POD on recreation )
       Solution: Another Kubernates components called Service is used

`Service`: (Provides Static IP for the PODS)
      1. Permanent IP address
      2. Lifecycle of POD and service are not connected (benifit: Even if the POD dies now the IP doesnt change and hence not affecting th endpoint)

      Types:
           a. External Service (Accesing application end point from host browser)
           b. Internal Service (For Database as we don't want to expose it to the outer world)


`Ingress`: To make the application end point look more practical domain
      e.g http://172.22.185.45:8080   -----> https://my-app.com
      Note (Request First goes to the Ingress and then its routed to the service)

`Config Map and Secrets`:
      External configuartion of your application (e.g url of Db or other services we use)
      
      Secrets: 
             a. Used to store secret data (e.g Credentails and certificates)
             b. Base64 encoded

`Volume`:
      Storage on local machine (Like DB data)
      Attaches physical Hard drive to the POD

`Deployments`:
      Blueprint for my-app pod
      You create deployments 
      Abstaraction over pods

      Note: Database can not be replicated using Deployments as Db's will have a state

# ---------------------------------------------

`What Features do Orchestartion tools offer ?`

High Availabilty or no down time
Scalability or high performance
Disaster Recover - backup and Restore

# ---------------------------------------------

# Processes Running on Every Master Node
4 processes are running on every Master Node
   1. API server :- Enables Interaction with cluster acts a entry point or gateway from client (Clinet could be command line tool <e.g kubectl> or Kubernates dashboard or Kubernates API)

   2. Scheduler
   3. Controller Manager
   4. etcd <Key value Store> also can be termed as cluster brain as it holds information related to resource availability and the cluster state information and any change (change like pods running or dead or crashed), Cluster health, 
   Note: In etcd application data is not stored.

# Processes Running on Every Worker Node
   1. Kubelet --- Runs the pods using container runtime on worker nodes
   2. Kubeproxy
   3. Container Runtime

# Example Cluster Setup
 Generally a Cluster Setup will be of 
   1. 2 Master Nodes  -- Requires Less resources (CPU | RAM | STORAGE)
   2. 3 Slave Nodes   -- Requires More resources (CPU | RAM | STORAGE)


# ################# MINIKUBE && KUBECTL ###################################
# <minikube>

Minikube -- Its a one Node cluster setup on local or virtual machine where both Master process and Worker processes run together for testing purpose.

Basically it creates a Virutual Node on local machine using the Virtual Box Installation.

# NOTE: Minikube comes with Docker preInstalled.

# <kubectl> required for configuring the minikube clusters

Command line tool for kubernates cluster to create pods and other kubernates components.

# <INSTALLATION> Minikube and Kubectl  -----> https://phoenixnap.com/kb/install-minikube-on-ubuntu
# How to Install Minikube on Ubuntu 20.04 LTS / 21.04 ---> https://www.linuxtechi.com/how-to-install-minikube-on-ubuntu/


 Note: Minikube can only be installed on a system where Virtualization is enabled and Hypervisor(e.g Virtaul Box) is installed

Note: Minikube has kubectl as dependency

 Minimum system requirements for minikube
2 GB RAM or more
2 CPU / vCPU or more
20 GB free hard disk space or more
Docker / Virtual Machine Manager – KVM & VirtualBox

# Start Minikube
`minikube start --driver=none`
`minikube start --driver=docker`

# Interact with your cluster
https://minikube.sigs.k8s.io/docs/start/


# Clean up Minikube and Re-Install in case of Instllation failure.
Solution:
      first:
            docker system prune
      second:
            minikube delete
      and finally:

            minikube start --driver=docker

# ############### Basics kubectl Commands ####################

Create and Debug Pods in a minikube cluster

# CRUD Commands
Create deployment             ----> kubectl create deployment [name]
Edit deployment               ----> kubectl edit deployment [name]
Delete deployment             ----> kubectl delete deployment [name]

# Status of different k8s components
kubectl get nodes | pods | services | replicaset | deployment

# Debugging Pods
Log to console                -----> kubectl logs [pod_name]
Get Interactive Terminal      -----> kubectl exec -it [pod_name] -- bin/bash
Get Info about POD            -----> kubectl describe pod [pod_name]          || kubectl describe pod -o wide [pod_name]
Get Service Info              -----> kubectl describe service [service_name]  || kubectl describe service -o wide [service_name]

kubectl apply -f [file_name]  ----> To apply configuration file (i.e creating deployment with yaml script)
kubectl delete -f [file_name] ----> Delete with configuration file

# Kubernetes Basics Modules

1. Create a Kubernetes cluster
   Cluster details
   Let’s view the cluster details. We’ll do that by running kubectl cluster-info:
   `kubectl cluster-info`

    To view the nodes in the cluster, run the kubectl get nodes command
    `kubectl get nodes`


2. Deploy an app

Let’s deploy our first app on Kubernetes with the kubectl create deployment command. We need to provide the deployment name and app image location (include the full repository url for images hosted outside Docker hub).

kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/kubernetes-bootcamp created

kubectl get deployments

3. Explore your app
4. Expose your app publicly
5. Scale up your app
6. Update your app

# ############# Layers of Abstraction ##################

Deployment is an abstraction over replicaset and manages a Replicaset..
Replicaset is an abstraction over pod and Replicaset manages a pod ...
pod is an abstraction of container and manages a container...
container

anything below container level can be easily managed directly by kubernates


# ################## K8's YAML Configuration File ####################
- The 3 parts of configuration file (1. Metadata 2. Specification 3. Status (Automatically genearted by Kubernates. It matches for Destired == Actual state))

- Connecting Deployments to Services to pods
- demo


`kubectl get deployment ngnix-deployment -o yaml > ngnix-deployment.yaml`

# Complete Application setup <Mongo express and MongoDb> with Kubernates Components

# Create a Secret yaml component and store the base64 encoded userId and password

`echo -n 'db_userName' | base64`
`echo -n 'db_userPasword' | base64`

NOTE: Apply the secret before using in the deployment yaml file if any or else it will trow error during applying deployment.

Note to List All Kubernates components <pods, deployment, replcaset, service>: `kubectl get all`

# ################################################################################################
                                 
                        KUBERNATES SERVICES

# ################################################################################################

Different Service Type?

1. ClusterIP Services
2. NodePort Services
3. Headless Services
4. LoadBalancer Services

What is service and when we need it ?

Each Pod has its own IP adress and It changes when Pods are destroyed and recreated.

# svc -> Provides a satic IP adress and loadbalancing to Pods to acess running containers within the Pods

ClusterIP services -> default Type
check IP adress of pods use cmd -> kubectl get pod -o wide
'OUTPUT:

C:\Users\11459>kubectl get pod -n argocd -n argocd -o wide

NAME                                               READY   STATUS    RESTARTS       AGE     IP           NODE             NOMINATED NODE   READINESS GATES
argocd-application-controller-0                    1/1     Running   2 (3d4h ago)   5d2h    10.1.0.110   docker-desktop   <none>           <none>
argocd-applicationset-controller-87fc4b8d7-zdwx7   1/1     Running   3 (3d4h ago)   5d2h    10.1.0.103   docker-desktop   <none>           <none>
argocd-dex-server-7bf478fc94-8wvv6                 1/1     Running   2 (3d4h ago)   5d2h    10.1.0.104   docker-desktop   <none>           <none>
argocd-notifications-controller-5964fbfc9d-v7l9w   1/1     Running   2 (3d4h ago)   5d2h    10.1.0.113   docker-desktop   <none>           <none>
argocd-redis-5dff748d9c-rlt4t                      1/1     Running   2 (3d4h ago)   5d2h    10.1.0.111   docker-desktop   <none>           <none>
argocd-repo-server-6c8778b8d8-98fgg                1/1     Running   2 (3d4h ago)   5d2h    10.1.0.114   docker-desktop   <none>           <none>
argocd-server-69d84fdd78-dqf5c                     1/1     Running   2 (3d4h ago)   5d2h    10.1.0.112   docker-desktop   <none>           <none>
flask-app-deploy-86bdff9454-hn2h7                  1/1     Running   0              2d18h   10.1.0.119   docker-desktop   <none>           <none>
flask-app-deploy-86bdff9454-n78jp                  1/1     Running   0              2d18h   10.1.0.118   docker-desktop   <none>           <none>

=======================================================================================================================







