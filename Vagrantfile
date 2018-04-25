# -*- mode: ruby -*-
# vi: set ft=ruby :
# vi: set shiftwidth=2
# vi: set softtabstop=2
require 'yaml'
inventory = YAML.load_file 'inventory.yml'

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.synced_folder ".", "/vagrant"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  inventory.each do |group, hosts|
    hosts['hosts'].each do |name, settings|
      config.vm.define name do |machine|
        machine.vm.hostname = name
        machine.vm.network "private_network", ip: settings['ansible_host']
        machine.vm.provider "virtualbox" do |vb|
          vb.name = name
        end
      end
    end
  end

  # Copy over key pairs for sshing between machines
  config.vm.provision "file", source: "id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
  public_key = File.read("id_rsa.pub")
  config.vm.provision "shell", inline: <<-SHELL
      echo 'Copying public SSH Keys to the VM'
      mkdir -p /home/vagrant/.ssh
      chmod 700 /home/vagrant/.ssh
      echo '#{public_key}' >> /home/vagrant/.ssh/authorized_keys
      chmod -R 600 /home/vagrant/.ssh/authorized_keys
      echo 'Host 192.168.*.*' >> /home/vagrant/.ssh/config
      echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
      echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
      chmod -R 600 /home/vagrant/.ssh/config
      echo 'Copying ssh key pair to root'
      cp -r /home/vagrant/.ssh /root/.ssh
    SHELL

  groups = Hash.new()
  inventory.each do |group, hosts|
    groups[group] = hosts['hosts'].keys
  end
  groups['control'].each do |name|
    config.vm.define name do |control|
      control.vm.provision "shell", inline: <<-SHELL
        sudo yum install -y vim-enhanced.x86_64
        sudo yum install -y git
      SHELL
      control.vm.provision "ansible_local" do |ansible|
        ansible.verbose = "vvvv"
        ansible.playbook = "playbook.yml"
        ansible.inventory_path = "inventory.yml"
        ansible.groups = groups
        ansible.become = "yes"
        ansible.limit = "all"
      end
    end
  end
end
