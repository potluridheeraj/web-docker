---
title: Setup vagrant for consul
keywords: consul,install, vagrant
summary: "Installing vagrant"
sidebar: consul_sidebar
permalink: vagrant_consul
folder: consul
---

## Setting up vagrant
This demo provides a very simple `Vagrantfile` that creates Consul server nodes

> To install vagrant please go to [Vagrant](https://www.vagrantup.com/downloads.html)
{% include tip.html content="To follow through the whole demo you can you use below vagrant file" %}

Below `Vagrantfile` creates:  
   
1 consul server  
1 lb server  
4 web servers  

{% include warning.html content="We highly discourage single-server production deployments.  " %}

```shell

Vagrant.configure("2") do |config|
  config.vm.box = "suse/sles12sp2"
  config.vm.box_version = "0.0.1"

  config.vm.define "consul-server" do |cs|
    cs.vm.hostname = "consul_server"
    cs.vm.network "private_network", ip: "172.60.60.6"
  end

  config.vm.define "lb" do |lb|
    lb.vm.hostname = "lb"
    lb.vm.network "private_network", ip: "172.60.60.11"
  end

  (1..4).each do |i|
    config.vm.define "web#{i}" do |web|
      web.vm.hostname = "web#{i}"
      web.vm.network "private_network", ip: "172.60.60.2#{i}"
    end
  end

end

```

## Vagrant Consul Demo
To get started, you can start the nodes by just doing:

```shell
vagrant up
```

{% include warning.html content="If you prefer a different Vagrant box, you can set the `config.vm.box` " %}

Once it is finished, you should be able to see the following:

```shell
vagrant status # will show status of each server 
```
```
Current machine states:
consul-server                         running (virtualbox)
lb                                    running (virtualbox)
web 1 through 4                       running (virtualbox)     
```

At this point the 6 nodes are running and you can SSH in to play with them:

```shell
vagrant ssh consul-server  #will let you login to vagrant machine
```


>consul version   
>Consul v0.7.5   
>Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)   
>exit   

and

```shell
vagrant ssh lb
```

>consul version  
>Consul v0.7.5  
>Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)  
>exit  


## Where to Next?

To learn more about starting Consul, joining nodes into a cluster, and
interacting with the agent, check out: [Consul Pre-requisites](consul_prerequisites).

If want to skip understanding what script does, you can jump directly to install consul with a script
{% include note.html content="Single file scrip can be found [here](consul_install) " %}