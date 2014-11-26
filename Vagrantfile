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

# copy local.conf
cp /vagrant/local.conf devstack/

# Installation
ip link set dev eth2 up
chown -R vagrant:vagrant devstack
cd devstack
sudo -u vagrant env HOME=/home/vagrant ./stack.sh
ovs-vsctl add-port br-ex eth2
SCRIPT

## Vagrant config
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "trusty64"
  config.vm.hostname = "devstack"

  # eth1, the private network
  config.vm.network :private_network, ip: "192.168.33.10"

  # eth2, the public network
  config.vm.network :private_network, ip: "172.24.1.2",
    :netmask => "255.255.255.0", :auto_config => false

  config.vm.provider :virtualbox do |vb|
    # more RAM
    vb.customize ["modifyvm", :id, "--memory", "4096"]
    # enable promiscuous mode on the public network
    vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
  end

  config.vm.provision "shell", inline: $script
end

