# Nexus 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Creating a yum mirror with nexus

 First mirror :  http://mirror.centos.org/centos/

In the /etc/yum.repos.d/, create a file named nexus.repo and configure the  base as : 
 
 http://mirror.centos.org/centos/$version/os/$arch
 
 Remove all the others repo file, and then you can run a yum update, it will feed your cache in nexus.