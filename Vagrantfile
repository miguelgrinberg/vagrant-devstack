# -*- mode: ruby -*-
# vi: set ft=ruby:

$script = <<SCRIPT
# download required packages
apt-get update
apt-get -y install git vim-gtk libxml2-dev libxslt1-dev libpq-dev python-pip libsqlite3-dev
apt-get -y build-dep python-mysqldb
pip install git-review tox

# download devstack
git clone git://git.openstack.org/openstack-dev/devstack
chown -R vagrant:vagrant devstack

# create minimal localrc
cat <<_LOCALRC > devstack/local.conf
[[local|localrc]]

# Passwords
ADMIN_PASSWORD=password
MYSQL_PASSWORD=password
RABBIT_PASSWORD=password
SERVICE_PASSWORD=password
SERVICE_TOKEN=password
SWIFT_HASH=password

# Logging
LOGFILE=/opt/stack/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=True
SCREEN_LOGDIR=/opt/stack/logs

# Neutron
disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta
enable_service neutron
#Q_PLUGIN=linuxbridge

# Swift
#enable_service s-proxy
#enable_service s-object
#enable_service s-container
#enable_service s-account

# Disable security groups
Q_USE_SECGROUP=False
LIBVIRT_FIREWALL_DRIVER=nova.virt.firewall.NoopFirewallDriver

# Network settings
HOST_IP=192.168.33.10
FLOATING_RANGE=172.24.1.0/24
PUBLIC_NETWORK_GATEWAY=172.24.1.1
FIXED_RANGE=10.0.0.0/24
NETWORK_GATEWAY=10.0.0.1
_LOCALRC

# Installation
ip link set dev eth2 up
cd devstack
sudo -u vagrant env HOME=/home/vagrant ./stack.sh
ovs-vsctl add-port br-ex eth2
SCRIPT

## Vagrant config
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.hostname = "devstack"

  # eth1, the private network
  config.vm.network :private_network, ip: "192.168.33.10"

  # eth2, the public network
  config.vm.network :private_network, ip: "172.24.1.2",
    :netmask => "255.255.255.0", :auto_config => false

  config.vm.provider :virtualbox do |vb|
    # set explicit VM name
    vb.name = "devstack"
    # change memory of our VM to 2048MB
    vb.customize ["modifyvm", :id, "--memory", "4096"]
    # enable promiscuous mode on the public network
    vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
  end

  config.vm.provision "shell", inline: $script
end

