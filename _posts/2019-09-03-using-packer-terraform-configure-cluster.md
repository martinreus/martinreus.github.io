---
layout: post
title:  "Configure a Swarm Cluster Using Packer and Terraform"
author: martin
categories: [ DevOps ]
# image: assets/images/posts/moeda-real-low-wide.jpg
# image-credits: "Imagem: [Orlando Sant'Anna](https://unsplash.com/@ojsant?utm_medium=referral&utm_campaign=photographer-credit&utm_content=creditBadge)"
# featured: true
hidden: false
---
[sarcasm enabled] I love the term cloud computing [/sarcasm disabled]. Everything is in the cloud. The cloud is so easy to use. But what the general public doesn't see is how much work goes into configuring a cloud environment. Sure, you could already use a pre-configured Kubernetes environment on Google GCP, but where is the fun in that? 

In this post I will show you how to configure servers in an automated and configurable way using [Packer](https://www.packer.io/) and [Terraform](https://www.terraform.io/) on [Scaleway](https://www.scaleway.com), a cheap (well, cheaper than Amazon AWS and Google GCP) [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) and [IaaS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service) provider. As a bonus, I will show you how to configure Docker Swarm Cluster of several instances on top of this setup, so you can deploy your awesome application on it! Also be sure to follow the official [scaleway documentation](https://www.scaleway.com/en/docs/deploy-cloud-servers-with-packer-and-terraform/) on how to use these tools.

## Why Packer and Terraform?
Both Packer and Terraform have configuration files that describe exactly what will be installed or configured on your servers after these tools have run. This means that your infrastructure configuration can be stored as code and versioned in Git! Adding to that, Terraform is a well established standard for configuring servers, and Packer can be used to generate OS images for most of IaaS providers out there.

The basic difference between them two is that Packer will create base OS images which are common to all servers - for instance a Linux image which will have Docker installed. Terraform on the other hand will perform additional configuration specific to each machine; setting hostnames, a Docker label...

So for instance when you need an additional server in a Swarm cluster, you would run Terraform on top of an image already created with Packer. If you need a completely new image - say you want to configure a DB cluster with a different operating system - you would configure your base image with Packer, and then use Terraform to configure your machines in the cluster.

![Show me the code!](/assets/images/posts/talk-is-cheap-show-me-the-code-linus-torvalds.jpg)

## Creating a base image with Packer

