---
title: Consul Installation script
keywords: consul,install, consul install
summary: "Installing consul in your machine"
sidebar: consul_sidebar
permalink: consul_install
folder: consul
---

# Single shell script to install consul

> copy entire bellow scrip to `consul.sh` and run it in each and every vm and run it in all the 6 Vagant vm's

{% include tip.html content="Run '$service consul restart' after installing consul in every machine and check for status" %}
    
```shell

# Select required version
consul_version='1.6.1'

# Set local/private IP address
local_ipv4="`ip addr | grep 'state UP' -A2 | tail -n1 | awk '{print $2}' | cut -f1  -d'/'`"

# Detect package management system.
YUM=$(which yum 2>/dev/null)
APT_GET=$(which apt-get 2>/dev/null)
ZYPPER=$(which zypper 2>/dev/null)

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

###############################
# Install and Configure Dnsmasq
###############################
  echo "Installing dnsmasq"
  sudo zypper install -y dnsmasq

echo "Configuring dnsmasq to forward .consul requests to consul port 8600"
sudo sh -c 'echo "server=/consul/127.0.0.1#8600" >> /etc/dnsmasq.d/consul'

sudo systemctl enable dnsmasq
sudo systemctl restart dnsmasq


##############################
# Install and Configure Consul
##############################
CONSUL_VERSION="${consul_version}"
CONSUL_ZIP="consul_${CONSUL_VERSION}_linux_amd64.zip"
CONSUL_URL="https://releases.hashicorp.com/consul/${CONSUL_VERSION}/${CONSUL_ZIP}"

echo "Downloading consul ${CONSUL_VERSION}"
echo "curl --silent --output /tmp/${CONSUL_ZIP} ${CONSUL_URL}"
curl --silent --output /tmp/${CONSUL_ZIP} ${CONSUL_URL}

logger "Installing consul"
sudo unzip -o /tmp/${CONSUL_ZIP} -d /usr/local/bin/
sudo chmod 0755 /usr/local/bin/consul
sudo chown consul:consul /usr/local/bin/consul
sudo mkdir -pm 0755 /etc/consul.d
sudo mkdir -pm 0755 /opt/consul/data
sudo chown consul:consul /opt/consul/data

echo "/usr/local/bin/consul --version: $(/usr/local/bin/consul --version)"

# Write base client Consul config
sudo tee /etc/consul.d/consul-default.json <<EOF
{
  "advertise_addr": "${local_ipv4}",
  "data_dir": "/opt/consul/data",
  "client_addr": "0.0.0.0",
  "log_level": "INFO",
  "ui": true
}
EOF

###############################
# Create Consul Systemd Service
###############################
sudo mkdir -p /tmp/consul/init/systemd/
sudo tee /tmp/consul/init/systemd/consul.service <<'EOF'
[Unit]
Description=Consul Agent
Requires=network-online.target
After=network-online.target

[Service]
Restart=on-failure
ExecStart=/usr/local/bin/consul agent -config-dir /etc/consul.d
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
User=consul
Group=consul

[Install]
WantedBy=multi-user.target
EOF

  SYSTEMD_DIR="/etc/systemd/system"
  echo "Installing Consul systemd service"
  sudo cp /tmp/consul/init/systemd/consul.service ${SYSTEMD_DIR}
  sudo chmod 0664 ${SYSTEMD_DIR}/consul.service

sudo systemctl enable consul
service consul start
service consul status

echo "################ Consul installation complete #################"

echo "################ Adding Acl token #################"
echo "{
\"acl\" : {
  \"enabled\" : true,
  \"default_policy\" :\"allow\",
  \"enable_token_persistence\" : true,
  \"tokens\" : {
              \"master\" : \"398073a8-5091-4d9c-871a-bbbeb030d1f6\"
   }
}
}">/etc/consul.d/consul-acl.json

###############################
# Restarting Service
###############################
service consul restart
sleep 15
cat /etc/consul.d/consul-acl.json

````


## Where to Next?
By this time you will have consul available in all servers.    
 
Joining nodes into a cluster, and
interacting with the agent, check out: [See how to setup Data center for consul](consul_dc_setup).