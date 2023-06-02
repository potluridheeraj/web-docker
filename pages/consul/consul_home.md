---
title: Consul  
keywords: consul
#summary: "DHEERAJ POTLURI"
sidebar: consul_sidebar
permalink: consul
folder: consul
---

## Service Mesh Made Easy
A distributed networking layer to connect, secure and observe services across any runtime platform and public or private cloud.

Consul Service Mesh Architecture:
 Consul has a client-server architecture and is the “control plane” for the service mesh. Multiple servers are deployed for high availability, and a pool of clients run on every host. Clients integrate with sidecar proxies, such as Envoy, that provide the “data plane” for the service mesh.

The centralized servers hold the service registry, access and traffic policies, configurations and certificate authorities, which are efficiently transferred to the distributed clients in real time. The clients configure local proxies, cache data and policies, and provide health checking.

## Service Discovery Made Easy

Service registry, integrated health checks, and DNS and HTTP interfaces enable any service to discover and be discovered by other services

- Service discovery for connectivity to remove manual processes
- Service discovery for dynamic infrastructure.
- Improve productivity by reducing lead time of connecting services from weeks to seconds without operator intervention.
- Reduce cost by eliminating the need for east-west load balancers to connect services
- Reduce risk by lowering the probability of downtime introduced when managing and updating load balancers

## Getting Started

 pre-requisites:
 - [Vm ware](https://www.vmware.com/) or [Virtualbox](https://www.virtualbox.org/) 
 - [Vagrant](https://www.vagrantup.com/docs/installation/) 
 
## Where to Next?

To setup vagrant for Consul, joining nodes into a cluster, and
interacting with the agent, check out: [Setting up Dc for consul](vagrant_consul).