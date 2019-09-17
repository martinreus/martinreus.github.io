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

So let's get started!

## So what's Kubernetes?
Kubernetes, also often written K8S, is a container orchestration engine. Think Docker Swarm on steroids! Like Docker Swarm, K8S has a master and zero or more worker nodes serving containers to the outside world. Kubernetes feels like an Onion to me though, if compared to Docker Swarm. There are many layers of "things" wrapping the containers before they can properly work on a production environment. Docker Swarm on the other hand feels more "bare-bones". I'll try to explain with an image:

[attach image]

That's a lot of colourful boxes. And every one of these can be defined as a yaml file which can then be applied onto a K8S cluster. I'll give some examples of yaml files in this article, but the [official documentation](https://kubernetes.io/docs/reference/) is very comprehensive and explains every parameter in detail. Just click on the version you will be currently using - in my case it was [V1.12](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/).

## First steps
Before starting playing with Kubernetes, we have to install it locally. Installing a production ready Kubernetes is definitely **NOT** an easy task. That's why minikube exists; think of it as a lightweight kubernetes engine that instantiates a single node where you can test your containers and configurations. Please follow the [tutorial](https://minikube.sigs.k8s.io/docs/start/) to install it on your machine. I'm installing it on linux.

After having installed minikube, you'll also have a tool called [kubectl](#kubectl), which is nothing more than a command line tool for interacting with your cluster. This is one of the things that got me a bit confused at the beginning; in Docker, every command would be issued using `docker ...`. In Kubernetes land you have this "separation", where your cluster is one thing, and your control tool is another one. Makes a lot of sense, though :)

Now let's see how to launch our first service.

### Pods
Pods are the most basic building blocks for running containers in Kubernetes. An example of a pod can be defined in a yaml file, like so:

```yaml 
# pod.yml
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

Issuing the command `kubectl apply -f pod.yml` will apply this pod definition on a K8S cluster. After the container started you can check that it indeed exists by executing `kubectl get pods --all`. You should see the running container.

When I first saw the Pod definition, I thought 

> oh, this looks like the service definition in Docker Swarm

 It turns out that this is not quite the case. A pod does not expose the service running in the container to the outside world. For this we will have to use [Services](#services). A major difference here also is that after this container is set to run and for whatever reason crashes, Kubernetes will not respawn it on another node. For the container to be automatically recreated, we will have to use [Replica Sets](#replica-sets).

 The most important thing a pod yaml defines is the Docker container it will run. That's right: Kubernetes runs Docker containers per default. But it could also run containers using [RKT](https://coreos.com/rkt/docs/latest/using-rkt-with-kubernetes.html).

### <a name="services"></a>Services 


### <a name="replica-sets"></a>Replica Sets







## Tools and how to spin up Kubernetes



### <a name="kubectl"></a>Kubectl 