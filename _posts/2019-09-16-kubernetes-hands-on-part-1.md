---
title: Kubernetes hands on - Part 1
date: 2019-09-16 07:56
category: [DevOps]
author: martin
# tags: []
# summary:
image: assets/images/posts/kubernetes.png
featured: true
hidden: true
---

In this series of articles I will try to summarize some of the functionalities Kubernetes offers. The official docs are really good though, so be sure to check them out [here](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)! Where possible I will also try to bridge concepts between Docker and Kubernetes.

Let's get started!

## So what's Kubernetes?

Kubernetes, also often written K8S, is a container orchestration engine. Think Docker Swarm on steroids! Like Docker Swarm, K8S has a master and zero or more worker nodes serving containers to the outside world. Kubernetes feels like an Onion to me though, if compared to Docker Swarm. There are many layers of "things" wrapping the containers before they can properly work on a production environment. Docker Swarm on the other hand feels more "bare-bones". I'll try to explain with an image:

<div class="draw-io-embed">
  <div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile modified=\&quot;2019-09-21T11:44:57.631Z\&quot; host=\&quot;www.draw.io\&quot; agent=\&quot;Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:69.0) Gecko/20100101 Firefox/69.0\&quot; etag=\&quot;f5vJ8aI5GATuqvCg-bOz\&quot; version=\&quot;11.3.0\&quot; type=\&quot;device\&quot; pages=\&quot;1\&quot;&gt;&lt;diagram id=\&quot;TVPadQyQwi-CoDUWOwtm\&quot; name=\&quot;Page-1\&quot;&gt;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&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
  <script type="text/javascript" src="https://www.draw.io/js/viewer.min.js"></script>
</div>

That's a lot of boxes. And every one of these can be defined as a yaml file - except for the kubelet part, which is the K8S agent running on every node - which can then be applied onto a K8S cluster. I'll give some examples of yaml files in this article, but the [official documentation](https://kubernetes.io/docs/reference/) is very comprehensive and explains every parameter in detail. Just click on the version you will be currently using - in my case it was [V1.12](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/).

#### First steps

Before starting playing with Kubernetes, we have to install it locally. Installing a production ready Kubernetes is definitely [**NOT**](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) [an easy](https://stefanprodan.com/2018/kubernetes-scaleway-baremetal-arm-terraform-installer/) [task](https://medium.com/@salqadri/build-your-own-multi-node-kubernetes-cluster-with-monitoring-346a7e2ef6e2). That's why minikube exists; think of it as a lightweight kubernetes engine that instantiates a single node on your local computer, where you can test your containers and configurations. Please follow the [tutorial](https://kubernetes.io/docs/tasks/tools/install-minikube/) to install it locally.

After having followed the minikube installation guide, you should also have a tool called [kubectl](#kubectl), which is the command line tool for interacting with your cluster. Now let's see how to launch our first application.

#### Pods

A Pod is the most basic building block for running containers in Kubernetes. It's a wrapper around a runnable container. An example of a pod can be defined in a yaml file, like so:

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

Issuing the command `kubectl apply -f pod.yaml` will apply this pod definition on a K8S cluster. After the container started, you can check that it indeed exists by executing `kubectl get pods --all`. You should see the running container.

The most important thing a pod yaml defines is the Docker container it will run. That's right: Kubernetes runs Docker containers per default. But it could also run containers using [RKT](https://coreos.com/rkt/docs/latest/using-rkt-with-kubernetes.html).

When I first saw the Pod definition, I thought

> ooh, this looks like the service definition in Docker Swarm

It turns out that this is not quite the case. A pod alone cannot expose a port of a container to the outside world, or even to another container inside the cluster. For this we will have to use [Services](#services). An important distinction here also is that after this container is set to run and for whatever reason crashes, Kubernetes will not respawn it on another node. For the container to be automatically recreated, we will have to use [Replica Sets](#replica-sets).

#### <a name="services"></a>Services

Services enable us to expose a container port out to the world or internally to other containers inside K8S, so that pods can communicate with one another. For those who have already worked with Docker Swarm, explicitly exposing a port of a service inside the cluster is not needed as it is already the default behaviour.

Very simply put, a service will have a fixed ip which will point to one or more pods (more on that in [Replica Sets](#replica-sets)). For a service to know where to route connections to, it uses selectors. Looking at the pod definition yaml we created in the last section, you can see in the metadata that it has a labels tag - and there could be many of these labels - which are used in the service definition.

Again, lets define a yaml file for a service which will connect to our pod.

```yaml
# service.yaml
kind: Service
apiVersion: v1
metadata:
  # can be the same value as above or any other thing
  name: webserver
spec:
  selector:
    # this NEEDS to be exactly the same value of one (or more)
    # of the labels defined in the pod.
    app: webserver
  # defines which port(s) we are going to expose
  ports:
    - name: http
      port: 80 # port of the running service
      nodePort:
        30080 # outside world port. A limitation of
        # NodePorts is that the number of the
        # port has to be greater than 30000 so as
        # to not collide with ports used by K8S.

  # the type of service defines to where we want to expose our ports:
  # type: ClusterIp # exposes the service only internally for
  # kubernetes cluster
  type: NodePort # exposes service port to the outside world
  # type: LoadBallancer # exposes service using a LoadBallancer.
  # Not supported on clusters that do not
  # have an integrated load ballancer
  # (such as minikube).
```

A quick note on the NodePort type: when a Load Ballancer is not available for the kubernetes cluster, and the service still needs to be deployed with a port 80 - very common for websites :) - a quite common approach is to create an Ingress Service. I will talk a bit more about this in a future post. For testing purposes, port 30080 will do the trick.

Now if you execute `kubectl apply -f service.yaml`, you should see a new entry in your cluster when running `kubectl get all`. To access the website, you'll need the ip of the server, which is the ip of your kubernetes cluster - in this case, the ip of minikube. Run `minikube ip` in you console, and then access the example website on your browser using [IP ADDRESS]:30080. You should see an nginx demo page.

#### <a name="replica-sets"></a>Replica Sets

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

If you run `kubectl apply -f replica-sets.yaml` and then `kubectl get all`, you should not only see that replicaset.apps/webserver is now listed but also that more than one pod/webapp-[uniqueIdentifier] instances also are running. Try deleting one of the pods by running `kubectl delete pod/webserver-[uniqueIdentifier]` (of course substituting [uniqueIdentifier] with one of the values that appear to you). You can confirm with `kubectl get pods` that one of the pods is younger than the others (check out the AGE column of your terminal's output).

What if we change the version of the nginxdemos/hello container in the replica set definition? Kubernetes is going to recreate all the webserver containers, and there could be a possible downtime for the site, which is undesirable in production environments. A better approach is to use [Deployments](#deployments) instead.

#### <a name="deployments"></a>Deployments

Deployments, very simply put, are a more powerful version of a service. Deployments allow us to do rolling updates as well as rollbacks in case they fail! The yaml file is almost identical to the definition of a replica set, except that the tag _kind_ is now _Deployment_.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
spec:
  # important for rolling updates; here pods will need to
  # be up for 30 seconds without crashing for the rolling
  # update to be considered successful. Default is 0!
  minReadySeconds: 30
  selector:
    matchLabels:
      app: webserver
  replicas: 3
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
        - name: webserver
          image: "nginxdemos/hello:0.2-plain-text" # we now use a fixed version
```

Before applying this definition, a good thing to do is to clear things in our cluster first. You can delete your old replica set by running `kubectl delete replicasets.apps/webserver` or simply `kubectl delete rs webserver` (rs stands for replicaset in its short form). Leave the service running though, because it is still needed.

Ok, so now apply your deployment with `kubectl apply -f deployment.yaml` and check that it was created with `kubectl get all`. Looking at the terminal, you should see something like this:

```bash
NAME                            READY   STATUS    RESTARTS   AGE
pod/webserver-b59854878-2hz9s   1/1     Running   0          28m
pod/webserver-b59854878-btq6j   1/1     Running   0          28m
pod/webserver-b59854878-qvl66   1/1     Running   0          28m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        269d
service/webserver    NodePort    10.100.236.132   <none>        80:30080/TCP   7m46s

NAME                        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webserver   3         3         3            3           28m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/webserver-b59854878   3         3         3       28m
```

When looking at the names of every type of resource created, we understand how things are created in chain when we apply the definition of our deployment. A deployment is created with name _deployment.apps/webserver_, which in turn spawns a new _replicaset.apps/webserver-b59854878_ replica set for us, which results in 3 pods being created after that.

The names of the pods, as with all resource types in kubernetes, must be unique. Each pod has a prefix _pod/webserver-_ followed by an identifier created for the replica set (in our case _b59854878_) followed by another identifier, which in the end looks like this: _pod/webserver-b59854878-2hz9s_.

For the updating a deployment, or in other words, to perform a rolling update, we can change the container image to a new version,

```yaml
image: "nginxdemos/hello:0.2"
```

and then apply the deployment again. You should see that in the first 90 seconds, the webserver presented to us will still be the old one - try that in your browser. That's because each pod will be created individually one at a time and must be stable for at least 30 seconds; we have 3 pods, so 90 seconds in total. After that, kubernetes will scale down the old replica set, which will still be available for issuing [rollbacks](#deployment-rollback) though.

To check the status of the deployment update, use the command `kubectl rollout status deployment.apps/webserver`. You can check the history of rollouts (up to ten) using `kubectl rollout history deployment.apps/webserver`.

###### <a name="deployment-rollback"></a>Deployment rollback

Reverting a deployment is also possible, even though it should be used as a last resource only; rolling back will put your cluster in a state different from what your yaml file defines!

Even so, imagine a new rollout creates a catastrophic failure on your website and renders it completely useless to your customer. Until you debug your app and figure out what's going on, a rollback could pottentially give you time to solve the problem while still guaranteeing delivery of service of its older version. An example of a rollout for this article can be done executing `kubectl rollout undo deployment.apps/webserver`.

## Wrapping up

With [pods](#pods), [services](#services), [replica sets](#replica-sets) and [deployments](#deployments), we can now create a simple service which can be updated and rolled back. Pretty neat! But there is more to come; Kubernetes has just so many things to cover! Hope you enjoyed it, stay tuned :)
