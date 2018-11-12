# ETCD 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


## INSTALLATION

http://play.etcd.io/install

To run it in a local mode, simple run :

	etcd
	
The default have persistent datastore.

## USAGE and connection with confd

in  **/etc/confd/confd.toml**

	backend = "etcd"
	confdir = "/etc/confd"
	log-level = "debug"
	interval = 600
	nodes = [
		"http://127.0.0.1:2379",
	]
	noop = false
	prefix = "/dev"


## Version 2 and version 3

 warning, there's some trap, as both datamodel exist in memory

 * https://hackernoon.com/notes-on-moving-from-etcd2-to-etcd3-dedb26057b90
 * https://coreos.com/blog/announcing-etcd-3.3
 
 * RUN confd -onetime -backend etcdv3 -node ${ETCD_ENDPOINTS}

	

## GET and SET Value

**SET**
	../etcd/etcdctl set /dev/services/example/text father2
	../etcd/etcdctl set /dev/services/example/foo father2

**GET**

	../etcd/etcdctl get /dev/services/example/ 


## DELETE values


 Warning we need to do the export to be in version 3 of the API.
 
    #warning if you use API V3 for deletion, you'll use it also for put and get.
	#if you put in V3, you can not get it in V2..... 
	#you need also to configure confd to use it in V3.
	
	export ETCD_API=3  

	../etcd/etcdctl del /dev/services/example/foo \\0    //to del all subtree, it's the index position 
	
	
	
## installing a local cluster

This will start a cluster with 3 nodes, and a proxy to redirect traffik.

	etcd1: bin/etcd --name infra1 --listen-client-urls http://127.0.0.1:2379 --advertise-client-urls http://127.0.0.1:2379 --listen-peer-urls http://127.0.0.1:12380 --initial-advertise-peer-urls http://127.0.0.1:12380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof --logger=zap --log-outputs=stderr
	etcd2: bin/etcd --name infra2 --listen-client-urls http://127.0.0.1:22379 --advertise-client-urls http://127.0.0.1:22379 --listen-peer-urls http://127.0.0.1:22380 --initial-advertise-peer-urls http://127.0.0.1:22380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof --logger=zap --log-outputs=stderr
	etcd3: bin/etcd --name infra3 --listen-client-urls http://127.0.0.1:32379 --advertise-client-urls http://127.0.0.1:32379 --listen-peer-urls http://127.0.0.1:32380 --initial-advertise-peer-urls http://127.0.0.1:32380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof --logger=zap --log-outputs=stderr
	#proxy: bin/etcd grpc-proxy start --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --listen-addr=127.0.0.1:23790 --advertise-client-url=127.0.0.1:23790 --enable-pprof

	
	