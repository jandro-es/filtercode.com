Title: Install global NPM packages without sudo in Mac OSX
Date: 2015-07-03 10:20
Category: devops
Tags: npm, nodejs, npm, devops
Slug: npm-global-without-sudo
Authors: jandro_es
Summary: How to install **npm** global packages in **OSX** without using *sudo*.

The usual installation for **nodejs** and **npm** in an OSX system will make you use *sudo* every time you want to install any *npm* package as global. And most of them, will not work if *npm* was installed using sudo. These are the steps to install it in such a way so you don't have those problems.

####Install HomeBrew

If you don't already have it, is as simple as typing the following command:

~~~~{.language-bash}
	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~~



####Install Node

We need to install it without npm so we can set it up properly afterwards:

~~~~{.language-bash}
	brew install node --without-npm
~~~~



####Install NPM

You need to create a directory for storing your global packages. For example on called *npm-global-pkg* inside you home directory:

~~~~{.language-bash}
	mkdir ${HOME}/.npm-global-pkg
~~~~

In your *profile* or *bashrc* file, add the following environment variable export:

~~~~{.language-bash}
	export NPM_PACKAGES=${HOME}/.npm-global-pkg
~~~~

now you need to create you **npm** *configuration file*. Create a file called **.npmrc** in your home directory and add the following line:

~~~~{.language-bash}
	prefix=${HOME}/.npm-global-pkg
~~~~

install **npm**:

~~~~{.language-bash}
	curl -L https://www.npmjs.org/install.sh | sh
~~~~

Once everything is installed, we need to configure all the paths, so *node* and our system can find all files. Edit you **profile** or **bashrc* file and add the following exports:

~~~~{.language-bash}
	export NODE_PATH="$NPM_PACKAGES/lib/node_modules:$NODE_PATH"
	export PATH="$NPM_PACKAGES/bin:$PATH"
~~~~

restart you terminal or **source** your *profile* and you'll be ready to start using **npm** for installing global packages without sudo.