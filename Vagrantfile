# -*- mode: ruby -*-
# vi: set ft=ruby :
# vi: set shiftwidth=2
# vi: set softtabstop=2

control = "cobra"
build = "bigeye"
live = "trevally"

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  [control, build, live].each do |name|
    config.vm.define name do |machine|
      machine.vm.hostname = name
      machine.vm.provider "virtualbox" do |vb|
        vb.name = name
      end
    end
  end
end
