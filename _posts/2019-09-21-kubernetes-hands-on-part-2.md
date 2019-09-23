---
layout: post
title: Kubernetes hands on - Part 2
date: 2019-09-21 11:00
category: [ DevOps ]
author: martin
tags: [ DevOps, Kubernetes, Microservices ]
---

In a previous article about Kubernetes (check it out [here](/kubernetes-hands-on-part-1) if you haven't read it yet), I introduced some of the basic building blocks and wrote about how to deploy a very basic webserver on K8S. In this article I'll explain a little bit about service discovery in Kubernetes.

## Kubernetes Service Discovery
Remember that from the [previous article](/kubernetes-hands-on-part-1), each service had its own ip address. These addresses were allocated dinamically, so it would not be a very good idea to reference them directly from another container. Like in Docker Swarm, Kubernetes offers a DNS service which allows you to reference another container from within a container using the name of the service instead of its ip.

This DNS service also runs on K8S, but it does not show up when doing a simple `kubectl get services`. This is because Kubernetes has namespaces. Namespaces allow grouping of K8S deployed things, to better isolate different systems of your application. This makes a lot of sense when we think of a microservices architecture, which might contain hundreds of different services running, each one having very specific uses. Think of front-end services like your UI and a gateway service, and then think of a group of microservices that deal only with authentication.

You can check which namespaces are currently available on Kubernetes typing `kubectl get namespaces`. We have so far been using the *default* namespace.

Returning to the example of DNS server: if we want to see the DNS service, which resolves our application names to actual IP addresses, we need to check what is running on the *kube-system* namespace. To do so, run `kubectl get all -n kube-system`. One of the entries that you'll find is the DNS service for Kubernetes:

```bash
[...]
service/kube-dns          ClusterIP   10.96.0.10     <none>      53/UDP,53/TCP   271d
[...]
```

We are going to do a simple test here: let's deploy a mysql database in one service and another service just with a mysql client. This Mysql deployment is **not** production ready; when its pod dies, all data will be lost. Remember, this is just an example. Effectively, we will have the following setup:

<div class="draw-io-embed">
  <div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;www.draw.io\&quot; modified=\&quot;2019-09-23T06:40:59.371Z\&quot; agent=\&quot;Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:69.0) Gecko/20100101 Firefox/69.0\&quot; etag=\&quot;yauHXByv_bcAiuRy4Lrt\&quot; version=\&quot;11.3.0\&quot; pages=\&quot;1\&quot;&gt;&lt;diagram id=\&quot;6hX2ZumDEAG7ygAPRqpY\&quot; name=\&quot;Page-1\&quot;&gt;7VhZbxshEP41fnS1yx5xHn3kkJqkqdwqSd/wMvai4MVl8dVfX9gF761ctqJKlayYGYYBPr454p43Xu6uBF7Ft5wA6yGH7HrepIeQ6yPU0x+H7HPNmX+WKxaCEmNUKKb0DxilY7RrSiCtGErOmaSrqjLiSQKRrOiwEHxbNZtzVt11hRfQUEwjzJraB0pknGsHgVPor4EuYruz65iZJbbGRpHGmPBtSeVd9Lyx4Fzmo+VuDEyDZ3HJ1112zB4OJiCRr1kQ/Lyc0XsxlYT5MHSuHm7Su755nQ1ma3PhHgqZ8jcidKMPLfcGifD3Wp90NOeJ7KfZOw2Vgeuv1FuPink1Wujv2/30+40yGDOqD6gPlzueiQ/5tW7URbMjVrUfPfU/ccj7b5OuDVDFMdrGVMJ0hSMtb1WEKqNYLpmSXDXE6SqPmTndATEnMDHo+geHZY4Z2m1ASNiVVIZzV8CXIMVemZhZ5Bv+mwTghkbeFuHk2nCKy6FkldiE8OLgu2C5Ghiiv4H0XoP0X9czEAlIlWmQM7mbqr9TEBuqcKtDKvg6IRqsiaMAegFggtM4s3Vr4AZHAtepgusNmuAirwXc8FTY+i0JpYYgJGSoM7OSIobTlEYKjFRiIZvqEpawo/JRw/4lMNKTeQQ9nuzKwt4KibpTaZEWn6w/LRTLMsmu63yalK9FBC9zS91mAfLlxAukUn6aD116yLYgsToBDEu6qRattsc1O9xzmiVly6NBlUf+eY0f+b3NqnKRqTnyUI2QqOYoB6bhKOPa4dqvop94eOz3nxHB4a/h9eBc/Pgpg76NoNPVM50XQPyvQuUqdKJznqSYNZJrS5x35tsg/Mxi1sr4tg7uIyVrThkbc8ZFttabz+coyhO04M9QmiHhLAzC4+B6yDcW17MmroM2WL1TwdrsEd4aN2EH2ydY4hlO4TjR2bXLoX15VWipl5NVGlSfO+EJ1LhhVJjRRaJLtnpplRa9keYBVf8/Dc3EkhKit2nlXZWZhAoVwZTrZVtIZWYgsdGcO7WYPhL3glrJUpoG9/wW7tUr29God7oWqt6BFu1RuTkq9Uod7ZFtxWxbVlrV2Yq19jvv67FQs8fqTo2f1lLVWvPAfWdLdTiQ7c3qvfu7WyolFj8/5ObFjzjexV8=&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
  <script type="text/javascript" src="https://www.draw.io/js/viewer.min.js"></script>
</div>

Use this definition to run your service discovery tests:

```yaml
# service-discovery.yaml
# first comes the database pod. We'll use a mysql
# instance
kind: Pod
apiVersion: v1
metadata:
  name: mysql-server
  labels:
    app: mysql-server
spec:
  containers:
  - name: mysql-server
    image: mysql:5
    env:
    # passing environment variables to the container
    - name: MYSQL_ROOT_PASSWORD
      value: pass
    - name: MYSQL_DATABASE
      value: test


---
# service definition for the database so
# that is is accessible by the mysql client pod
kind: Service
apiVersion: v1
metadata:
  # this is the name that is going to be referenced 
  # from our mysql client pod.
  name: database
spec: 
  selector:
    app: mysql-server
  ports:
  - port: 3306 # port of the running service
  # Should only be visible internally in K8S.
  type: ClusterIP

---
# mysql client pod - we will try to access
# our database using this pod
kind: Pod
apiVersion: v1
metadata:
  name: mysql-client
spec:
  containers:
  - name: mysql-client
    image: martinreus/mysql-client

```

Apply this to your cluster using `kubectl apply -f service-discovery.yaml`. Ensure that you have something like this running:

```bash
NAME                            READY   STATUS    RESTARTS   AGE
pod/mysql-client                1/1     Running   0          95s
pod/mysql-server                1/1     Running   0          95s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/database     ClusterIP   10.96.22.42      <none>        3306/TCP       5s
```

If everything went well, you can now check that you can access the database pod from the running mysql-client instance. To do so, you are going to start a shell session in your mysql-client pod with `kubectl exec -it pod/mysql-client sh`. Once you are in your pod, you can now issue a connect command to the database with the password and database name that was set in the yaml definition for the mysql-server pod, so in this case `mysql -h database -uroot -ppass test`. It should be able to connect to the server pod.

Another thing that is important to mention is the fact that *database* is not actually the [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) for the service. If you execute a `nslookup database` still inside your mysql-client pod, you'll see that it resolves *database* to the ip listed above (in this case 10.96.22.42) and the FQDN as *database.default.svc.cluster.local*. Notice that *default* in between the FQDN: this means that it was deployed in the default namespace. When connecting a microservice to another, it is generally a good idea to reference services using FQDN so as to avoid name clashes.

## Wrapping up
So this was it for service discovery in K8S, in a very small nutshell. Thanks for reading and stay tuned for more!