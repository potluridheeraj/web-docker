---
title: Consul Dc setup for server
keywords: consul
#summary: "DHEERAJ POTLURI"
sidebar: consul_sidebar
permalink: consul_ui
folder: consul
---
{% include note.html content="Get ip address from /etc/consul.d/consul-acl.json" %}   

Through previous installed script you have already enabled "UI" using -> "ui": true

Now use usr http://172.60.60.6:8500 (this is initially setup at [vagrant](vagrant_consul) setup )       
          
        
       
{% include inline_image.html file="consul_servers.png" alt="SDK button" %} 


## Where to Next?

To setup vagrant for Consul, joining nodes into a cluster, and
interacting with the agent, check out: [Setting up Dc for consul](consul_dc_setup).