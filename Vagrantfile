# -*- mode: ruby -*-
# vi: set ft=ruby :
# vi: set shiftwidth=2
# vi: set softtabstop=2
require 'yaml'
machines = YAML.load_file 'vagrant.yml'

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.synced_folder ".", "/vagrant"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  machines.each do |role, settings|
    name = settings['name']
    config.vm.define name do |machine|
      machine.vm.hostname = name
      machine.vm.network "private_network", ip: settings['ip_address']
      machine.vm.provider "virtualbox" do |vb|
        vb.name = name
      end
    end
  end

  config.vm.define machines['control']['name'] do |control|
    control.vm.provision "shell", inline: <<-SHELL
      sudo yum install -y vim-enhanced.x86_64
      sudo yum install -y git
      sudo yum install -y ansible
    SHELL
  end
end
