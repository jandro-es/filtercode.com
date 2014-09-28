Title: Apache Cassandra's development boxes with Ansible
Date: 2014-09-28 10:20
Category: Python
Tags: devops, cassandra, ansible
Slug: ansible-cassandra-playbook
Authors: jandro_es
Summary: An easy way to create development boxes for **Apache Cassandra** using **Ansible** tocreate a Cassandra architecture for develop our applications.

As you probably know, install and configure different Cassandra's nodes and cluster, is not a straight forward task. One way to ease up the task when we're developing our applications, is to delegate this task to **Ansible**. This way we can create Cassandra boxes in **Vagrant** in an easy way, and even test it's fault tolerance when some of the nodes are down.

As with any of these local playbooks, we need to install Vagrant, Virtual Box, etc. We can follow the instructions in this other [post](http://www.filtercode.com/python/dev-with-vagrant-ansible), up and including the creation of the **Vagrantfile**.

Once we have our environment ready for Vagrant and Ansible, we create, as always our **playbook** file:

**playbook.yml**
~~~~{.language-python}
	---

	- hosts: localhost
  	  remote_user: root
      roles:
        - common-utils
        - cassandra
        - opscenter
~~~~

Here we're telling ansible, that we want to install three roles in our localhost machine (the one vagrant will create).

Roles:

   * **common-utils** will install *Oracle's Java 7* which is the recommended **JVM** version to use with Cassandra.
   * **cassandra** will install Cassandra's latest version and configure the node/cluster based on several parameters.
   * **opscenter** will install DataStax's Ops Center for managing Cassandra instances.


####common-utils role

As always, we need to update the dist files to their latest versions:

~~~~{.language-python}
	- name: Update system dist files
  	  action: apt update_cache=yes
      tags: common-utils
~~~~

we install several common utilities and python libraries needed by Cassandra's commands:

~~~~{.language-python}
	- name: Install several packages needed and utilities
	  action: apt pkg={{ item }} state=installed
	  tags: common-utils
	  with_items:
	    - htop
	    - vim
	    - python-pip
	    - curl
	    - git
	    - unzip
	    - python-software-properties
~~~~

we add Oracle's Java 7 repository:

~~~~{.language-python}
	- name: Add JRE ppa
  	  apt_repository: repo=ppa:webupd8team/java state=present
      tags: oracle-java-7
~~~~

in last versions, *Oracle's Java 7* needs you to accept their license while installing, this usually prevents to automatise it's installation, fortunately we can force it's acceptance with the following command:

~~~~{.language-python}
	- name: Automatically select the Oracle License
  	  shell: echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections
      tags: oracle-java-7
~~~~

and finally, we install the JVM

~~~~{.language-python}
	- name: Install JRE
  	  apt: pkg=oracle-java7-installer state=latest update-cache=yes force=yes
      tags: oracle-java-7
~~~~

####cassandra role

We add Cassandra's repository as well as the needed keys

~~~~{.language-python}
	- name: Adds Cassandra's repository
	  apt_repository: repo='deb http://www.apache.org/dist/cassandra/debian 20x main' state=present
	  tags: Cassandra

	- name: Adds the key for Cassandra's repository
	  apt_key: keyserver=pgp.mit.edu id=F758CE318D77295D
	  tags: Cassandra

	- name: Adds other needed keys
	  apt_key: keyserver=pgp.mit.edu id=2B5C1B00
	  tags: Cassandra
~~~~

we install Cassandra's package

~~~~{.language-python}
	- name: Installs Cassandra
	  apt: name=cassandra state=present update_cache=yes
	  tags: Cassandra
~~~~

Now we need two templates to configure our Cassandra instance:

*	**cassandra.yml** which is Cassandra's main configuration file, you can see the template in [here](https://github.com/jandro-es/CassandraAnsible/blob/master/ansible/roles/cassandra/templates/cassandra.j2).
*	**cassandra-env.sh** which overrides several system configurations, mainly the JVM one. You can see the template in [here](https://github.com/jandro-es/CassandraAnsible/blob/master/ansible/roles/cassandra/templates/cassandra-env.j2).

These templates needs quite a few variables so you can create different clusters/nodes and fine tunning you JVM for Cassandra. This is my default configuration:

~~~~{.language-python}
	# Cassandra node configuration values
	cluster_name: test_cluster
	seeds: 10.0.0.10
	listen_address: localhost

	# JVM configuration

	# 1/2 system memory recommended
	MAX_HEAP_SIZE: 1GB
	# 1/4 MAX_HEAP_SIZE recommended
	HEAP_NEWSIZE: 256MB
~~~~

If you want ot have different nodes and clusters, you just have to change **cluster_name** and **seeds** values among boxes to achieve the architecture you wanted.

Now we copy those templates and restart Cassandra with this new configuration values

~~~~{.language-python}
	- name: Creates Cassandra's configuration file
	  template: src=cassandra.j2 dest=/etc/cassandra/cassandra.yml
	  tags: Cassandra

	- name: Creates Cassandra's JVM Tunning file
	  template: src=cassandra-env.j2 dest=/etc/cassandra/cassandra-env.sh
	  tags: Cassandra

	- name: Restarts Cassandra
	  service: name=cassandra state=restarted
	  tags: Cassandra
~~~~

With these steps, we already have a Cassandra instance up and running.

####ops-center role

For easy Cassandra's instances management, we are going to install [DataStax's Opscenter](http://www.datastax.com/what-we-offer/products-services/datastax-opscenter). This is an easy one.

~~~~{.language-python}
	- name: Adds datastax repository for opscenter
	  apt_repository: repo='deb http://debian.datastax.com/community stable main' state=present
	  tags: OpsCenter

	- name: Adds the key for the repository
	  apt_key: url=http://debian.datastax.com/debian/repo_key state=present
	  tags: OpsCenter

	- name: Installs Opscenter
	  apt: name=opscenter state=present update_cache=yes
	  tags: OpsCenter

	- name: Starts Opscenter
	  service: name=opscenterd state=started
	  tags: OpsCenter
~~~~

And we're done, we already have a Cassandra box fully functional to start working with it.

**This entire playbook is ready for you to use in it's [Github's repository](https://github.com/jandro-es/CassandraAnsible)**


