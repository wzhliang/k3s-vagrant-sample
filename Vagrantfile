# -*- mode: ruby -*-
# vi: set ft=ruby :

$configureCommon = <<-SHELL

  # apt-get update
  # apt-get upgrade -y
  mkdir -p /opt/cni/bin
  curl -sSL https://github.com/containernetworking/plugins/releases/download/v0.7.5/cni-plugins-amd64-v0.7.5.tgz | tar xzf - -C /opt/cni/bin

SHELL

$configureMaster = <<-SHELL
  IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)

  curl -sfL https://get.k3s.io | sh -

  # kubectl apply -f /vagrant/kube-flannel.yaml

  cp /var/lib/rancher/k3s/server/node-token /vagrant/token

SHELL

$configureNode = <<-SHELL

  export K3S_TOKEN=$(cat /vagrant/token)
  export K3S_URL=https://192.168.33.11:6443
  IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
  curl -sfL https://get.k3s.io | sh -

SHELL

Vagrant.configure(2) do |config|

  node_num = 3

  (1..node_num).each do |i|

    if i == 1 then
      vm_name = "master"
    else
      vm_name = "node#{i-1}"
    end

    config.vm.define vm_name do |s|

      s.vm.hostname = vm_name
      s.vm.box = "ubuntu/xenial64"
      private_ip = "192.168.33.#{i+10}"
      s.vm.network "private_network", ip: private_ip

      s.vm.provision "shell", inline: $configureCommon

      if i == 1 then
        s.vm.provision "shell", inline: $configureMaster
      else
        # For Nodes
        s.vm.provision "shell", inline: $configureNode
      end

    end
  end
end
