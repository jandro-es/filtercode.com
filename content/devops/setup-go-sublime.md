Title: Setup Sublime for Go development in OSX.
Date: 2015-08-26 10:20
Category: devops
Tags: devops, go, golang, sublime
Slug: setup-go-sublime
Authors: jandro_es
Summary: How to install **Go** in OSX and setup **Sublime Text** for Go development. 

In this case we're not going to use *Ansible/Vagrant virtual machines*, as **Go** programs are intended to be compiled in almost any platform, we don't need the overhead of using virtual machines.

####Install Go and related packages

First we'll use **Homebrew** to install Go:

~~~~{.language-bash}
	brew install go --cross-compile-common
~~~~

If  you already have brew installed, remember to update it's recipes with:

~~~~{.language-bash}
	brew update
~~~~

Once this is finished, we already have **Go** in our system. As you probably know, Go relies a lot in *convention over configuration*, so let´s follow those conventions and set up Go's paths. All your Go code and all the imported Go code will reside inside what it's called a **workspace** which in our case will be a directory in the file system. Go ahead and create a folder for all the code. For example:

~~~~{.language-bash}
	mkdir /Users/username/sourcecode/go
~~~~

Now we need to tell the system that all Go's code will be in that folder we´ve just created. We edit our *.profile* or *.bash_profile* and add this lines:

~~~~{.language-bash}
	export GOPATH=/Users/username/sourcecode/go
	export PATH=$PATH:$GOPATH/bin
~~~~

we reload our configuration file with:

~~~~{.language-bash}
	source .profile
~~~~

Now all our paths should be properly setup. We check that we have the intended version:

~~~~{.language-bash}
	go version
~~~~

the output should be similar to:

~~~~{.language-bash}
	go version go1.5 darwin/amd64
~~~~

We'll install the following packages:

~~~~{.language-bash}
	go get golang.org/x/tools/cmd/godoc
	go get golang.org/x/tools/cmd/vet
	go get golang.org/x/tools/cmd/goimports
	go get golang.org/x/tools/cmd/oracle
	go get github.com/golang/lint/golint
~~~~

####Install Sublime Text packages

This post explains how to setup **Sublime Text version 3** for Go develoment (with the most common packages, there are a lot more than the ones explained here), if you are using version 2 the steps should be quite similar.

Once we have installed *Sublime Text 3* and **Sublime Package Manager** we´ll install the packages **GoSublime** and **GoOracle**. Open the package manager interface with **cmd+shift+p** and search for **Package Control: Install package**, once the list of packages appears search for *GoSublime* and install it. Do the same for *GoOracle*.

####Setup Sublime´s Go packages

For setting up *GoSublime* we go to **Sublime Text -> Preferences -> Package Settings -> Go Sublime -> Settings - User** and replace the contents with this. This configuration will **lint** and **run** your tests everytime you save. You can tweak this configuration as you like to execute the commands you desire.

~~~~{.language-python}
	{
		"env": {"GOPATH": "/Users/username/sourcecode/go"},
		"fmt_cmd": ["goimports"],
		"on_save": [{
	    "cmd": "gs9o_open", "args": {
	    "run": ["sh", 
	        "go build . && go test -i && go vet && golint ."],
	    "focus_view": false
	}}],
	"autocomplete_closures": true,
	"complete_builtins": true,
	}
~~~~

Other Sublime packages that I use are:

- [**Emmet**](http://emmet.io/download/) for **HTML/CSS**
- [**Git**](https://github.com/kemayo/sublime-text-git) & [**GitGutter**](https://github.com/jisaacks/GitGutter) for managing **Git**
- [**Markdown Preview**](https://github.com/revolunet/sublimetext-markdown-preview) for **Markdown**
- [**Terminal**](http://wbond.net/sublime_packages/terminal) for **Terminal** access

You're ready to create awesome **Go** programs.
