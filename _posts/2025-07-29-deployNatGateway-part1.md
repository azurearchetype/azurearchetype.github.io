---
layout: post
title: "deployNatGateway"
subtitle: "Part 1 - theInception"
date: 2025-07-29
author: Matthew
---

## Why are we here?

In my first series as inspiring blogger I wanted to write about something that would be a helpful solution to a current issue. So I chose the deployment of an Azure NAT Gateway to provide outbound public Internet traffic for virtual machine resources deployed in Azure.
Microsoft will retire the default outbound internet access for new Azure virtual machines (VMs) on September 30, 2025. After this date, newly created VMs will require explicit outbound connectivity methods, my recommendation being a NAT gateway.  I will describe why a NAT gateway is a cost-effective and secure method for providing Internet access for compute workloads when a Firewall solution is most likely cost prohibitive.

This project had the following objectives:
1. Deployment of a NAT Gateway resource by IaC (Infrastructure as Code)
2. Explore the possibilities and limitations of customUIdefinition schemas for a user interface when deploying a custom template.
3. Leverage a public GitHub repository and use the ‘Deploy to Azure’ button functionality to create the NAT Gateway in the user’s tenant.
This last one is a bucket list item that I just wanted to understand how it all came together.

## What’s the Deal with a NAT Gateway anyway?

Simply put, an Azure NAT Gateway is a fully managed service that provides outbound internet connectivity for virtual machines (VMs) in a virtual network (VNet). It performs Source Network Address Translation (SNAT), mapping private IP addresses of VMs to one or more static public IP addresses for outbound traffic.
The function of the gateway is to enable VMs in a subnet to access the internet without exposing them to inbound connections, ensuring secure and scalable outbound connectivity. It simplifies IP management, supports high-volume traffic, and provides consistent public IPs for whitelisting by external services.
A NAT Gateway could be preferred over a firewall for outbound internet access in these scenarios:

1. Simple Outbound Access: Use for VMs needing internet for updates or APIs without complex firewall rules.
2. Scalability: Handles high outbound traffic (e.g., web apps) with up to 64,000 SNAT ports.
3. Cost-Effective: Cheaper than Azure Firewall for basic outbound needs, avoiding advanced security costs.
4. Fixed IP for Whitelisting: Provides static public IPs for external services requiring predictable IPs.
5. Outbound-Only Needs: Ideal when VMs don’t need inbound connections, ensuring simplicity.
6. Low Maintenance: No firewall rule management, perfect for dev/test or non-security-critical workloads.
Use of a firewall instead would be more appropriate for advanced security, inbound traffic control, or compliance needs.

And I don’t think it is necessary to go into why assigning a public IP directly to a VM is a bad idea, maybe that could be a future topic for a rant.

On my upcoming posts, we’ll cover the process building the main template, my shattered expectations about creating a really cool Azure UI, and how cool is GitHub anyway.  Lastly, I will have some thoughts as to why this is probably better server up as an ‘agent’ in today’s world.

## Let’s Do This

With all that, the deployNatGateway template is live at the following respository:

[deployNatGateway](https://github.com/azurearchetype/deployNatGateway)

Check it out. It’s in alpha, but I wanted to get it out as there has been some traffic on the social medias regarding the Azure NAT Gateway with the announcement of the availability of Private Subnets in Azure and the September 30th deadline.

I am hoping to have GitHub discussions online here soon for feedback and comments, but in the meantime reach out and let’s do something collaborative.

## Pro Tip

If you have an AVD host pool configured with ‘autoscale’ enabled, creation of a new VMs to meet resource needs will be effected and not have public access to the Internet by default.  Outbound traffic to the Internet must be explicitly defined. Deployment of a NAT Gateway just might be the ticket to avoid a bad day and most likely will improve your user experience.
