---
title: Kubernetes for Docker Users
date: 2019-09-16 07:56
category: [ DevOps ]
author: martin
# tags: []
# summary: 
hidden: true
---
As a moderately frequent user of Docker Swarm, I wanted to up my game a little bit by using Kubernetes, since it appeared to offer some additional features which were not present in Docker Swarms - Stateful Sets, for instance. So I started reading the official docs and got really overwhelmed by the amount of new concepts and features described there.

In this article I will try to summarize the differences and similarities between Docker and Kubernetes for someone fairly knowleadgeable in Docker Swarm.

So let's get started!

## So what's Kubernetes?
Kubernetes, also often written K8S, is an orchestration engine. Think Docker Swarm on steroids! Analogous to Docker Swarm, K8S also will have a master node and multiple worker nodes serving containers to the outside world. Kubernetes feels like an Onion to me though, if compared to Docker Swarm. There are so many layers of "things" wrapping my container. Docker Swarm on the other hand feels more "bare-bones". I'll try to explain with an image:

[attach image]
