# How to move your configuration files to template

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Configuring application

There is many reason why we need to configure application. Among the most common we can say :
	- configure some URL (jdbc pool, web service access, .... ) 
	- configure some behavior : feature enable / disable
	- configure some parameters depending of the system : memory, thread number, pool size, .... 
	- configure some parameters depending of the environment : prod / QA / DEV like certifacts, ...
	
Ages ago, we used to store all these informations in some configuration files. 
When moving these applications to containers (docker or other) we face some challenges.
	
In this document I'll will describe an approach, to make it easier to maintain in the container world, without changing the existing application.
	
This approach will fit pretty well for most of the configuration fields. For some specific case like URL or service I'll favor using directly the service discovery approach like Traeffik as example.
Of if you plan to move to k8S, to use mechanism provided.
Note : If you plan to move to K8S, simple replace the docker compose section by using the k8S POD mechanism.

## Useful links 

 * https://docs.docker.com/compose/compose-file/
 * https://docs.docker.com/storage/volumes/#create-and-manage-volumes
 

 
	
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

### Going further with confd 


#### building conditional step 

The confd syntax is a go template, so you can use the if then else syntax.

	{{if getenv "DUDE"}}  DUDE exist {{else}} DUDE n'existe pas {{end}}


#### Using the range functionnality 

Often, you have a set of blocks that depends of the environment. A standard example, is the ldap servers definition in an apache configuration file. In dev you may have only the entreprise ldap, and in production you may have several.
To deals with such situation a way is to use the range functionnality 

First define in the etcd server some settings for 3 ldap servers, with different IP and authentification procedure 

	etcdctl set /dev/myScript/services/ldap/example/IP 10.1.1.1
	etcdctl set /dev/myScript/services/ldap/example/authent custom

	etcdctl set /dev/myScript/services/ldap/prod/IP 10.2.2.2
	etcdctl set /dev/myScript/services/ldap/prod/authent password

	etcdctl set /dev/myScript/services/ldap/common/IP 10.12.45.56
	etcdctl set /dev/myScript/services/ldap/common/authent encrypted_file

	
Secondly, let's define the template :

     #FIRST we do a loop on the children of the ldap directory in the etcd configuration : 3 children : example, prod, common
	{{range $ldapname := (lsdir "/myScript/services/ldap/")}}

		#HTTPD conf we want to generate
		<ldap_configuration ldap_server_name="{{.}}">
		    #to ease reading, display the value we are looping over 
			<serverName>  {{.}} </servername>
			
			#create a variable : IP with the value /myScript/services/ldap/prod/IP as example
			#                    this is because getv does not support "/myScript/services/ldap/$ldapname/IP" syntax.
			# and then, we got the value for this variable in the etcd server
			{{ $ip := printf "/myScript/services/ldap/%s/IP" $ldapname }} <IP> {{getv $ip  }} </IP>
			
			#we do the same for the authentification method
			{{ $method := printf "/myScript/services/ldap/%s/authent" $ldapname }} <Authent medod>   {{getv $method}} </Authent_method>
		<ldap/>
	{{end}}

3rd step, run it, and look at the output 

	<ldap_configuration ldap_server_name="common">
		<serverName>  common </servername> 
		<IP> 10.12.45.56 </IP>
		<Authent medod>   encrypted_file </Authent_method>
	<ldap/>

	<ldap_configuration ldap_server_name="example">
		<serverName>  example </servername> 
		<IP> 10.1.1.1 </IP>
		<Authent medod>   custom </Authent_method>
	<ldap/>

	<ldap_configuration ldap_server_name="prod">
		<serverName>  prod </servername> 
		<IP> 10.2.2.2 </IP>
		<Authent medod>   password </Authent_method>
	<ldap/>


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

	ARG UBUNTU_VERSION=18.04
	FROM ubuntu:${UBUNTU_VERSION}

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

	ARG UBUNTU_VERSION=18.04
	FROM ubuntu:${UBUNTU_VERSION}

	ADD confd /tmp/confd
	ADD conf/confd.toml /etc/confd/confd.toml
	ADD conf/myScript.conf.toml /etc/confd/conf.d/myScript.conf.toml
	ADD conf/myScript.conf.tmpl /etc/confd/templates/myScript.conf.tmpl
	ADD myScript.sh /tmp/myScript.sh


    CMD /tmp/confd -onetime -backend etcd  && cat /tmp/myScript.conf && /tmp/myScript.sh

 	
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

First, we saw the output of the "docker exec ..... cat /tmp/myScript.conf", in order to see the configuration file is correctly updated.
And then, the output of the script. 

Every thing is fine.

### To go further in this way.

In order to go further, you should not run confd in "once" mode, but in "watch" mode. It will automatically call the reload on your application if the config file change due to some new values in etcd.

IF your configuration is not changing, you can use a multi stage build, to avoid having confd in your final image (	https://docs.docker.com/develop/develop-images/multistage-build/)

 	
### Some docker /Containers  comments 

From a containers point of view, this is not perfect. Indeed, in the image we have two processes running: the confd daemon and the application itsfelf.
We may have some situation where the confd daemon die, and nobody spot it. So you may expect your configuration to change, but nothing happen.

A better way to do it, it's to have a side-car container with the confd daemon, using containers volume to share the resulting configuration file.

Note : you can use this approach without confd to inject your configuration in your container.

#### General idea 

The key idea is to do something like 

New docker file : 
	FROM ubuntu:18.04
	
	ADD myScript.sh /tmp/myScript.sh
	
	CMD cat /tmp/conf/myScript.conf && /tmp/myScript.sh


Command :
	docker run -v /tmp/myScript.conf:/tmp/conf/myScript.conf b6bff9f40840

Output : 	
	#Generated automatically by confd at 2018-06-25 15:57:37.914457599 +0200 CEST m=+0.010121250
	DUDE=BETTY
	SUBJECT=RAPHAEL
	VERB=is
	PARENT_RELATIONSHIP=brother
	
	Hello BETTY !
	RAPHAEL is your brother

#### Using dcoker compose to prepare the side-car container and the volume to share the data  	


First, create a simple docker-compose.yml file 
	
	version: '3'
	services:
		script:
			build: .
			volumes :
				- /tmp/myScript.conf:/tmp/conf/myScript.conf


And then run *docker-compose -f docker-compose.yml up

	...
	script_1  | #Generated automatically by confd at 2018-06-25 15:57:37.914457599 +0200 CEST m=+0.010121250
	script_1  | DUDE=BETTY
	script_1  | SUBJECT=RAPHAEL
	script_1  | VERB=is
	script_1  | PARENT_RELATIONSHIP=brother
	
	script_1  | Hello BETTY !
	script_1  | RAPHAEL is your brother
	
	bin_script_1 exited with code 0


#### Using a side-car container and docker compose.

Now, let's do it with a real side car container 


The confd container :

	ARG UBUNTU_VERSION=18.04
	FROM ubuntu:${UBUNTU_VERSION}

	ADD confd /app/confd
	ADD conf/confd.toml /etc/confd/confd.toml
	ADD conf/myScript.conf.toml /etc/confd/conf.d/myScript.conf.toml
	ADD conf/myScript.conf.tmpl /etc/confd/templates/myScript.conf.tmpl

	CMD /app/confd -onetime -backend etcd  && cat /tmp/myScript.conf && sleep 20 

The simple script is the same. I'm just updating the compose file :

	version: '3'
	services:
		script:
			build: .
			volumes :
				- /tmp/:/tmp/conf/
		confd:
			build:
			context: .
			dockerfile: Dockerfile.confd
			volumes :
				- /tmp/:/tmp
			network_mode: host

I added the network element to access my etcd server, and i changed the volumes of the first container to allow updates on the configuraiton file.

Now, if i'm running *docker-compose -f docker-compose.yml up* it will output : 

	confd_1   | 2018-06-26T12:29:15Z localhost.localdomain /tmp2/confd[7]: DEBUG Overwriting target config /tmp/myScript.conf
	confd_1   | 2018-06-26T12:29:15Z localhost.localdomain /tmp2/confd[7]: DEBUG Running echo done
	confd_1   | 2018-06-26T12:29:15Z localhost.localdomain /tmp2/confd[7]: DEBUG "done\n"
	confd_1   | 2018-06-26T12:29:15Z localhost.localdomain /tmp2/confd[7]: INFO Target config /tmp/myScript.conf has been updated
	confd_1   | #Generated automatically by confd at 2018-06-26 12:29:15.428570512 +0000 UTC m=+0.012612208
	confd_1   | DUDE=BETTY
	confd_1   | SUBJECT=RAPHAEL
	confd_1   | VERB=is
	confd_1   | PARENT_RELATIONSHIP=brother


	script_1  | #Generated automatically by confd at 2018-06-26 12:29:15.428570512 +0000 UTC m=+0.012612208
	script_1  | DUDE=BETTY
	script_1  | SUBJECT=RAPHAEL
	script_1  | VERB=is
	script_1  | PARENT_RELATIONSHIP=brother
	script_1  | Hello BETTY !
	script_1  | RAPHAEL is your brother
	bin_script_1 exited with code 0

We can noticed the generation time of the configuration file used by the container 'script_1'

#### Using a volume 

The new docker compose is : 
(note : i'm using the 3.3 syntax for volume)

	version: '3.3'
	volumes:
        mydata:

	services:
		script:
			build: .
			volumes :
				- type: volume
				  source: mydata
				  target: /tmp/conf
		confd:
			build:
				context: .
				dockerfile: Dockerfile.confd
			volumes :
				- type: volume
				  source: mydata
				  target: /tmp

    network_mode: host



The output is : 

	confd_1   | 2018-06-26T12:44:46Z localhost.localdomain /tmp2/confd[7]: DEBUG Overwriting target config /tmp/myScript.conf
	confd_1   | 2018-06-26T12:44:46Z localhost.localdomain /tmp2/confd[7]: DEBUG Running echo done
	confd_1   | 2018-06-26T12:44:46Z localhost.localdomain /tmp2/confd[7]: DEBUG "done\n"
	confd_1   | 2018-06-26T12:44:46Z localhost.localdomain /tmp2/confd[7]: INFO Target config /tmp/myScript.conf has been updated
	confd_1   | #Generated automatically by confd at 2018-06-26 12:44:46.741580849 +0000 UTC m=+0.013764417
	confd_1   | DUDE=BETTY
	confd_1   | SUBJECT=RAPHAEL
	confd_1   | VERB=is
	confd_1   | PARENT_RELATIONSHIP=brother

	script_1  | #Generated automatically by confd at 2018-06-26 12:44:46.741580849 +0000 UTC m=+0.013764417
	script_1  | DUDE=BETTY
	script_1  | SUBJECT=RAPHAEL	
	script_1  | VERB=is
	script_1  | PARENT_RELATIONSHIP=brother
	script_1  | Hello BETTY !
	script_1  | RAPHAEL is your brother


### Let mixed this approach with using the flexibility of environment variables 

I'm changing a little bit my script to read first the environment variable, else the value got in the config file, if nothing found in the two first attempt, it will set a default value.

	#!/bin/bash

	source /tmp/conf/myScript.conf

    DUDE=${DUDE:-"Xtof"}
    SUBJECT=${SUBJECT:-"She"}
    VERB=${VERB:-"is"}
    # AS i'm lazy, just doing it for the last parameter.
	PARENT_RELATIONSHIP=${PARENT_RELATIONSHIP:${PARENT_RELATIONSHIP_FROM_CONFIG:-"uncle"}}

	echo "Hello ${DUDE} !"
	echo "${SUBJECT} ${VERB} your ${PARENT_RELATIONSHIP}"


Changing the compose file :

	version: '3.3'
	volumes:
        mydata:

	services:
		script:
			build: .
			environment:
				- PARENT_RELATIONSHIP=father in law
			volumes :
				- type: volume
				  source: mydata
				  target: /tmp/conf
		confd:
		build:
			context: .
			dockerfile: Dockerfile.confd
		volumes :
			- type: volume
			  source: mydata
			  target: /tmp

		network_mode: host

Now, a run of these images will give :

	confd_1   | 2018-06-26T13:44:24Z localhost.localdomain /tmp2/confd[8]: DEBUG Overwriting target config /tmp/myScript.conf
	confd_1   | 2018-06-26T13:44:24Z localhost.localdomain /tmp2/confd[8]: DEBUG Running echo done
	confd_1   | 2018-06-26T13:44:24Z localhost.localdomain /tmp2/confd[8]: DEBUG "done\n"
	confd_1   | 2018-06-26T13:44:24Z localhost.localdomain /tmp2/confd[8]: INFO Target config /tmp/myScript.conf has been updated
	confd_1   | #Generated automatically by confd at 2018-06-26 13:44:24.4423676 +0000 UTC m=+0.010820386
	confd_1   | DUDE=BETTY
	confd_1   | SUBJECT=RAPHAEL
	confd_1   | VERB=is
	confd_1   | PARENT_RELATIONSHIP_FROM_CONFIG=brother

	script_1  | #Generated automatically by confd at 2018-06-26 13:44:24.4423676 +0000 UTC m=+0.010820386
	script_1  | DUDE=BETTY
	script_1  | SUBJECT=RAPHAEL
	script_1  | VERB=is
	script_1  | PARENT_RELATIONSHIP_FROM_CONFIG=brother
	script_1  | Hello BETTY !
	script_1  | RAPHAEL is your father in law


In this case, the variable in the docker compose file is not critical, but the value may differ from one environment to another one. So, it's not perfet.
	

### Conclusion on this approach.
	
This approach feet pretty well for old fashion application, with a static configuration file.
In general I will encourage to avoid duplication :
 - several docker file (prod/QA/DEV) is a duplication 
 - several compose file is a duplication 
.... 

So, in this first part, I've avoid duplication by using the dev/prod/QA prefix in the variables defined in etcd, and by using the same image all over the environments.

An other way to avoid this duplicatoin is to use docker-app. Let's see it quickly.

## Docker App

You can have more information here : https://github.com/docker/app

The objective is to make easier to maintain the environment value for the different environments.


### First, initialisation of the docker app from the compose file

	docker-app init --single-file myScript

It generate a myScript.dockerApp file with the following content.
Note : i've manually extract the variable : PARENT_RELATIONSHIP_COMPOSE in the setting section. 


	# This section contains your application metadata.
	# Version of the application
	version: 0.1.0
	# Name of the application
	name: myScript
	# A short description of the application
	description: 
	# Namespace to use when pushing to a registry. This is typically your Hub username.
	#namespace: myHubUsername
	# List of application maitainers with name and email for each
	maintainers:
	- name: ouelcum
	  email: 
	# Specify false here if your application doesn't support Swarm or Kubernetes
	targets:
		swarm: true
		kubernetes: true

	--
	# This section contains the Compose file that describes your application services.
	version: '3.3'
	volumes:
        mydata:

	services:
		script:
			build: .
			environment:
				- PARENT_RELATIONSHIP=${PARENT_RELATIONSHIP_COMPOSE}
			volumes :
				- type: volume
				source: mydata
				target: /tmp/conf
		confd:
			build:
				context: .
				dockerfile: Dockerfile.confd
			volumes :
				- type: volume
				source: mydata
				target: /tmp

			network_mode: host

	--
	# This section contains the default values for your application settings.
	PARENT_RELATIONSHIP_COMPOSE: father in law


### Render it 

Now, let render it to generate the compose file automatically

	>$ docker-app render
	version: "3.3"
	services:
	confd:
		build:
			context: .
			dockerfile: Dockerfile.confd
		network_mode: host
		volumes:
			- type: volume
			source: mydata
			target: /tmp
	script:
		build:
			context: .
		environment:
			PARENT_RELATIONSHIP: father in law
		volumes:
			- type: volume
			source: mydata
			target: /tmp/conf
	volumes:
		mydata: {}

So, that's good, at this point i'm able to generate my compose file from the template.

### extracting variables in a settings file specific per environment 

the new myScript.dockerApp is :

	...
	services:
		script:
			build: .
			environment:
				- PARENT_RELATIONSHIP=${PARENT_RELATIONSHIP_COMPOSE}
			volumes :
				- type: volume
				source: mydata
				target: /tmp/conf
		confd:
			build:
				context: .
				dockerfile: Dockerfile.confd
			volumes :
				- type: volume
				source: mydata
				target: /tmp

			network_mode: host

	--
	# This section contains the default values for your application settings.
	# 
	{}

The dev setting file is : 

	PARENT_RELATIONSHIP_COMPOSE: father in law

The prod setting file is : 

	PARENT_RELATIONSHIP_COMPOSE: grand father

Now, let's render the files 
For the dev : 

	> docker-app render -f dev.yaml 
	version: "3.3"
	services:
		confd:
			build:
				context: .
				dockerfile: Dockerfile.confd
			network_mode: host
			volumes:
				- type: volume
				source: mydata
				target: /tmp
		script:
			build:
				context: .
				environment:
					PARENT_RELATIONSHIP: father in law
				volumes:
					- type: volume
					source: mydata
					target: /tmp/conf
		volumes:
			mydata: {}

For the production 
		
	> docker-app render -f prod.yml 
	version: "3.3"
	services:
		confd:
			build:
				context: .
				dockerfile: Dockerfile.confd
			network_mode: host
			volumes:
				- type: volume
				source: mydata
				target: /tmp
		script:
			build:
				context: .
			environment:
				PARENT_RELATIONSHIP: grand father
			volumes:
				- type: volume
				source: mydata
				target: /tmp/conf
	volumes:
		mydata: {}

	
## Conclusion 

We have mixed several approach, static and dynamic configuration. USing a key value store and  environment variables.

Depending of your situation one, the other or both approache can fit. So use the one that minimise duplication and ease maintenance.


HAve a nice day 


	
