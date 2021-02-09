# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  #https://app.vagrantup.com/debian/boxes/contrib-buster64
  config.vm.box = "debian/contrib-buster64"

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 4
    vb.memory = "4096"

    # USB3 Passthrough
    vb.customize ["modifyvm", :id, "--usbxhci", "on"]
    # Will find the manu and product from `VBoxManage list usbhost`
    vb.customize ["usbfilter", "add", "0", 
      "--target", :id, 
      "--name", "USB Ethernet",
      "--manufacturer", "Realtek",
      "--product", "USB 10/100/1000 LAN"]
   end

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
sudo apt-get update
sudo apt-get install -y --no-install-recommends \
  stoken \
  docker.io \
  dnsutils

sudo tee --append /etc/network/interfaces <<'EOF'
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp
  metric 100

# The 'Gateway' network interface
allow-hotplug eth1
iface eth1 inet static
   address 192.168.5.2/24
EOF

sudo ifup eth1

sudo docker build -t local/dns /vagrant/docker/dns
sudo docker build -t local/pac /vagrant/docker/pac

sudo cp -fv /vagrant/systemd/*.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now dns
sudo systemctl enable --now pac

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

if [[ -f /vagrant/.stokenrc ]]; then
  sudo cp -f /vagrant/.stokenrc $HOME/
fi

  SHELL
end
