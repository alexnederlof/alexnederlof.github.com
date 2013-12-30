---
layout: post
title: "Running and testing Puppet master locally"
date: 2013-12-25 13:33
comments: true
categories: ["Automate everything", "Puppet", Testing]
---
About a month ago we started to use [Puppet](http://puppetlabs.com/) at [Magnet.me](http://magnet.me). Puppet now automates all our server provisioning from our dev server, to the Raspberry Pi we have running at the office. While learning Puppet, we could not find any information about developing Puppet manifests locally and testing them on local copies of the machines. As a result, we ended up editing the Puppet manifests directly over an SSH connection. This prevented us from having a nice pull-request Git flow and often led to syntax errors in the manifests making it impossible for operational nodes to update. In this blog post, I will explain the solution we came up with to test our configurations on local VMs.

<!--more-->

This blog post assumes you have a Puppet configuration like in `/etc/puppet/` on your Puppet master which is also a Git repository. You should also have [Vagrant](http://vagrantup.com) and [VirtualBox](https://www.virtualbox.org/) installed. If you're trying to figure out how to set-up a Puppet master and some slaves I recommend reading the [Pro Puppet][pro-puppet] book.

## Setting up the Virtual Machine.
We do not actually require a running Puppet master but instead will use the `puppet apply` command. This will apply a given configuration from its local `/etc/puppet` directory depending on the VM's hostname. 

First we set-up our Vagrant file to start a machine (Ubuntu in this example), and we mount the `/etc/puppet` folder into place. Remember that your working directory (the Git repo) is that directory. To keep things tidy, we put the Vagrant file in a `test` folder in the repository. This means the directory structure of you Git repo will be something like:

	- manifests
 		\- site.pp
	- modules
 		\- your modules (if any)
	- test
 		\- update-puppet.sh
 		\- Vagrantfile
	- puppet.conf

{% codeblock Vagrantfile lang:ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box     = "precise32"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", 1028, "--cpus", 2]
  end
  
  # Mount our repo onto /etc/puppet
  config.vm.synced_folder "../", "/etc/puppet"

  # Run our Puppet shell script  
  config.vm.provision "shell" do |s|
	  s.path = "update-puppet.sh"
  end

  config.vm.hostname = "localdev.example.com"
end
{% endcodeblock %}

As you can see we are using the "shell" provisioner to run a special script `update-puppet` that runs `puppet apply`. The script first installs Puppet 3 and then runs `puppet apply`.


```bash update-puppet.sh
#!/bin/bash
echo "Puppet version is $(puppet --version)"
if [ $( puppet --version) != "3.4.1" ]; then 
	echo "Updating puppet"
	apt-get install --yes lsb-release
	DISTRIB_CODENAME=$(lsb_release --codename --short)
	DEB="puppetlabs-release-${DISTRIB_CODENAME}.deb"
	DEB_PROVIDES="/etc/apt/sources.list.d/puppetlabs.list" 

	if [ ! -e $DEB_PROVIDES ]
	then
    	wget -q http://apt.puppetlabs.com/$DEB
    	sudo dpkg -i $DEB
	fi
	sudo apt-get update
	sudo apt-get install -o Dpkg::Options::="--force-confold" --force-yes -y puppet
else
	echo "Puppet is up to date!"
fi

echo "Running puppet"
sudo puppet apply /etc/puppet/manifests/site.pp
```

Because the hostname of the machine is set to `localdev.example.com` Puppet will run that configuration from `manifests/site.pp`. The manifest installs some packages. For this example it just installs Git and vim. It looks like this:

```puppet site.pp
# -*- mode: ruby -*-
# vi: set ft=ruby :
# 
node 'localdev.example.com' {
	package { ['vim','git'] :
		ensure => latest
	}
}
```
Once you have all these files in place you can visit your `test` folder and run `vagrant up`. This will load a new VM, install Puppet 3 using `update-puppet.sh` and then run puppet. You should see it installs vim and Git like in the output like so:

	Notice: Compiled catalog for localdev.example.com in environment production in 0.09 seconds
	Notice: /Stage[main]/Main/Node[localdev.example.com]/Package[git]/ensure: created
 	Notice: /Stage[main]/Main/Node[localdev.example.com]/Package[vim]/ensure: ensure changed 'purged' to 'latest'


## Configuring multiple machines

If you like us have multiple machines, we can easily adjust the files a little to test multiple machines. Lets say we also have the node `production.example.com`. First, we delete the previous VM we made by running `vagrant destroy -f` in the `test` folder. After that we adjust the Vagrant file to configure two hosts. First remove the old line `config.vm.hostname = "localdev.example.com"`, then add: 

	config.vm.define "localdev" do |localdev|
		localdev.vm.hostname = "localdev.example.com"
	end

	config.vm.define "production" do |production|
		production.vm.hostname = "app.magnet.me"
	end

Lets say our new production server should have *Curl* installed. We extend the `site.pp` with the new production server node:

	node 'production.example.com' inherits 'localdev.example.com'  {
	  package { ['curl'] :
	      ensure => latest
	  }
	}
	
Now if we run `vagrant up` it will start both machines. You can also run `vagrant up production` to just start the production machine. In the output, you should see that *production* installs *Curl*, but *localdev* does not. Congratulations, you can now develop Puppet on your local machine.

[pro-puppet]: http://it-ebooks.info/book/730/  "Pro Puppet book"
