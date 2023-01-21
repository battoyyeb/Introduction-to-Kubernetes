# Introduction-to-Kubernetes

Let us assume we now have our application packaged into a Docker container. How do you run it in production? What if your
application relies on other containers such as database or messaging services or other backend services? What if the number of users increase and you need to scale your application? You would also like to scale down when the load decreases. To enable these functionalities you need an underlying platform with a set of resources. The platform needs to orchestrate the connectivity between the containers and automatically scale up or down based on the load. This whole process of automatically deploying and managing containers is known as Container Orchestration. Kubernetes is thus a container orchestration technology. 

There are various advantages of container orchestration. Your application is now highly available as hardware failures do not bring your application down because you have multiple instances of your application running on different nodes. The user traffic is load balanced across the various containers. When demand increases, deploy more instances of the application seamlessly and within a matter of second and we have the ability to do that at a service level. When we run out of hardware resources, scale the number of nodes up/down without having to take down the application. And do all of these easily with a set of declarative object configuration files.

## Basic Concept of Kubernetes

Before we head into setting up a kubernetes cluster, it is important to understand some of the basic concepts. This is to make sense of the terms that we will come across while setting up a kubernetes cluster. 

Let us start with Nodes. A node is a machine – physical or virtual – on which kubernetesis installed. A node is a worker machine and this is were containers will be launched by kubernetes.

A cluster is a set of nodes grouped together. This way even if one node fails you have your application still accessible from the other nodes. Moreover having multiple nodes helps in sharing load as well.

Now we have a cluster, but who is responsible for managing the cluster? Were is the information about the members of the cluster stored? How are the nodes monitored? When a node fails how do you move the workload of the failed node to another worker node? That’s were the Master comes in. The master is another node with Kubernetes installed in it, and is configured as a Master. The master watches over the nodes in the cluster and is responsible for the actual orchestration of containers on the worker nodes.


## Kubernetes Components

When you install Kubernetes on a System, you are actually installing the following components, API Server, ETCD service, kubelet service, Container Runtime, Controllers and Schedulers.

The API server acts as the front-end for kubernetes. The users, management devices,Command line interfaces all talk to the API server to interact with the kubernetes cluster.

ETCD is a distributed reliable key-value store used by kubernetes to store all data used to manage the cluster. Think of it this way, when you have multiple nodes and multiple masters in your cluster, etcd stores all that information on all the nodes in the cluster in a distributed manner. ETCD is
responsible for implementing locks within the cluster to ensure there are no conflicts between the Masters.

The scheduler is responsible for distributing work or containers across multiple nodes. It looks for newly created containers and assigns them to Nodes. 

The controllers are the brain behind orchestration. They are responsible for noticing and responding when nodes, containers or endpoints goes down. The controllers makes decisions to bring up new containers in such cases.

The container runtime is the underlying software that is used to run containers. In our case it happens to be Docker.

And finally kubelet is the agent that runs on each node in the cluster. The agent is responsible for making sure that the containers are running on the nodes asexpected.

## Master and Worker node Concept

The worker node (or minion) as it is also known, is were the containers are hosted. For example Docker containers, and to run docker containers on a system, we need a container runtime installed. And that’s were the container runtime falls. In this case it happens to be Docker. The worker nodes have the kubelet agent that is responsible for interacting with the master to provide health information of the worker node and carry out actions requested by the master on the worker nodes.

The master server has the kube-apiserver and that is what makes it a master. All the information gathered are stored in a key-value store on the Master. The key value store is based on the popular etcd framework as we just discussed. The master also has the controller manager and the scheduler. There are other components as well, but we will stop there for now. The reason we went through this is to understand what components constitute the master and worker nodes. This will help us install and configure the right components on different systems when we setup our infrastructure.


## Kubectl

The kube control tool is used to deploy and manage applications on a kubernetes cluster, to get cluster information, get the status of nodes in the cluster and many other things. The kubectl run command is used to deploy an application on the cluster. The kubectl cluster-info command is used to view information about the cluster and the kubectl get pod command is used to list all the nodes part of the cluster.

Install Docker
```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce
sudo systemctl status docker
sudo usermod -aG docker ${USER}
```
Now exit and ssh back into the server before you proceed.


Install kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
```
To get the version of kubenetes run

```
kubectl version --client --output=yaml
```

To check the flavor and version of operating system on which the Kubernetes nodes are running, Run

```
kubectl get nodes -o wide
```

Install minikube

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Install conntrack
```
sudo apt-get update -y
sudo apt-get install -y conntrack
```

Install cri-dockerd
```
git clone https://github.com/Mirantis/cri-dockerd.git
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile

cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
sudo cp -a packaging/systemd/* /etc/systemd/system
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
```

Install crictl
```
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.25.0/crictl-v1.25.0-linux-amd64.tar.gz
sudo tar zxvf crictl-v1.25.0-linux-amd64.tar.gz -C /usr/local/bin
```
Run
```
minikube start --vm-driver=none
kubectl get nodes
kubectl get po -A
```


Create a networkoverlay
```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl -n kube-system apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```


Install kubectl and minikube on your laptop. (https://kubernetes.io/docs/tasks/tools)

<img width="885" alt="Screen Shot 2022-11-23 at 8 42 38 PM" src="https://user-images.githubusercontent.com/74343792/203675480-27a07556-5c29-4db5-bcc7-345c7a0fe60c.png">

<img width="893" alt="Screen Shot 2022-11-23 at 8 43 14 PM" src="https://user-images.githubusercontent.com/74343792/203675488-a4389c98-11d9-4105-96aa-4fac785579cb.png">

Test using a deployment to confirm everything is fine (All the commands are available in the link above. you will see it after insttalling minikube)

<img width="762" alt="Screen Shot 2022-11-23 at 8 55 44 PM" src="https://user-images.githubusercontent.com/74343792/203676856-73307d20-d603-4694-8a50-a11821d14a7d.png">

<img width="901" alt="Screen Shot 2022-11-23 at 8 51 50 PM" src="https://user-images.githubusercontent.com/74343792/203676479-5a633adc-a71b-44c3-8082-b8ee30101357.png">


## Pods

Before we head into understanding PODs, we would like to assume that the following have been setup already. At this point, we assume that the application is already developed and built into Docker Images and it is availalble on a Docker repository like Docker hub, so kubernetes can pull it down. We also assume that the Kubernetes cluster has already been setup and is working. This could be a single-node setup or a multi-node setup, doesn’t matter. All the services need to be in a running state.

The ultimate aim of kubernetes is to deploy our application in the form of containers on a set of machines that are configured as worker nodes in a cluster. However, kubernetes does not deploy containers directly on the worker nodes. The containers are encapsulated into a Kubernetes object known as PODs. A POD is a single instance of an application. A POD is the smallest object, that you can create in kubernetes. What if the number of users accessing your application increase and you need to scale your application? You need to add additional instances of your web application to share the load. Now, were would you spin up additional instances? Do we bring up a new container instance within the same POD? No! We create a new POD altogether with a new instance of the same application. We can have two instances of our web application running on two separate PODs on the same kubernetes system or node. What if the user base FURTHER increases and your current node has no sufficient capacity? Well THEN you can always deploy additional PODs on a new node in the cluster. You will have a new node added to the cluster to expand the cluster’s physical capacity. PODs usually have a one-to-one relationship with containers running your application. To scale UP you create new PODs and to scale down you delete PODs. You do not add additional containers to an existing POD to scale your application.

We just said that PODs usually have a one-to-one relationship with the containers, but, are we restricted to having a single container in a single POD? No! A single POD CAN have multiple containers, except for the fact that they are usually not multiple containers of the same kind. If our intention was to scale our application, then we would need to create additional PODs. But sometimes you might have a scenario were you have a helper container, that might be doing some kind of supporting task for our web application such as processing a user entered data, processing a file uploaded by the user etc. and you want these helper containers to live along side your application container. In that case, you CAN have both of these containers part of the same POD, so that when a new application container is created, the helper is also created and when it dies the helper also dies since they are part of the same POD. The two containers can also communicate with each other directly by referring to each other as ‘localhost’ since they share the same network namespace. Plusthey can easily share the same storage space as well.

An example: Let’s assume we were developing a process or a script to deploy our application on a docker host. Then we would first simply deploy our application using a simple docker run python-app command and the application runs fine and our users are able to access it. When the load increases we deploy more instances of our application by running the docker run commands many more times. This works fine and we are all happy. Now, sometime in the future our application is further developed, undergoes architectural changes and grows and gets complex. We now have new helper containersthat helps our web applications by processing or fetching data from elsewhere. These helper containers maintain a oneto-one relationship with our application container and thus, needs to communicate with the application containers directly and access data from those containers. For this we need to maintain a map of what app and helper containers are connected to each other, we would need to establish network connectivity between these containers ourselves using links and custom networks, we would need to create shareable volumes and share it among the containers and maintain a map of that as well. And most importantly we would need to monitor the state of the application 47 container and when it dies, manually kill the helper container as well as its no longer required. When a new container is deployed we would need to deploy the new helper container as well. With PODs, kubernetes does all of this for us automatically. We just need to define what containers a POD consists of and the containers in a POD by default will have access to the same storage, the same network namespace, and same fate as in they will be created together and destroyed together. Even if our application didn’t happen to be so complex and we could live with a single container, kubernetes still requires you to create PODs. But this is good in the long run as your application is now equipped for architectural changes and scale in the future.


To deploy a container on kubenetes and also view it run

```
kubectl run nginx --image=nginx
kubectl get pods
kubectl describe pod nginx
kubectl delete pod nginx
```
<img width="851" alt="Screen Shot 2022-11-23 at 9 25 12 PM" src="https://user-images.githubusercontent.com/74343792/203680483-293052a3-a4d4-42ff-9371-78aa056df7c6.png">

## Run a Pod 

Create pod.yaml

```
vim pod.yaml
```

Paste
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec: 
  containers:
  - name: nginx
    image: nginx
```
Run
```
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod nginx
```

![Screen Shot 2023-01-20 at 6 46 46 PM](https://user-images.githubusercontent.com/74343792/213824669-6c0e6eb1-c108-4b62-9375-8545364bbf74.png)

Run  the following commnad to delete

```
kubectl delete pods <name>
```

## Replication controller

Controllers are the brain behind Kubernetes. They are processes that monitor kubernetes objects and respond accordingly. Let us talk about the replication controller

What if for some reason, our application crashes and the POD fails? Users will no longer be able to access our application. To prevent users from losing access to our application, we would like to have more than one instance or POD running at the same time. That way if one fails we still have our application running on the other one. The replication controller helps us run multiple instances of a single POD in the kubernetes cluster thus providing High Availability. So does that mean you can’t use a replication controller if you plan to have a single POD? No! Even if you have a single POD, the replication controller can help by automatically bringing up a new POD when the existing one fails. Thusthe replication controller ensures that the specified number of PODs are running at all times. Even if it’s just 1 or 100.


Another reason we need replication controller is to create multiple PODs to share the load across them. For example, in this simple scenario we have a single POD serving a set of users. When the number of users increase we deploy additional POD to balance the load across the two pods. If the demand further increases and If we were to run out of resources on the first node, we could deploy additional PODs across other nodes in the cluster. As you can see, the replication controller spans across multiple nodes in the cluster. It helps us balance the load across multiple pods on different nodes as well as scale our application when the demand increases.

The replication controller and Replica set are the same with everything said above. But there are minor difference in the way each of them works. 
The main difference is the selector. It is anly required in ReplicaSet. The The selector section helps the replicaset identify what pods fall under it i.e replica set can ALSO manage pods that were not created as part of the eplicaset creation. Say for example, there were pods created BEFORE the creation of the ReplicaSet that match the labels specified in the selector, the replica set will also take THOSE pods into consideration when creating the replicas.

```
kubectl create -f replicaset-definition.yml  - used to create a replca set
kubectl get replicaset  - to see list of replicasets created
kubectl delete replicaset myapp-replicaset  -  to delete the replicaset.
kubectl replace -f replicaset-definition.yml - to replace or update replicaset
kubectl scale –replicas=6 -f replicaset-definition.yml  -  to scale the replicas simply from the command line without having to modify the file
```

Make a directory for pods and replicasets

```
mkdir pods
mkdir replicasets
```

In the pods directory, create a yaml file for nginx.

```
vim nginx.yaml
```

Paste the code below
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
  labels:
    app: myapp
spec:
  containers:
  - name: nginx
    image: nginx
```

In the replicasets directory, create a replicaset file

```
vim replicaset
```

Paste the code below
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  replicas: 3
  template:
    metadata:
      name: nginx-2
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
 ```
 
 Run 
 
 ```
 kubectl create -f replicaset.yaml
 kubectl get replicaset
 kubectl get pods
 ```
 
 ![Screen Shot 2023-01-20 at 8 49 52 PM](https://user-images.githubusercontent.com/74343792/213835707-ec33d71b-1d46-40b4-91f6-c51ebefe8c90.png)

Try to create a pod that has the same label as the replica set, and you will notice it won't add it to the number of pods running becase our replicaset has been set to 3. Also try to delete one of the pods and it aill automatically create a new pod.

![Screen Shot 2023-01-20 at 9 03 04 PM](https://user-images.githubusercontent.com/74343792/213837268-c4f7569d-4261-469a-acaf-feb5191b2187.png)

Change the number of replicasets from 3 to 4.

![Screen Shot 2023-01-20 at 9 07 03 PM](https://user-images.githubusercontent.com/74343792/213838252-83330e4b-c4df-49ea-b70d-d54845edf129.png)

To delete the rplicaset, use this command
```
kubectl delete replicaset myapp-replicaset
```


## Kubenetes Deployments

Say for example you have a web server that needs to be deployed in a production environment. You need not ONE, but many such instances of the web
server running for obvious reasons. Secondly, when newer versions of application builds become available on the docker registry, you would like to UPGRADE your docker instances seamlessly. However, when you upgrade your instances, you do not want to upgrade all of them at once as we just did. This may impact users accessing our applications, so you may want to upgrade them one after the other. And that kind of upgrade is known as Rolling Updates. Suppose one of the upgrades you performed resulted in an unexpected error and you are asked to undo the recent update. You would like to be able to rollback the changes that were recently carried out. Finally,say for example you would like to make multiple changes to your environment such as upgrading the underlying WebServer versions, as well as scaling your environment and also modifying the resource allocations etc. You do not want to apply each change immediately after the command is run, instead you would like to apply a pause to your environment, make the changes and then resume so that all changes are rolled-out together. All of these capabilities are available with the kubernetes Deployments.

Create a new directory called deolyments

```
mkdir deployments
```

Create a deployment.yaml file

```
vim deployment.yaml
```

Paste the following code

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    tier: frontend
    app: nginx
spec:
  selector:
    matchLabels:
      app: myapp
  replicas: 3
  template:
    metadata:
      name: nginx-2
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
```

Run
```
kubectl create -f deployment.yaml
kubectl get deployments
kubectl get pods

or 

kubectl get all

kubectl describe deployment myapp-deployment
```

To delete
```
kubectl delete deploy myapp-deployment
```

![Screen Shot 2023-01-20 at 9 54 56 PM](https://user-images.githubusercontent.com/74343792/213840723-044373b7-67b7-4126-bcce-da3093aba03b.png)

Whenever you create a new deployment or upgrade the images in an existing deployment it triggers a Rollout. A rollout is the process of gradually deploying or upgrading your application containers. When you first create a deployment, it triggers a rollout. A new rollout creates a new Deployment revision. Let’s call it revision 1. In the future when the application is upgraded – meaning when the container version is updated to a new one – a new rollout is triggered and a new deployment revision is created named Revision 2. This helps us keep track of the changes made to our deployment and enables us to rollback to a previous version of deployment if necessary.


Change the number of replicas to 6.

Run
```
kubectl create -f deployment.yaml --record
kubectl rollout status deployment.apps/myapp-deployment
kubectl rollout history deployment.apps/myapp-deployment
kubectl describe deployment myapp-deployment
kubectl get pods
```

![Screen Shot 2023-01-20 at 10 14 02 PM](https://user-images.githubusercontent.com/74343792/213841234-b2edcb22-0423-4cc2-bc88-eb81e0d2cb94.png)


Now let us try to make a change to the deployments. In the deployment.yaml file, change the nginx in container section to nginx:1.18

![Screen Shot 2023-01-20 at 10 25 23 PM](https://user-images.githubusercontent.com/74343792/213841553-076028c6-c743-41ef-9aeb-367c9c1dc79c.png)

![Screen Shot 2023-01-20 at 10 25 40 PM](https://user-images.githubusercontent.com/74343792/213841559-3db61af3-7708-449a-9a30-cc9f0bb5d769.png)

To undo a rollout, run

```
kubectl rollout undo deployment/myapp-deployment
```

## Kubernetes Networking

Let us look at the very basics of networking in Kubernetes. We will start with a single node kubernetes cluster. The node has an IP address, say it is 192.168.1.2 in this case. This is the IP address we use to access the kubernetes node, SSH into it etc. On a side note, remember if you are using a MiniKube setup, then I am talking about the IP address of the minikube virtual machine inside your Hypervisor. Your laptop may be having a different IP like 192.168.1.10. So its important to understand how VMs are setup. So on the single node kubernetes cluster we have created a Single POD. As you know a POD hosts a container. Unlike in the docker world were an IP address is always assigned to a Docker CONTAINER, in Kubernetes the IP address is assigned to a POD. Each POD in kubernetes gets its own internal IP Address. In this case its in the range 10.244 series and the IP assigned to the POD is 10.244.0.2. So how is it getting this IP address? When Kubernetes is initially configured it creates an internal private network with the address 10.244.0.0 and all PODs are attached to it. When you deploy multiple PODs, they all get a separate IP assigned. The PODs can communicate to each other through this IP. But accessing other PODs using this internal IP address MAY not be a good idea as its subject to change when PODs are recreated. We will see BETTER ways to establish communication between PODs in a while. For now its important to understand how the internal networking works in kubernetes.

So it’s all easy and simple to understand when it comes to networking on a single node. But how does it work when you have multiple nodes in a cluster? In this case we have two nodes running kubernetes and they have IP addresses 192.168.1.2 and 192.168.1.3 assigned to them. Note that they are not part of the same cluster yet. Each of them has a single POD deployed. As discussed in the previous slide these pods are attached to an internal network and they have their own IP addresses assigned. HOWEVER, if you look at the network addresses, you can see that they are the same. The two networks have an address 10.244.0.0 and the PODs deployed have the same address too. This is NOT going to work well when the nodes are part of the same cluster. The PODs have the same IP addresses assigned to them and that will lead to IP conflicts in the network. Now that’s ONE problem. When a kubernetes cluster is SETUP, kubernetes does NOT automatically setup any kind of networking to handle these issues. As a matter of fact, kubernetes expects US to setup networking to meet certain fundamental requirements. Some of these are that all the containers or PODs in a kubernetes cluster MUST be able to communicate with one another without having to configure NAT. All nodes must be able to communicate with containers and all containers must be able to communicate with the nodes in the cluster. Kubernetes expects US to setup a networking solution that meets these criteria.

Fortunately, we don’t have to set it up ALL on our own as there are multiple pre-built solutions available. Some of them are the cisco ACI networks, Cilium, Big Cloud Fabric, Flannel, Vmware NSX-t and Calico. Depending on the platform you are deploying your Kubernetes cluster on you may use any of these solutions. For example, if you were setting up a kubernetes cluster from scratch on your own systems, you may use any of these solutions like Calico, Flannel etc. If you were deploying on a Vmware environment NSX-T may be a good option. If you look at the play-with-k8s labs they use WeaveNet. In our demos in the course we used Calico. Depending on your environment and after evaluating the Pros and Cons of each of these, you may chose the right networking solution.


## Kubernetes Services

Kubernetes Services enable communication between various components within and outside of the application. Kubernetes Services helps us connect applications together with other applications or users. For example, our application has groups of PODs running various sections, such as a group for serving front-end load to users, another group running back-end processes, and a third group connecting to an external data source. It is Services that enable connectivity between these groups of PODs. Services enable the front-end application to be made available to users, it helps communication between back-end and front-end PODs, and helps in establishing connectivity to an external data source. Thus services enable loose coupling between microservices in our application.

The kubernetes service is an object just like PODs, Replicaset or Deploymentsthat we worked with before. One of its use case is to listen to a port on the Node and forward requests on that port to a port on the POD running the web application. This type of service is known as a NodePort service because the service listens to a port on the Node and forwards requests to PODs. 

The three types of services are NodePort. ClusterIp and Loadbalancer. NodePort were the service makes an internal POD accessible on a Port on the Node. The second is ClusterIP – and in this case the service creates a virtual IP inside the cluster to enable communication between different services such as a set of front-end servers to a set of backendservers. The third type is a LoadBalancer, were it provisions a load balancer for our service in supported cloud providers.

![Screen Shot 2023-01-21 at 4 25 46 PM](https://user-images.githubusercontent.com/74343792/213887715-2e353de0-e2ae-4bad-b03e-b0de2bbc5ad1.png)

![Screen Shot 2023-01-21 at 4 26 09 PM](https://user-images.githubusercontent.com/74343792/213887719-5dc1756a-8a90-46a8-be01-169a395c5142.png)

















