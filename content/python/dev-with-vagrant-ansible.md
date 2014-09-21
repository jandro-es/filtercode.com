Title: Easy development environments with Vagrant and Ansible
Date: 2014-09-19 10:20
Category: Python
Tags: python, ansible, vagrant, devops
Slug: dev-with-vagrant-ansible
Authors: jandro_es
Summary: How to create, provision, evolve and use custom development environments using **Vagrant** and **Ansible**. An easy way of simplify your development environments.

Usually we need to develop in different environments **Python**, **Scala**, **PHP**, **NodeJS** etc.. forcing us to struggle with having local instalations for all of those environments causing conflicts most of the time. Also when we need to change computers or reinstall them, we have to configure everything again. And one of the most common issues is to have projects on different versions of the same environment. Luckily in **Python** we have **virtualenv** to help addressing thins issues, but to keep the same working methology among all the environments we can use **Vagrant** and **Ansible** to make our lifes easier.

First we need to download and install [Vagrant](https://www.vagrantup.com) which will be responsible for managing all of our virtual machines. We need also to download and install [VirtualBox](https://www.virtualbox.org) which is going to be our virtualizer, Vagrant support a lot more of them, you can check them on their webpage.

In order to speed up the execution of the virtual machines we need to install Vagrant's plugin for the *guest additions*

~~~~{.language-bash}
	vagrant plugin install vagrant-vbguest
~~~~

now we need to install **Ansible**, in OSX we can execute one of the following commands depending in which package manager are we using:

**macports**
~~~~{.language-bash}
	sudo port install ansible
~~~~
**homebrew**
~~~~{.language-bash}
	brew install ansible
~~~~

and in **Ubuntu**

~~~~{.language-bash}
	sudo apt-get install python-software-properties
	sudo add-apt-repository ppa:rquillo/ansible
	sudo apt-get update
	sudo apt-get install ansible
~~~~

Now we create in the same root as our application a *Vagrant's configuration file*, we call it **Vagrantfile**

~~~~{.language-ruby}
	# -*- mode: ruby -*-
	# vi: set ft=ruby :

	# Vagrant Configuration file using Ansible as provisioning
	VAGRANTFILE_API_VERSION = "2"
	Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	  config.vm.box = "Precision64"
	  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
	  
	  config.vm.network "private_network",ip: "10.0.0.5"
	  config.vm.network "forwarded_port", guest: 80, host: 8080
	  
	  config.vm.define "localhost" do |l|
	    l.vm.hostname = "localhost"
	  end

	  config.vm.provider :virtualbox do |vb|
	    vb.name = "ansible-provisioned-machine"
	  end
	  
	  config.vm.synced_folder ".", "/var/www/local.project", type: "nfs"

	  config.vm.provider :virtualbox do |vb|
	    vb.customize ["modifyvm", :id, "--memory", "2048"]
	    vb.customize ["modifyvm", :id, "--cpus", "2"] 
	  end

	  config.vm.provision "ansible" do |ansible|
	    ansible.sudo = true
	    ansible.playbook = "ansible/playbook.yml"
	    ansible.verbose = "v"
	    ansible.host_key_checking = false
	  end
	end
~~~~

Let's examine it:
~~~~{.language-ruby}
	config.vm.box = "Precision64"
	config.vm.box_url = "http://files.vagrantup.com/precise64.box"
~~~~
Here we're telling Vagrant which image hast to use to create our virtual machine, in this case we're using **64-bit Ubuntu 12.04 LTS** image, there are a lot of them already pre configured, you can see a list in [Vagrantbox.es](http://www.vagrantbox.es).

~~~~{.language-ruby}
	config.vm.network "private_network",ip: "10.0.0.5"
	config.vm.network "forwarded_port", guest: 80, host: 8080
~~~~
In these lines we're setting the IP address the virtual machine is going to use, in this case **10.0.0.5**, we're also forwading the 8080 port on our host machine *(a.k.a. our computer)* to port 80 on the guest machine *(a.k.a. the virtual machine)*, so to access it we'll use **10.0.0.5:8080**. You need to have different IP addresses for each virtual machine you intend to execute at the same time.

~~~~{.language-ruby}
	config.vm.provider :virtualbox do |vb|
		vb.name = "ansible-provisioned-machine"
	end
~~~~
**ansible-provisioned-machine** this is the name that will appear on the VirtualBox UI interface for this machine, you need to have different names for every environment you create.

~~~~{.language-ruby}
	config.vm.synced_folder ".", "/var/www/local.project", type: "nfs"
~~~~
Here we're telling Vagrant to synchronise our current path **.** to the path **/var/www/local.project** in the virtual machine using **nfs**. This way we can work locally on the files using our favourite IDE and keep the file up to date on the virtual machine where we run our application.

~~~~{.language-ruby}
	config.vm.provider :virtualbox do |vb|
		vb.customize ["modifyvm", :id, "--memory", "2048"]
	    vb.customize ["modifyvm", :id, "--cpus", "2"] 
	end
~~~~
We're telling vagrant to use 2048MB of RAM and 2 cores for the virtual machine, you can adjust this depending on you needs.

~~~~{.language-ruby}
	config.vm.provision "ansible" do |ansible|
	    ansible.sudo = true
	    ansible.playbook = "ansible/playbook.yml"
	    ansible.verbose = "v"
	    ansible.host_key_checking = false
	end
~~~~

This is where we tell vagrant to use ansible for provisioning the virtual machine, in this case we'll use the **playbook** located in **ansible/playbook.yml**.

Now we'll create our playbook, inside a folder called *ansible* we edit a file called **playbook.yml**

~~~~{.language-python}
	- hosts: localhost
	  roles:
	    - devmachine
	    - php5
	    - apache
	    - mysql
	    - memcache
	    - node
	    - css
	    - redis
	    - sphinx
	    - hosts
	    - mongodb
~~~~
in here we're telling **ansible** that for the host *locahost* (it's the one we define on our Vagrantfile) we want the following roles provisioned: *devmachine, php5, apache, etc..*

**You can see this playbook fully funcional in this [Github repository](https://github.com/jandro-es/DevBoxAnsible).**

A full explanation of Ansible's roles, handlers, etc.. is out of the scope of this post, you can see them all in [Ansible's webpage](http://www.ansible.com/home).

But let's see some of the most used commands:

####Install packages
~~~~{.language-python}
	- name: Update dist files
	  action: apt update_cache=yes
	  tags: common

	- name: Installs common utilities for development
	  action: apt pkg={{ item }} state=installed
	  tags: common
	  with_items:
	    - htop
	    - vim
	    - ruby
	    - rubygems
	    - python-pip
	    - curl
	    - git
	    - sendmail
	    - default-jre
	    - unzip
~~~~

####Create an user and add it to the sudo group
~~~~{.language-python}
	- name: Create user
	  user: home={{ user_home }} name={{ user }} shell=/bin/bash state=present

	- name: Add user to sudo group
	  user: name={{ user }} groups=sudo append=true
~~~~

####Change permissions to a folder
~~~~{.language-python}
	- name: Change permissions
	  shell: chown -R {{ user }}:{{ user }} {{ folder }}
~~~~

####Pull a repository from Github
~~~~{.language-python}
	- name: Pull source from Github
	  git: repo={{ repo }} dest={{ directory }} accept_hostkey=yes
	  sudo: yes
	  sudo_user: '{{ user }}'
	  tags: 
	    - deploy
~~~~

####Install Python requirements using pip
~~~~{.language-python}
	- name: Install requirements
	  pip: requirements={{ directory }}requirements.txt virtualenv={{ user_home }}env
	  sudo: yes
	  sudo_user: '{{ user }}'
	  tags:
	    - deploy
~~~~

####Install a Nginx configuration file
~~~~{.language-python}
	- name: Copy nginx config
	  template: src=nginx.conf.j2 dest=/etc/nginx/sites-enabled/{{ user }}
	  tags:
	    - deploy
~~~~

####Reload a service, in this case Nginx
~~~~{.language-python}
	- name: Reload nginx
	  service: name=nginx state=reloaded
	  tags:
	    - deploy
~~~~
Ansible uses **Jinja2** as template engine, so you can use the familiar template syntax to create really flexible playbooks.

and a lot more, you can check the DevBox example in [Github repository](https://github.com/jandro-es/DevBoxAnsible) or check all the examples provided by [Ansible](http://www.ansible.com/home).