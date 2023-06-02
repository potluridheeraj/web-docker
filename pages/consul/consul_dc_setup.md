---
title: Consul Dc setup for server
keywords: consul
#summary: "DHEERAJ POTLURI"
sidebar: consul_sidebar
permalink: consul_dc_setup
folder: consul
---
{% include note.html content="Run this to setup DC4 in consul-server" %}
We are setting up as DC4 as a dc for this setup
```shell
sudo tee /etc/consul.d/consul-server.json <<EOF
{
  "datacenter": "dc4",
  "server": true,
  "bootstrap_expect": 1
}
EOF
```
{% include tip.html content="We highly discourage single-server production deployments." %}
> Ideal setup whould be 5 server setup so it will be helpfull during failover.   
> Bootstrap_expect should be changed based on number of servers.  

Restart consul server:

```shell
service consul restart

service consul status
```

Once you restart consul server you will see <span style="color:green">Active</span> as soon as you restart and see status
## Where to Next?

To setup vagrant for Consul, joining nodes into a cluster, and
interacting with the agent, check out: [Use consul UI](consul_ui).