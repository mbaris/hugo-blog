---
title: "Infrastructure as Code"
date: 2021-04-25T00:02:25+02:00
draft: true
tags: ["devops", "infrastructure"]
description: "Infrastructure as code is the practice of managing and provisioning resources through version controller definition files"
---

Problem?

Infrastructure as code definition
	Infrastructure as code is the practice of managing and provisioning resources through version controller definition files

Few things to note before:
	Cloud providers ike AWS, Azure and GCP allowed us to spin up servers in seconds. 
	We still need a way to create new servers when needed and manage an inventory of them somehow
	This topic is not necessarily coupled with this topic but cattle/pet can help when we use VMs from a cloud provider
	Cloud providers does not only provide servers and hosting. They serve many building blocks we can use to create an app
	Networking components, databases, cache servers, permission models etc.
	We want to automate as much as we can


Consider a service with 3 regions(us,eu,ap) and 3 environments(dev,staging,prod):
Initial imperative solution: Bash scripts to create and kill servers? 

Shortcomings of scripts like this after the initial setup
	Even for initial setup, terraform and cloudformation would provide easier config options
	And for later, every change would mean thinking about api calls whereas in terraform it is probably just a variable in a resource


Solution?

Keeping all configuration related to our systems in the code
Because we have good best practices around git(peer reviews, versioning, diffs etc.)
Tie infrastructure updates to merge requests with pull requests
		Accountability
		Consistently
		Faster


Action?
use terraform