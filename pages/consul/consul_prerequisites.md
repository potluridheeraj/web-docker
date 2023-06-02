---
title: Consul prerequisites
keywords: consul,install, consul install
summary: "Installing consul in your machine"
sidebar: consul_sidebar
permalink: consul_prerequisites
folder: consul
---
## Decide on which version of consul need to be installed 
{% include note.html content="I am installing consul version 1.4.4 as stated below please install latest version if needed" %}
    
Below section will set local ip-address
```shell

# Select required version
consul_version='1.6.1'

# Set local/private IP address
local_ipv4="`ip addr | grep 'state UP' -A2 | tail -n1 | awk '{print $2}' | cut -f1  -d'/'`"

# Detect package management system.
YUM=$(which yum 2>/dev/null)
APT_GET=$(which apt-get 2>/dev/null)
ZYPPER=$(which zypper 2>/dev/null)

```


## Prerequisite configuration for consul

```shell
#######################
# Install Prerequisites
#######################
echo "Installing jq"
sudo curl --silent -Lo /bin/jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64
sudo chmod +x /bin/jq

sudo zypper install -y wget unzip bind-utils ntp

echo "Disable reverse dns lookup in SSH"
sudo sh -c 'echo "\nUseDNS no" >> /etc/ssh/sshd_config'

echo "Completed Installing Prerequisites"
```

## Setting up a user in shell 

```shell
####################
# Set up Consul User
####################
USER="consul"
COMMENT="Consul"
GROUP="consul"
HOME="/srv/consul"

echo "Creating Consul user"

  # SUSE user setup
  sudo /usr/sbin/groupadd --force --system ${GROUP}

  if ! getent passwd ${USER} >/dev/null ; then
    sudo /usr/sbin/useradd \
      --system \
      --gid ${GROUP} \
      --home ${HOME} \
      --no-create-home \
      --comment "${COMMENT}" \
      --shell /bin/false \
      ${USER}  >/dev/null
  fi

```
## Installing Dnsmasq 
```shell
###############################
# Install and Configure Dnsmasq
###############################
  echo "Installing dnsmasq"
  sudo zypper install -y dnsmasq

echo "Configuring dnsmasq to forward .consul requests to consul port 8600"
sudo sh -c 'echo "server=/consul/127.0.0.1#8600" >> /etc/dnsmasq.d/consul'

sudo systemctl enable dnsmasq
sudo systemctl restart dnsmasq

```


## Where to Next?

To setup vagrant for Consul, joining nodes into a cluster, and
interacting with the agent, check out: [install and setup consul](consul_install_configure_consul).