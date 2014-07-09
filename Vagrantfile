#!/usr/bin/env ruby
Vagrant.configure("2") do |config|
  config.vm.box = "precise64"
  config.vm.box_url = 'http://files.vagrantup.com/precise64.box'

  inventory_files = File.expand_path("./inventory/files/", File.dirname(__FILE__))
  # config.vm.provision "shell", inline: "echo "+inventory_files


  config.vm.provider :virtualbox do |virtualbox|
    virtualbox.customize ["modifyvm", :id, "--memory", 512]
  end

  config.vm.define :dbserver do |node|
    node.vm.box = "precise64"
    node.vm.hostname = "dbserver"
    node.vm.network :forwarded_port, guest: 22, host: 2253
    node.vm.network :private_network, ip: "192.168.111.253"
  end

  config.vm.define :appserver1 do |node|
    node.vm.box = "precise64"
    node.vm.hostname = "appserver1"
    node.vm.network :forwarded_port, guest: 22, host: 2101
    node.vm.network :private_network, ip: "192.168.111.101"
  end

  config.vm.define :appserver2 do |node|
    node.vm.box = "precise64"
    node.vm.hostname = "appserver2"
    node.vm.network :forwarded_port, guest: 22, host: 2102
    node.vm.network :private_network, ip: "192.168.111.102"
  end

  config.vm.define :appserver3 do |node|
    node.vm.box = "precise64"
    node.vm.hostname = "appserver3"
    node.vm.network :forwarded_port, guest: 22, host: 2103
    node.vm.network :private_network, ip: "192.168.111.103"
  end

  config.vm.define :loadbalancer do |node|
    node.vm.box = "precise64"
    node.vm.hostname = "loadbalancer"
    node.vm.network :forwarded_port, guest: 80, host: 8080
    node.vm.network :forwarded_port, guest: 22, host: 2010
    node.vm.network :private_network, ip: "192.168.111.100"
    # define ansible provisioning only on last machine
    # in order to deal with https://github.com/mitchellh/vagrant/issues/1784
    node.vm.provision :ansible do |ansible|
      ansible.playbook = "playbook/site.yml"
      ansible.sudo = true
      ansible.inventory_path = "inventory/vagrant"
      ansible.extra_vars = { inventory_files: inventory_files }
    end
  end

end
