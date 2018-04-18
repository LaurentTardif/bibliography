# PuppetChocolatey

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Installing the Chocolatey module on the Puppet master

First, let us install the Windows puppet module pack onto our Puppet master, which includes the Chocolatey module. This will enable us to create Puppet classes where Chocolatey will be used to deploy packages. In this setup, I have a Puppet master with the hostname "puppet-master" and a Windows client with the hostname 'win-test-1". Please note I assume your Puppet master is been configured.

	[root@puppet-master ~]# puppet module install puppetlabs/windows

As you can see, there is now a Chocolatey module on my Puppet master:

	[root@puppet-master /]# ls -l /etc/puppetlabs/code/environments/production/modules/chocolatey/
	total 104
	-rw-r--r--. 1 puppet root  1027 Apr  3 20:47 appveyor.yml
	-rw-r--r--. 1 puppet root  7579 Apr  3 20:47 CHANGELOG.md
	-rw-r--r--. 1 puppet root  8491 Apr  3 20:49 checksums.json
	-rw-r--r--. 1 puppet root  7942 Apr  3 20:47 CONTRIBUTING.md
	drwxr-xr-x. 2 puppet root    21 Apr  3 20:49 examples
	-rw-r--r--. 1 puppet root  7871 Apr  3 20:47 Gemfile
	drwxr-xr-x. 5 puppet root    50 Apr  3 20:49 lib
	-rw-r--r--. 1 puppet root 11358 Apr  3 20:47 LICENSE
	drwxr-xr-x. 2 puppet root    25 Apr  3 20:49 locales
	-rw-r--r--. 1 puppet root   182 Apr  3 20:47 MAINTAINERS.md
	drwxr-xr-x. 2 puppet root    93 Aug 31 13:23 manifests
	-rw-r--r--. 1 puppet root  1300 Apr  3 20:49 metadata.json
	-rw-r--r--. 1 puppet root   774 Sep 23  2016 NOTICE
	-rw-r--r--. 1 puppet root  6203 Apr  3 20:47 Rakefile
	-rw-r--r--. 1 puppet root 29704 Jan  3  2017 README.md
	drwxr-xr-x. 5 puppet root    73 Apr  3 20:49 spec
	drwxr-xr-x. 2 puppet root    39 Apr  3 20:49 templates
	drwxr-xr-x. 6 puppet root    84 Apr  3 20:49 tests

## Creating a Chocolatey package Puppet module

On our Puppet master, I want to create a module specifically for installing "Git" on my Windows nodes. To do this I perform the following steps:

    Create a module directory "gitinstall" with a manifests subfolder.
    Create a "gitinstall" Puppet class within the gitinstall/manifests/init.pp file.
    Update the site.pp manifest so that "win-test-1" installs the Chocolatey "Git" package.

On puppet-master, create the module directory for "gitinstall":

	[root@puppet-master ~]#  mkdir -p /etc/puppetlabs/code/environments/production/modules/gitinstall/manifests

In the manifests folder we created the file "init.pp," which will have our "gitinstall" class inside. We specify that Chocolatey should be the package provider (it defaults to Windows), and that Chocolatey should ensure "Git" is installed on the node.

	[root@puppet-master ~]# vi /etc/puppetlabs/code/environments/production/modules/gitinstall/manifests/init.pp

	class gitinstall {
		package { 'git':
			ensure   => installed,
			provider => 'chocolatey',
		}
	}

Now we create a node definition in our "site.pp" manifest for "win-test-1" and add the chocolatey and gitinstall modules using "include":

	[root@puppet-master ~]#  vi /etc/puppetlabs/code/environments/production/manifests/site.pp

	node 'win-test-1'{
		include chocolatey
		include gitinstall
	}

Basically, what the "site.pp" is stating is that when "win-test-1" pulls its configuration from the Puppet master and applies it to itself, it should ensure "Chocolatey" and "Git" are installed. In a production scenario, you would want to configure Chocolatey further so that it at least points to an internal package source since by default it will pull packages from the Chocolatey public repository.
Installing a Chocolatey package on a Windows node

Now comes the easy part, simply running puppet agent –t on "win-test-1" inside PowerShell, which will apply the Puppet configuration. As you can see below the Puppet agent installed "Chocolatey" and then "Git".

	PS C:> puppet agent -t
	Info: Retrieving pluginfacts
	Info: Retrieving plugin
	Info: Loading facts
	Info: Caching catalog for win-test-1
	Info: Applying configuration version '1504208593'
	Notice: /Stage[main]/Chocolatey::Install/Exec[install_chocolatey_official]/returns: executed successfully
	Notice: /Stage[main]/Gitinstall/Package[git]/ensure: created
	Notice: Finished catalog run in 56.97 seconds
