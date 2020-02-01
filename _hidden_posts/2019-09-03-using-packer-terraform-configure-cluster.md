---
layout: post
title:  "Configure a Swarm Cluster Using Packer and Terraform"
author: martin
categories: [ DevOps ]
# image: assets/images/posts/moeda-real-low-wide.jpg
# image-credits: "Imagem: [Orlando Sant'Anna](https://unsplash.com/@ojsant?utm_medium=referral&utm_campaign=photographer-credit&utm_content=creditBadge)"
# featured: true
hidden: true
---
[sarcasm enabled] I love the term cloud computing [/sarcasm disabled]. Everything is in the cloud. The cloud is so easy to use. But what the general public doesn't see is how much work goes into configuring a cloud environment. Sure, you could already use a pre-configured Kubernetes environment on Google GCP, but where is the fun in that? 

In this post I will show you how to configure servers in an automated and configurable way using [Terraform](https://www.terraform.io/) on [Scaleway](https://www.scaleway.com), a cheap (well, cheaper than Amazon AWS and Google GCP) [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) and [IaaS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service) provider. As a bonus, I will show you how to configure Docker Swarm Cluster of several instances on top of this setup, so you can deploy your awesome application on it! 

<!-- Also be sure to follow the official [scaleway documentation](https://www.scaleway.com/en/docs/deploy-cloud-servers-with-packer-and-terraform/) on how to use these tools. -->

<!-- ## Why Packer and Terraform?
Both Packer and Terraform have configuration files that describe exactly what will be installed or configured on your servers after these tools have run. This means that your infrastructure configuration can be stored as code and versioned in Git!  -->

<!-- ##### Packer 
Packer will create base OS images for multiple cloud platforms from a single configuration file. You describe what you want your image to be - Windows with Docker, for instance - and Packer will create this image on Scaleway, Amazon AWS... -->

##### What is Terraform?
Terraform is a command line tool for provisioning servers on different cloud providers. When using Terraform, instead of having to manually provision each server in your PaaS provider's website, you can do this automatically from within the command line, which is repeatable, faster and less error prone. You can use the base image you created with Packer, provision it and also add specific configuration to each server.

[Follow this tutorial...](https://blog.ruanbekker.com/blog/2019/03/21/how-to-deploy-a-docker-swarm-cluster-on-scaleway-with-terraform/)

<!-- ##### Bottomline
If you need an additional server in a Swarm cluster, you would run Terraform using an image already created with Packer. If you need a completely new image - say you want to configure a DB cluster with a different operating system - you would configure your base image with Packer, and then use Terraform to provision your machines in the cluster. -->

![Show me the code!](/assets/images/posts/talk-is-cheap-show-me-the-code-linus-torvalds.jpg)

## Creating a base image with Packer

