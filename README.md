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

To get the version of kubenetes run

```
kubectl version --client --output=yaml
```

To check the flavor and version of operating system on which the Kubernetes nodes are running, Run

```
kubectl get nodes -o wide
```

Install kubectl and minikube on your laptop. (https://kubernetes.io/docs/tasks/tools)

<img width="885" alt="Screen Shot 2022-11-23 at 8 42 38 PM" src="https://user-images.githubusercontent.com/74343792/203675480-27a07556-5c29-4db5-bcc7-345c7a0fe60c.png">

<img width="893" alt="Screen Shot 2022-11-23 at 8 43 14 PM" src="https://user-images.githubusercontent.com/74343792/203675488-a4389c98-11d9-4105-96aa-4fac785579cb.png">

Test using a deployment to confirm everything is fine (All the commands are available in the link above. you will see it after insttalling minikube)

<img width="762" alt="Screen Shot 2022-11-23 at 8 55 44 PM" src="https://user-images.githubusercontent.com/74343792/203676856-73307d20-d603-4694-8a50-a11821d14a7d.png">

<img width="901" alt="Screen Shot 2022-11-23 at 8 51 50 PM" src="https://user-images.githubusercontent.com/74343792/203676479-5a633adc-a71b-44c3-8082-b8ee30101357.png">








