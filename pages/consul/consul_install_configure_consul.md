---
title: Install and setup consul
keywords: consul,install, consul install
summary: "Installing consul in your machine"
sidebar: consul_sidebar
permalink: consul_install_configure_consul
folder: consul
---
## Install Consul

```` shell
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
````

## Create consul as a service

```` shell
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

````


## Where to Next?
Now you should have consul installed in this machine.   
 
>In order to install I created single script which can be used to install in rest of the 5 servers

To setup vagrant for Consul, joining nodes into a cluster, and
interacting with the agent, check out: [Single Script](consul_install).