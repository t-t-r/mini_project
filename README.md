#Miniproggis

## Testing that changes take effect

### Setting up a virtual machines with Vagrant and Virtualbox
First i needed to set up a couple of virtual machines. 
These virtual machines will in future  act as minions and for me to test my salt project on.

First created a directory for the the project and created a Vagrant file in it with 'vagrant init' command.
	$ mkdir miniproject
	$ cd miniproject
	$ vagrant init

Next i edited the Vagrantfile with micro.

	# -*- mode: ruby -*-
	# vi: set ft=ruby :
	# Copyright 2019-2021 Tero Karvinen http://TeroKarvinen.com
	
	$tscript = <<TSCRIPT
	set -o verbose
	apt-get update
	TSCRIPT
	
	Vagrant.configure("2") do |config|
		config.vm.synced_folder ".", "/vagrant", disabled: true
		config.vm.synced_folder "shared/", "/home/vagrant/shared", create: true
		config.vm.provision "shell", inline: $tscript
		config.vm.box = "debian/bullseye64"
	
		config.vm.define "debian1" do |t001|
			t001.vm.hostname = "debian1"
			t001.vm.network "private_network", ip: "192.168.60.101"
		end
	
		config.vm.define "debian2", primary: true do |t002|
			t002.vm.hostname = "debian2"
			t002.vm.network "private_network", ip: "192.168.60.102"
		end
		
	end

