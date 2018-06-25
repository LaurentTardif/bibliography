# How to move your configuration files to template

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Simple running application

The objective of this tutorial is to migrate from an old fashion hard coded script, 
to a docker image, where we can customized easily the message displayed.

This will be done in several baby step :
 * Extract variables from the script
 * Create a configuration files
 * Create a template for the configuration file
 

### Initial script

Let's start with a simple application (a bash script) that display a text.
This text is hard coded in the script.

The source code of the script is here :

	#!/bin/bash

	echo "Hello Luke !"
	echo "I am your Daddy"
	
Without too many surpise, if I run the script i'll have the following output : 

	Hello Luke !
	I am your Daddy


	
### Extracting variables

The first step, is to extract the values displayed to make it easier to change.

The new version of the script is now :


	#!/bin/bash
	DUDE="Luke"
	SUBJECT="I"
	VERB="am"
	PARENT_RELATIONSHIP="father"
	
	echo "Hello ${DUDE} !"
	echo "${SUBJECT} ${VERB} your ${PARENT_RELATIONSHIP}"

Of course at any step you'll be able to run the tests or the application to ensure non regression.

### Creating configuration file 


#### first step, simple extraction

The configuration file will be named "myScript.conf", with the following content :

	DUDE="Luke"
	SUBJECT="I"
	VERB="am"
	PARENT_RELATIONSHIP="mother"

The script file is now : 

	#!/bin/bash

	source myScript.conf	
	echo "Hello ${DUDE} !"
	echo "${SUBJECT} ${VERB} your ${PARENT_RELATIONSHIP}"

When running the script I got 

	Hello Luke !
	I am your mother
	
	
#### Second step, making it a bit more generic and robust	
	
	
The script file is now : 

	#!/bin/bash
	source myScript.conf	
	
	DUDE=${DUDE:-"Xtof"}
	SUBJECT=${SUBJECT:-"She"}
	VERB=${VERB:-"is"}
	PARENT_RELATIONSHIP=${PARENT_RELATIONSHIP:-"uncle"}
	
	echo "Hello ${DUDE} !"
	echo "${SUBJECT} ${VERB} your ${PARENT_RELATIONSHIP}"

What's the benefits of this extra work ? 

1) It make an indirection between the variables names in the configuration file and their usage in the script. It's more robust against any typos in the configuration file.

2) It allow to define default values for all the variables. Allowing partial configuration file. 

3) With the two previous advantage, i can now define variables in the environment and use it in my script. Making debugging and tests easier.


If I run the script, with the previous configuration file 

	Hello Luke !
	I am your mother

Now, I'm deleting my configuration file, and run again the script	
	
	> myScript.conf: Aucun fichier ou dossier de ce type
	
	Hello Xtof !
	She is your uncle

If I'm defining some variables in the shell 
	
	> export SUBJECT="Louis"
    ./myScript.sh
	> myScript.conf: Aucun fichier ou dossier de ce type

	Hello Xtof !
	Louis is your uncle
 	
## Using confd to feed the configuration file
	
We will not cover how to install and configure confd. It's pretty simple and lot of informations can be found here (https://github.com/kelseyhightower/confd).


To make if work with confd, I will create : a template, and fill it with confd.

### First create the template for the resource

The resource is describe in a file located in the */etc/confd/conf.d* directory by default.
	
Let create the file myScript.conf.toml

	[template]
	src = "myScript.conf.tmpl"
	dest = "/etc/myScript/myScript.conf"
	mode = "0644"
	keys = [
		"/dev/myScript",
	]
	check_cmd = "cat {.src}}"
	reload_cmd = "echo reloaded"

### Create the template configuration file to be used

The full filename is : /etc/confd/templates/myScript.conf.tmpl


	#Generated automatically by confd at {{datetime}}
	DUDE={{getenv "DUDE" "default_value_DUDE"}}
	SUBJECT={{getv "SUBJECT" "default_value_SUBJECT"}}
	VERB={{getv "VERB" "default_value_VERB"}}
	PARENT_RELATIONSHIP={{getenv "PARENT_RELATIONSHIP" "default_value_PARENT"}}
	
### Feeding the configuration file with confd

First, let's define some environment variables as confd will use it (I'm using getenv)

	export DUDE=Antoine
	export VERB=IS
	export SUBJECT=Betty
	export PARENT_RELATIONSHIP=Aunt
	
Let's run confd once, using environment variables 

	../bin/confd -onetime -backend env ; cat /tmp/myScript.conf

it will output 

	#Generated automatically by confd at 2018-06-25 15:39:16.504324805 +0200 CEST m=+0.015080019
	DUDE=Antoine
	SUBJECT=Betty
	VERB=IS
	PARENT_RELATIONSHIP=Aunt
	
Now, let's run the script 

	Hello Antoine !
	Betty is your Aunt

### Intermediate conclusion

From this point, we can start to put this logic in a docker file, and we have images than we can easily configure.
But we'll go a bit further and we are going to use ETCD as key values store to provide values to confd.

## Using etcd

### First, linking etcd and confd

Updating the confd configuration file : **/etc/confd/confd.toml**

	backend = "etcd"
	confdir = "/etc/confd"
	log-level = "debug"
	interval = 600
	nodes = [
		"http://127.0.0.1:2379",
	]
	noop = false
	prefix = "dev"

Please note the prefix, that will allow me to isolate the environments	
	
### updating the configuration template

For memory the file is located at : /etc/confd/templates/myScript.conf.tmpl

	#Generated automatically by confd at {{datetime}}
	DUDE={{getv "/myScript/DUDE" "default_value_DUDE"}}
	SUBJECT={{getv "/myScript/SUBJECT" "default_value_SUBJECT"}}
	VERB={{getv "/myScript/VERB" "default_value_VERB"}}
	PARENT_RELATIONSHIP={{getv "/myScript/PARENT_RELATIONSHIP" "default_value_PARENT"}}

Please note the prefix, it will allow to isolate key per application easily in the etcd	

###Updating the resource file

For memory the file is located at **/etc/confd/conf.d/myScript.conf.toml** 

	[template]
	src = "myScript.conf.tmpl"
	dest = "/tmp/myScript.conf"
	mode = "0644"
	keys = [
		"/myScript",
	]
	check_cmd = "cat {{.src}}"
	reload_cmd = "echo done"

Please note the *keys* section allowing to get the keys from etcd. 
	
### Setting values in etcd

	etcdctl set "/dev/myScript/DUDE" BETTY
	etcdctl set "/dev/myScript/SUBJECT" Raphael
	etcdctl set "/dev/myScript/VERB" Is
	etcdctl set "/dev/myScript/PARENT_RELATIONSHIP" brother

Now, let's run confd once, to update values
	
    confd -onetime -backend etcd ; cat /tmp/myScript.conf 
 
and then , we can now run the script and get the following output

	Hello BETTY,
	Raphael is your brother !
	
### Intermediate conclusion

Now, we have a template mechanism set up. This mechanism allow to get values from a key store server.
It will allow to define different values  according to the environment (dev, integration, QA, prod, ...). 	
	

## Use it with docker

### Simple way 

Let's do a simple docker file to see how it works

Dockerfile :

	FROM ubuntu:18.04

	ADD myScript.sh /tmp/myScript.sh
	ADD myScript.conf /tmp/myScript.conf

	CMD /tmp/myScript.sh

####   Let remove a configuration 

myScript.conf :

	SUBJECT=RAPHAEL
	VERB=is
	PARENT_RELATIONSHIP=brother
	

#### Build it and run it 


	sudo docker build .  
	Sending build context to Docker daemon  5.859MB
	Step 1/4 : FROM ubuntu:18.04
	---> 113a43faa138
	Step 2/4 : ADD myDarkScript.v5.sh /tmp/myScript.sh
	---> Using cache
	---> e33cab98f1be
	Step 3/4 : ADD myScript.conf /tmp/myScript.conf
	---> 7790a4ddc73a
	Step 4/4 : CMD /tmp/myScript.sh
	---> Running in 82edd519e384
	Removing intermediate container 82edd519e384
	---> f79114a72db3
	Successfully built f79114a72db3
	
No, run it 

	sudo docker run f79114a72db3
	Hello Xtof !
	RAPHAEL is your brother

	
The Xtof is the default value in the script, for the key "DUDE".

From this point, i can configure easily my container by passing arguments at run time 

	sudo docker run -e DUDE=David f79114a72db3
	Hello David !
	RAPHAEL is your brother

There are several limitations to such approachs. Mainly it's difficult to maintain if you have many configuration to pass (btw, it's bad anyway), or to maintain consistency among all environments.


### Now, let's do it with etcd and confd²	
	
#### Adding ETC / CONFD to the image

 	
#### Let's run it 

I need to allow my container to access my host, in order to use the ETCD server on it .... (that's for demo purpose)


	sudo docker run --network host c5bc6d742f11


	#Generated automatically by confd at 2018-06-25 15:24:40.635747728 +0000 UTC m=+0.012251764
	DUDE=BETTY
	SUBJECT=RAPHAEL
	VERB=is
	PARENT_RELATIONSHIP=brother
	
	Hello BETTY !
	RAPHAEL is your brother

First, we saw the output of the "cat /tmp/myScript.conf", in order to see the configuration file is correctly updated.
And then, the output of the script. 

Every thing is fine.

## To go further

In order to go further, you should not run confd in "once" mode, but in "watch" mode. It will automatically call the reload on your application if the config file change due to some new values in etcd.
 	

	


	
