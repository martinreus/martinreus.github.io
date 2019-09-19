---
title: Kubernetes for Docker Users
date: 2019-09-16 07:56
category: [ DevOps ]
author: martin
# tags: []
# summary: 
hidden: true
---
As a moderately frequent user of Docker Swarm, I wanted to up my game a little bit by using Kubernetes, since it appeared to offer some additional features which were not present in Docker Swarm - Stateful Sets, for instance. So I started reading the official docs and got really overwhelmed by the amount of new concepts and features described there.

In this article I will try to summarize the differences and similarities between Docker and Kubernetes for someone fairly knowleadgeable in Docker Swarm, as well as trying to explain very supperficially some of the functionalities Kubernetes offers. The official docs are really good though, so be sure to check them out [here](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)!

Let's get started!

## So what's Kubernetes?
Kubernetes, also often written K8S, is a container orchestration engine. Think Docker Swarm on steroids! Like Docker Swarm, K8S has a master and zero or more worker nodes serving containers to the outside world. Kubernetes feels like an Onion to me though, if compared to Docker Swarm. There are many layers of "things" wrapping the containers before they can properly work on a production environment. Docker Swarm on the other hand feels more "bare-bones". I'll try to explain with an image:

[attach image]

That's a lot of colourful boxes. And every one of these can be defined as a yaml file which can then be applied onto a K8S cluster. I'll give some examples of yaml files in this article, but the [official documentation](https://kubernetes.io/docs/reference/) is very comprehensive and explains every parameter in detail. Just click on the version you will be currently using - in my case it was [V1.12](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/).

## First steps
Before starting playing with Kubernetes, we have to install it locally. Installing a production ready Kubernetes is definitely **NOT** an easy task. That's why minikube exists; think of it as a lightweight kubernetes engine that instantiates a single node where you can test your containers and configurations. Please follow the [tutorial](https://kubernetes.io/docs/tasks/tools/install-minikube/) to install it on your machine. I'm installing it on linux.

After having installed minikube, you'll also have a tool called [kubectl](#kubectl), which is nothing more than a command line tool for interacting with your cluster. This was one thing that got me a bit confused at the beginning; in Docker, every command would be issued using `docker [OPTIONS]`. In Kubernetes land you have this "separation", where your cluster is one thing, and your control tool is another one. Makes a lot of sense, though :)

Now let's see how to launch our first service.

### Pods
A Pod is the most basic building block for running containers in Kubernetes. An example of a pod can be defined in a yaml file, like so:

```yaml 
# pod.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: webserver
  labels:
    app: webserver
spec:
  containers:
  - name: webserver
    image: nginxdemos/hello
```

Issuing the command `kubectl apply -f pod.yaml` will apply this pod definition on a K8S cluster. After the container started you can check that it indeed exists by executing `kubectl get pods --all`. You should see the running container.

 The most important thing a pod yaml defines is the Docker container it will run. That's right: Kubernetes runs Docker containers per default. But it could also run containers using [RKT](https://coreos.com/rkt/docs/latest/using-rkt-with-kubernetes.html).


When I first saw the Pod definition, I thought 

> ooh, this looks like the service definition in Docker Swarm

 It turns out that this is not quite the case. A pod does not expose the service running in the container to the outside world. For this we will have to use [Services](#services). An important distinction here also is that after this container is set to run and for whatever reason crashes, Kubernetes will not respawn it on another node. For the container to be automatically recreated, we will have to use [Replica Sets](#replica-sets).

### <a name="services"></a>Services 
Services enable us to expose a service out to the world. Very simply, a service will have a fixed ip which will point to one or more pods (more on that in [Replica Sets](#replica-sets)).

A service "connects" to pods using selectors. Looking at the pod definition yaml, you can see in the metadata that it has a labels tag - and there could be many of them - which are used in the service definition.

Again, lets define a yaml file for a service which will connect to our pod.

```yaml
# service.yaml
kind: Service
apiVersion: v1
metadata:
  # can be the same, must not be, however
  name: webserver
spec: 
  selector:
    # this HAS to have exactly the same value of one (or more) 
    # of the labels defined in the pod.
    app: webserver
  # defines which port(s) we are going to expose
  ports:
    - name: http
      port: 80 # port of the running service
      nodePort: 30080 # outside world port. A limitation of 
                      # NodePorts is that the number of the
                      # port has to be greater than 30000.

  # the type of service defines to where we want to expose 
  # type: ClusterIp # exposes the service only internally for kubernetes cluster
  type: NodePort # exposes service port to the outside world
  # type: LoadBallancer # exposes service using a LoadBallancer. 
                        # Not supported on clusters that do not
                        # have an integrated load ballancer 
                        # (such as minikube). 
```

A quick note on the NodePort type: when a Load Ballancer is not available for the kubernetes cluster, and the service still needs to be deployed with a port 80 - very common for websites :) - a quite common approach to create an [Ingress Service](ingress-service). For testing purposes, port 30080 will do the trick.

Now if you execute `kubectl apply -f service.yaml`, you should see a new entry in your cluster when running `kubectl get all`. To access the website, you'll need the ip of the server, which is the ip of your kubernetes cluster - in this case, the ip of minikube. Run `minikube ip` in you console, and then access the example website on your browser using [IP ADDRESS]:30080. You should see an nginx demo page.

### <a name="replica-sets"></a>Replica Sets
A Backend application will eventually crash and so will the pod. No shame here, everyone makes mistakes. But a K8S node might also fail, and so the pod will be lost as well. One could imagine the pod would be automatically recreated, like it would if we were deploying it on Swarm. For kubernetes though, this behaviour needs to be defined using Replica Sets.

Replica Sets allow us to do define how many instances of a pod must be running at any given time in Kubernetes world, even in the event of pod failures or cluster node crashes.

Let's then define a replica set for the nginx webserver. Just a note here first: when writing a replica set file, you **don't need to write two separate files** (one for the pod and one for the replica set); you may just nest the pod definition under the `spec.template` tag of the yaml file. Have a look at it:

```yaml
# replica-sets.yaml
# 
# instead of v1 we now have apps/v1 because replica sets 
# are not part of the core kubernetes library
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  # can again be webserver because when 'kubectl get all'
  # is used it will appear under replicaset.apps/webserver
  name: webserver
spec:
  # seems redundant having a selector in a replica set where 
  # we nest a pod definition. But I guess they wanted to keep
  # consistency if we were to separate pod.yaml and 
  # replica-set.yaml files 
  selector:
    matchLabels:
      app: webserver
  # define how many instances we want here
  replicas: 3
  # pod template below
  template:
  ############ pod definition copied from example above ##########
    metadata:
      # name not needed anymore
      labels:
        # still needed because this is the selector for the 
        # service definition
        app: webserver
    spec:
      containers:
      - name: webserver
        image: nginxdemos/hello
  ################################################################
```

If you run `kubectl apply -f replica-sets.yaml` and then `kubectl get all`, you should not only see that replicaset.apps/webserver is now listed but also that more than one pod/webapp-[HASH] instances also are running. Try deleting one of the pods by running `kubectl delete pod/webserver-[HASH]` (of course substituting [HASH] with one of the values that appear to you). You can confirm with `kubectl get pods` that one of the pods is younger than the others (check out the AGE column of your terminal's output).

### <a name="ingress-service"></a>Ingress Service




## Tools and how to spin up Kubernetes



### <a name="kubectl"></a>Kubectl 