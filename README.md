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

Then with 'vagrant up' command created the two vm's both running on debian/bullseye64. After the vm's were up and running, tested that i could ssh to both of them.

	$vagrant ssh debian1
	$vagrant ssh debian2

As screenshot shows ssh, i could ssh to both my machines.
--screenschot--

### Install salt minion and master and connect minions to master

With vm's now running, to complete my test enviroment i installed salt-minion to both of vm's and salt-master to my own, not virtual, debian.
Prior to installing salt-minion to vm's i  ran apt-get update and apt-get dist-upgrade to get the machines up to date.
 
	$ sudo apt-get install salt-minion
	$ sudo apt-get install salt-master

Next i told the minions who their master is and where to connect. 
With 'hostname -I' command i checked my master's ip and put it in minion's file - minion.

	vagrant@debian1:~$ cd /etc/salt/
	vagrant@debian1:/etc/salt$ sudo nano minion

	master: 192.168.0.117
	id: debian1
	.
	.
	.

 Repeated the same to debian2. After making changes i restared salt-minions.
 Then as a master went to check if minions are waiting to be accepted. 
 
	vagrant@debian2:/etc/salt$ sudo systemctl restart salt-minion #restart
	
 	tuomo@debian:~/miniproject$ sudo salt-key
 	Accepted Keys:
 	Denied Keys:
 	Unaccepted Keys:
 	debian1
 	debian2

 Next accepted keys for both debian1 and debian2

 	tuomo@debian:~/miniproject$ sudo salt-key -a debian1
 	tuomo@debian:~/miniproject$ sudo salt-key -a debian2

And finally a little test to see if everything is working as it should be.

	tuomo@debian:~/miniproject$ sudo salt '*' cmd.run 'ls /etc/salt/'
	debian1:
	    minion
	    minion.d
	    minion_id
	debian2:
	    minion
	    minion.d
	    minion_id

Everything ok.

## Creating my own little salt-state

First created a simple salt-state for testing, apllied the state and at last cheked that the file had been created.

	tuomo@debian:~$ cd /srv/salt
	tuomo@debian:/srv/salt$ sudo mkdir miniproject
	tuomo@debian:/srv/salt$ cd miniproject
	tuomo@debian:/srv/salt/miniproject$ sudo micro init.sls

	/tmp/hello:
	  file.managed:
	    - contents: "Hello W"

	tuomo@debian:/srv/salt/miniproject$ sudo salt '*' state.apply miniproject

	tuomo@debian:/srv/salt/miniproject$ sudo salt '*' cmd.run 'cat  /tmp/hello'
	debian2:
	    Hello W
	debian1:
	    Hello W
	
	


 	
