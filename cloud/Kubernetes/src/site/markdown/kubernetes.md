# Kubernetes

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->
## Introduction 




## Quick start & demo

From : 

 * https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/
 * https://github.com/kubernetes/minikube/releases
 

 
 curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.18.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

 
No we need in the VM the virutalBox tools ...

 cd /etc/yum.repos.d
 sudo wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo

 sudo yum install epel-release
 sudo yum --enablerepo=epel install dkms
 
 sudo yum groupinstall "Development Tools"
 sudo yum install kernel-devel
 
 sudo yum install VirtualBox-5.1
 sudo usermod -a -G vboxusers admin

## To go further 

* https://github.com/kelseyhightower/kubernetes-the-hard-way
 
## Formation

### Architecture 

![Architecture overview](./images/architecture.png "Architecture Overview")


### Distributions

* mini kube : only one server 
* kubeadm : several server   
* fourniture standard : gke / azure / aws
* docker entreprise / OpenShift v3

Add-ons Mecanism to extend fonctionnalities 
datadog ? 

### Probing

* in this kind of architecture, the check must be atomic .... a WS should not say not ready if the db is not ready.
* readyness, http, tcp, exec (ready to be use)
* liveness, kube can have a delayed start. 

### common commands and parameters 

* run : use the command line and will create a pod
* apply/create : use the configuration file to create the pod
* delete : allow to delete a resource, --cascade=true|false allow to delete recursively resources.
* logs : allow to see logs from a running container
s
Most of the commands have a -w (watch) parameter. But it most useful to use the standard watch command.
   

#### describes

	kubectl describe po liveness-http

The ouput format can be controlled : ex -o yaml
	

#### explain

	kubectl explain pods.spec.containers.livenessProbe
	
## Architecture 

* Probing
* circuit breaker

## Kube common Elements


### Node 

	it's a physical or virtual server

### POD (PO)
	
	set of containers running, with configuration associated. It's a logical entity. 

### Replica set (RS)

	Allow to control the pods via labels, it's the runtime part of the POD. 

Scaling example : 

	kubectl scale --replicas=1 rs nginx 
    kubectl scale --replicas=10 rs nginx 
    kubectl scale --replicas=3 rs nginx 
	
	
### Daemon set (DS)

	We can see it as a RS which run on all nodes 

### namespace (NS)

	Logical organization that give permissions/quotas to user/pod 

### Jobs 

	Allow to run an action temporary (clean, check, update), and should finish

### Services  

	Set of pods , giving an uniq ip/port access to the outside

### Node PORT, endpoints


### Ingress

	Allow to export a service outside of kube, can also be use to proxy an external access.

### Volume	

	empty dir for temporary sharing file between containers in pod 

#### Persistent volume

	work with claim

#### Volume claim

	static or dynamic for clean association we should use labels

### ConfigMap

Manage configuration (like etcd)
A config Map is a yaml file, key/value, we can also define a file inside. It will generate a properties file inside the pod/container.

### secrets 

not really secrets :)

### Deployment

	auto scalling is possible

## Practices

### Gateway, side containers

It's quite useful to have a small containers, with curl/vi/... with full access to the clean container, in same network, just exposing a shell ...


### Deployment strategy



### Quota

#### LimitRange

  will be apply to the namespaec, can be set by pod too  

### Application design for kube

* immutable
* we can move it
* 12 factors ....  
* Disk data can be lost
* version configuration

### Principles 

	Image immutable principle

	High observabily principle
	
	Process Disposability Principles
	
	Lifecycele conformance principle
	
	Runtime confinement principle
	
	Single concern principle
	
	Self containement principle
	
## Helm

* package management for kubernetes
* contains a template mechanism

### Helm components 

* *Chart* is a package
* *Release* is an instance of Chart*
* *Repository* is a collection of charts
* *Template* 

### File structure 

* standard for all component template
	
	



