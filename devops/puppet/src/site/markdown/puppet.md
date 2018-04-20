# puppet 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


## puppet family

* **puppet discovery** (facter ? )  : The first step in your journey to pervasive automation is knowing what you have. Puppet Discovery shows you everything that s running across your on-premises and cloud infrastructure, and even what s in your containers, and lets you take action to bring it all under management.
* **pupper entreprise** : puppet :)
* **Puppet Pipelines** makes continuous delivery easy by meeting you where you are. It automates the way you build and deploy applications and containers running on any cloud and on prem.
* **Puppet Bolt** Troubleshoot systems, deploy one-off changes, and execute sequenced actions in a procedural way.

## Architecture overview 

### global

![Architecture overview](./images/puppetArchi.png "Architecture Overview")

### Run 

![run overview](./images/PuppetRun.jpg "Run")


### Catalogs

A catalog is a document that describes the desired system state for one specific computer. It lists all of the resources that need to be managed, as well as any dependencies between those resources.

Puppet configures systems in two stages:

    Compile a catalog.
    Apply the catalog.

To compile a catalog, Puppet uses several sources of information. For more info, see the pages on basics of the Puppet language and catalog compilation.

### The agent/master architecture

When set up as an agent/master architecture, a Puppet master server controls the configuration information, and each managed agent node requests its own configuration catalog from the master.

In this architecture, managed nodes run the Puppet agent application, usually as a background service. One or more servers run the Puppet master application, Puppet Server.

Periodically, each Puppet agent sends facts to the Puppet master, and requests a catalog. The master compiles and returns that node s catalog, using several sources of information it has access to.

Once it receives a catalog, Puppet agent applies it to the node by checking each resource the catalog describes. If it finds any resources that are not in their desired state, it makes the changes necessary to correct them. Or, in no-op mode, it reports on what changes would have been done.

After applying the catalog, the agent sends a report to the Puppet master.

For more information, see:

    Puppet Agent on *nix Systems
    Puppet Agent on Windows Systems
    Puppet Server

### Communications and security

Puppet agent nodes and Puppet masters communicate by HTTPS with client verification.

The Puppet master provides an HTTP interface, with various endpoints available. When requesting or submitting anything to the master, the agent makes an HTTPS request to one of those endpoints.

Client-verified HTTPS means each master or agent must have an identifying SSL certificate. They each examine their counterpart s certificate to decide whether to allow an exchange of information.

Puppet includes a built-in certificate authority for managing certificates. Agents can automatically request certificates through the master s HTTP API. You can use the puppet cert command to inspect requests and sign new certificates. And agents can then download the signed certificates.

For more information, see:

    A walkthrough of Puppet s HTTPS communications
    The Puppet master s HTTP API
    The Puppet master s auth.conf file
    Background reference on SSL and HTTPS.

### The stand-alone architecture

Alternatively, Puppet can run in a stand-alone architecture, where each managed node has its own complete copy of your configuration info and compiles its own catalog.

In this architecture, managed nodes run the Puppet apply application, usually as a scheduled task or cron job. You can also run it on demand for initial configuration of a server or for smaller configuration tasks.

Like the Puppet master application, Puppet apply needs access to several sources of configuration data, which it uses to compile a catalog for the node it is managing.

After Puppet apply compiles the catalog, it immediately applies it by checking each resource the catalog describes. If it finds any resources that are not in their desired state, it makes the changes necessary to correct them. Or, in no-op mode, it reports on what changes would have been needed.

After applying the catalog, Puppet apply stores a report on disk. You can configure it to send reports to a central service.

For more information, see the documentation for the Puppet apply application.

### Differences between agent/master and stand-alone

In general, Puppet apply can do the same things as the combination of Puppet agent and Puppet master, but there are several trade-offs around security and the ease of certain tasks.

If you don t have a preference, you should select the agent/master architecture. If you have questions, considering these trade-offs helps you make your decision.

*    Principle of least privilege. In agent/master Puppet, each agent only gets its own configuration, and is unable to see how other nodes are configured. With Puppet apply, it s impractical to do this, so every node has access to complete knowledge about how your site is configured. Depending on how you re configuring your systems, this can potentially raise the risks of horizontal privilege escalation.
*    Ease of centralized reporting and inventory. Agents send reports to the Puppet master by default, and the master can be configured with any number of report handlers to pass these on to other services. You can also connect the master to PuppetDB, a powerful tool for querying inventory and activity data. Puppet apply nodes handle their own information, so if you re using PuppetDB or sending reports to another service, each node needs to be configured and authorized to connect to it.
*    Ease of updating configurations. Only Puppet master servers have the Puppet modules, main manifests, and other data necessary for compiling catalogs. This means that when you need to update your systems  configurations, you only need to update content on one (or a few) master servers. In a decentralized puppet apply deployment, you ll need to sync new configuration code and data to every node.
*    CPU and memory usage on managed machines. Since Puppet agent doesn t compile its own catalogs, it uses fewer resources on the machines it manages, leaving them with more capacity for their designated tasks.
*    Need for a dedicated master server. The Puppet master takes on the performance load of compiling all catalogs, and it should usually be a dedicated machine with a fast processor, lots of RAM, and a fast disk. Not everybody wants to (or is able to) allocate that, and Puppet apply can get around the need for it.
*    Need for good network connectivity. Agents need to be able to reach the Puppet master at a reliable hostname in order to configure themselves. If a system lives in a degraded or isolated network environment, you might want it to be more self-sufficient.
*    Security overhead. Agents and masters use HTTPS to secure their communications and authenticate each other, and every system involved needs an SSL certificate. Puppet includes a built-in CA to easily manage certificates, but it s even easier to not manage them at all. (Of course, you ll still need to manage security somehow, since you re probably using Rsync or something to update Puppet content on every node.)

## HIERA 


Puppet’s strength is in reusable code. Code that serves many needs must be configurable: put site-specific information in external configuration data files, rather than in the code itself.

Puppet uses Hiera to do two things:

*    Store the configuration data in key-value pairs
*    Look up what data a particular module needs for a given node during catalog compilation

This is done via:

*    Automatic Parameter Lookup for classes included in the catalog
*    Explicit lookup calls 

Hiera’s hierarchical lookups follow a **defaults, with overrides** pattern, meaning you specify common data once, and override it in situations where the default won’t work. Hiera uses Puppet’s facts to specify data sources, so you can structure your overrides to suit your infrastructure. While using facts for this purpose is common, data-sources may well be defined without the use of facts.

Puppet 5 comes with support for JSON, YAML, and EYAML files.

Related topics: Automatic Parameter Lookup.

### Hiera hierarchies

Hiera looks up data by following a hierarchy - an ordered list of data sources.

Hierarchies are configured in a hiera.yaml configuration file. Each level of the hierarchy tells Hiera how to access some kind of data source.
Hierarchies interpolate variables

Most levels of a hierarchy interpolate variables into their configuration:

	path: "os/%{facts.os.family}.yaml".

The percent-and-braces %{variable} syntax is a Hiera interpolation token. It is similar to the Puppet language’s ${expression} interpolation tokens. Wherever you use an interpolation token, Hiera determines the variable’s value and inserts it into the hierarchy.

The facts.os.family uses the Hiera special key.subkey notation for accessing elements of hashes and arrays. It is equivalent to $facts['os']['family'] in the Puppet language but the 'dot' notation will produce an empty string instead of raising an error if parts of the data is missing. Make sure that an empty interpolation does not end up matching an unintended path.

You can only interpolate values into certain parts of the config file. For more info, see the hiera.yaml format reference.

With node-specific variables, each node gets a customized set of paths to data. The hierarchy is always the same.
Hiera searches the hierarchy in order

Once Hiera replaces the variables to make a list of concrete data sources, it checks those data sources in the order they were written.

Generally, if a data source doesn’t exist, or doesn’t specify a value for the current key, Hiera skips it and moves on to the next source, until it finds one that exists - then it uses it. Note that this is the default merge strategy, but does not always apply, for example, Hiera can use data from all data sources and merge the result.

Earlier data sources have priority over later ones. In the example above, the node-specific data has the highest priority, and can override data from any other level. Business group data is separated into local and global sources, with the local one overriding the global one. Common data used by all nodes always goes last.

That’s how Hiera’s “defaults, with overrides” approach to data works - you specify common data at lower levels of the hierarchy, and override it at higher levels for groups of nodes with special needs.

### Layered hierarchies

Hiera uses layers of data with a hiera.yaml for each layer.

Each layer can configure its own independent hierarchy. Before a lookup, Hiera combines them into a single super-hierarchy: global → environment → module.
Note: There is a fourth layer - default_hierarchy - that can be used in a module’s hiera.yaml. It only comes into effect when there is no data for a key in any of the other regular hierarchies
Assume the example above is an environment hierarchy (in the production environment). If we also had the following global hierarchy:

	version: 5
	hierarchy:
	- name: "Data exported from our old self-service config tool"
		path: "selfserve/%{trusted.certname}.json"
		data_hash: json_data
		datadir: data

And the NTP module had the following hierarchy for default data:

	version: 5
	hierarchy:
		- name: "OS values"
		  path: "os/%{facts.os.name}.yaml"
		- name: "Common values"
		  path: "common.yaml"
	defaults:
		data_hash: yaml_data
	datadir: data

Then in a lookup for the ntp::servers key, thrush.example.com would use the following combined hierarchy:

	\<CODEDIR\>/data/selfserve/thrush.example.com.json
    \<CODEDIR\>/environments/production/data/nodes/thrush.example.com.yaml
    \<CODEDIR\>/environments/production/data/location/belfast/ops.yaml
    \<CODEDIR\>/environments/production/data/groups/ops.yaml
    \<CODEDIR\>/environments/production/data/os/Debian.yaml
    \<CODEDIR\>/environments/production/data/common.yaml
    \<CODEDIR\>/environments/production/modules/ntp/data/os/Ubuntu.yaml
    \<CODEDIR\>/environments/production/modules/ntp/data/common.yaml

The combined hierarchy works the same way as a layer hierarchy. Hiera skips empty data sources, and either returns the first found value or merges all found values.
Note: By default, datadir refers to the directory named ‘data’ next to the hiera.yaml.

### Tips for making a good hierarchy

* Make a short hierarchy. Data files will be easier to work with.
*    Use the roles and profiles method to manage less data in Hiera. Sorting hundreds of class parameters is easier than sorting thousands.
*    If the built-in facts don’t provide an easy way to represent differences in your infrastructure, make custom facts. For example, create a custom datacenter fact that is based on information particular to your network layout so that each datacenter is uniquely identifiable.
*    Give each environment – production, test, development – its own hierarchy.

Related topics: codedir, confdir.

### Hiera configuration layers

Hiera uses three independent layers of configuration. Each layer has its own hierarchy, and they’re linked into one super-hierarchy before doing a lookup.

The three layers are searched in the following order: global → environment → module. Hiera searches every data source in the global layer’s hierarchy before checking any source in the environment layer.

#### The global layer

The configuration file for the global layer is located, by default, in $confdir/hiera.yaml. You can change the location by changing the hiera_config setting in puppet.conf.

Hiera has one global hierarchy. Since it goes before the environment layer, it’s useful for temporary overrides, for example, when your ops team needs to bypass its normal change processes.

The global layer is the only place where legacy Hiera 3 backends can be used - it’s an important piece of the transition period when you migrate you backends to support Hiera 5. It supports the following config formats: hiera.yaml v5, hiera.yaml v3 (deprecated).

Other than the above use cases, try to avoid the global layer. All normal data should be specified in the environment layer.

#### The environment layer

The configuration file for the global layer is located, by default, \<ENVIRONMENT DIR\>/hiera.yaml.

The environment layer is where most of your Hiera data hierarchy definition happens. Every Puppet environment has its own hierarchy configuration, which applies to nodes in that environment. Supported config formats include: v5,v5, v3 (deprecated).

#### The module layer

The configuration file for a module layer is located, by default, in a module's \<MODULE\>/hiera.yaml.

The module layer sets default values and merge behavior for a module’s class parameters. It is a convenient alternative to the params.pp pattern.
Note: To get the exact same behaviour as params.pp, the default_hierarchy should be used, as those bindings are excluded from merges. When placed in the regular hierarchy in the module’s hierarchy the bindings will be merged when a merge lookup is performed.

It comes last in Hiera’s lookup order, so environment data set by a user overrides the default data set by the module’s author.
Every module can have its own hierarchy configuration. You can only bind data for keys in the module’s namespace. For example:
Lookup key	Relevant module hierarchy
	ntp::servers 	ntp
	jenkins::port	jenkins
	secure_server 	(none)

Hiera uses the ntp module’s hierarchy when looking up ntp::servers, but uses the jenkins module’s hierarchy when looking up jenkins::port. Hiera never checks the module for a key beginning with jenkins::.

When you use the lookup function for keys that don’t have a namespace (for example, secure_server), the module layer is not consulted.

The three-layer system means that each environment has its own hierarchy, and so do modules. You can make hierarchy changes on an environment-by-environment basis. Module data is also customizable.
