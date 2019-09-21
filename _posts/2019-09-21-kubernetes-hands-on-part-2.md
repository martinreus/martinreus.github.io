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
Remember that from the previous article each service has its own ip address. These addresses are allocated dinamically, so it should not be a very good idea to reference it directly from another container. Like it is done in docker swarm, Kubernetes offers a DNS service, which allows you to reference another container from within a container using the name of the service instead of its ip.

This DNS service is normally hidden when doing a simple `kubectl get services`. This is because Kubernetes has namespaces. Namespaces allow grouping of logical groups if pods to better isolate different systems of your application. This makes a lot of sense when we think of a microservices architecture, which might contain hundreds of different services running, each one having very specific uses. Think of front-end services, like your UI, a gateway, a firewall. You can check which namespaces are currently available on Kubernetes typing `kubectl get namespaces`. We have so far been using the default namesystem.

Returning to the example of DNS server, if we want to see the DNS service, which resolves our application names to actual IP addresses, we need to check what is running on the kube-system namespace. To do so, run `kubectl get all -n kube-system`. One of the entries that you'll find is the DNS server for Kubernetes:

```bash
[...]
service/kube-dns          ClusterIP   10.96.0.10     <none>      53/UDP,53/TCP   271d
[...]
```

We are going to do a simple test here: let's deploy a mysql database in one service and another service just with a mysql client. This Mysql deployment is **not** production ready; when its pod dies, all data will be lost. Remember, this is just an example. Effectively, we will have the following setup:

<div class="draw-io-embed">
    <div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;www.draw.io\&quot; modified=\&quot;2019-09-21T11:53:09.833Z\&quot; agent=\&quot;Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:69.0) Gecko/20100101 Firefox/69.0\&quot; etag=\&quot;MwQ0Z2w7KD1kHg_al_0_\&quot; version=\&quot;11.3.0\&quot; type=\&quot;device\&quot; pages=\&quot;1\&quot;&gt;&lt;diagram id=\&quot;6hX2ZumDEAG7ygAPRqpY\&quot; name=\&quot;Page-1\&quot;&gt;zVZRb5swEP41PG4CA2n62CRtJ22tKqGp66ODr2DJwZlxErJfPzsYMJgobbSslXi4+7g7+777dOCF81V1L/A6f+AEmId8UnnhwkMoiBDy9OOTfY1cRVc1kAlKTFAHJPQPGNA36IYSKHuBknMm6boPprwoIJU9DAvBd/2wV876p65xBg6QpJi56DMlMq/Raex3+DegWd6cHPjmzQo3wQYoc0z4zoLCWy+cC85lba2qOTBNXsNLnXd35G17MQGFfEtC/PNuSZ9EIgmL4Ma/f/5RPn4xVbaYbUzDD/vy92GEWOIlLkGZCYgtTcG0IfcNN7ucSkjWONX+Ts3fC2e5XDHlBcrE5bqeyCutQF1iZs4CIaE62kTQUqM0BXwFUuxViEmIpoZNI6fg2vi7bjhBZLDcHkwDYiOIrK3dcaYMQ9s7KEQOhR6aMHXsjNCtMjNtKlZTXSllVHdqAtR5VsxIWsv8ePzHzwNNPt08Qmce3zdLEAVItUeQv3hMNGuKWBAOg4JvCqK5WfiKnxN8Elzmh9jgH3EZxD0uw6nLJQpHuJxcispoRNoDxqAgN3rNKi9luCxpqsgoJRbShS3uoKLyl6b5a2y8F0O6theV7ewbp1A9WUnafWnqaadLO3hNXn1nIM6iHwxG9cU3IoXT2lLtZSBP7QR30NYg45E5NpgAhiXd9q87NlxzwhOneqe0OhrsyOh6oI+6TZNlfzEGhULUL9T6TaGaB6fQQWtt2+fLL764/AJbfK0UT8nPFp+lxcvLD71Rfkf2zP+RX+QPVBOcKb8o7hdy9tzZ8lNu999Vh3d/r+HtXw==&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
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
  name: database
  labels:
    app: database
spec:
  containers:
  - name: mysql
    image: mysql:5
    env:
    # passing environment variables to the container
    - name: MYSQL_ROOT_PASSWORD
      value: pass
    - name: MYSQL_DATABASE
      value: test


--- # these "---" serve as a separator in yaml files for K8S

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
    app: database
  ports:
  - port: 3306 # port of the running service
  # Should only be visible internally in K8S.
  type: ClusterIp

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