# -*- mode: ruby -*-
# vi: set ft=ruby :

$configureCommon = <<-SHELL

  # apt-get update
  # apt-get upgrade -y

  ## cni pluginsを展開
  mkdir -p /opt/cni/bin
  curl -sSL https://github.com/containernetworking/plugins/releases/download/v0.7.5/cni-plugins-amd64-v0.7.5.tgz | tar xzf - -C /opt/cni/bin

SHELL

$configureMaster = <<-SHELL

  # プライベートネットワークのNICのIPアドレスを変数に格納
  IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)

  ## k3sのデプロイ flannelをOFF プライベートIPをkubeletのIPとするように指定
  ## ここで、flannelをOFFにしているのは、現在ではVxLANの送出IFの指定ができず、Vagrant環境の場合、NATインターフェースが選択されてしまうため都合が悪い
  ## k3sのFlannelはあきらめて、flannelは別途構築する
  curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--no-flannel --node-ip=${IPADDR}" sh -

  ## flannelのデプロイ
  kubectl apply -f /vagrant/kube-flannel.yaml

  ## agent登録用のトークンを共有フォルダに配置
  cp /var/lib/rancher/k3s/server/node-token /vagrant/token

SHELL

$configureNode = <<-SHELL

  export K3S_TOKEN=$(cat /vagrant/token)
  export K3S_URL=https://192.168.33.11:6443
  # プライベートネットワークのNICのIPアドレスを変数に格納
  IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)

  ## k3sのデプロイ flannelをOFF プライベートIPをkubeletのIPとするように指定
  curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--no-flannel --node-ip=${IPADDR}" sh -

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

      # ホスト名
      s.vm.hostname = vm_name
      # ノードのベースOSを指定
      s.vm.box = "ubuntu/xenial64"
      # ネットワークを指定
      private_ip = "192.168.33.#{i+10}"
      s.vm.network "private_network", ip: private_ip

      # 共通
      s.vm.provision "shell", inline: $configureCommon

      if i == 1 then
        # For Master
        s.vm.provision "shell", inline: $configureMaster
      else
        # For Nodes
        s.vm.provision "shell", inline: $configureNode
      end

    end
  end
end