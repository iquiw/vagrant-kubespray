# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.boot_timeout = 60000
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1536"
  end

  ["node2", "node3"].each do |name|
    config.vm.define name do |node|
      node.vm.hostname = "#{name}.local"
      node.vm.network "private_network", ip: "172.16.0.1#{name[-1]}"
      node.vm.provision "file", source: "id_ed25519.pub", destination: "~/.ssh/id_ed25519.pub"
      node.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y --no-install-recommends python3

        mkdir -p ~/.ssh
        chmod 700 ~/.ssh
        cat /home/vagrant/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
      SHELL
    end
  end

  config.vm.define "node1", primary: true do |node|
    node.vm.hostname = "node1.local"
    node.vm.network "private_network", ip: "172.16.0.10"
    node.vm.provision "file", source: "id_ed25519", destination: "~/.ssh/id_ed25519"
    node.vm.provision "file", source: "id_ed25519.pub", destination: "~/.ssh/id_ed25519.pub"
    node.vm.provision "shell", inline: <<-SHELL
      mkdir -p ~/.ssh
      chmod 700 ~/.ssh
      mv /home/vagrant/.ssh/id_ed25519 ~/.ssh/id_ed25519
      chown root:root ~/.ssh/id_ed25519
      chmod 600 ~/.ssh/id_ed25519
      cat /home/vagrant/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys

      apt-get update
      apt-get install -y --no-install-recommends python3 virtualenv

      git clone --depth 1 https://github.com/kubernetes-incubator/kubespray.git
      cd kubespray
      virtualenv -p python3 .virtualenvs
      ./.virtualenvs/bin/pip install ansible netaddr

      cp -rfp inventory/sample inventory/mycluster
      CONFIG_FILE=inventory/mycluster/hosts.ini python3 contrib/inventory_builder/inventory.py 172.16.0.10 172.16.0.11 172.16.0.12
      ./.virtualenvs/bin/ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml
    SHELL
  end
end
