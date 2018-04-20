# Best practices and common trap  

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


## best practices

### Use Modules when possible:

Puppet modules are something everyone should use. If you have an application you are managing, add a module for it, so that you can keep the manifests, plugins (if any), source files, and templates altogether.

### Keep your Puppet content in Version Control:
 
There is NO reason, not to use a version control system while developing puppet manifest/modules. You can pick your favorite systems — popular choices being Git, Mercurial or Bazaar which prove to be particularly useful due to the ease that they provide in managing multiple branches of code.
Version control is well-supported by the Puppet ecosystem. It is possible to use a mature Software Development Lifecycle to manage the development and maintenance of your Puppet manifests, tightly integrated with a branch-based workflow that truly realizes the ideals of **infrastructure-as-code.**
Using version control helps in opening up of lot of additional possibilities with puppet, like better tracking of changes, testing your Puppet manifests in an isolated environment, promoting your configuration from environment to environment, etc. Version control even provides a free backup for your configuration code.
By making a use of a collaboration tool like Github or Bitbucket, you and your team can easily revise each of the changes separately before they are applied. This results in much better, more sustainable Puppet code. Placing all your Puppet files under version control is counted as one of the best Puppet practice and whenever you are ready to **deploy** any kind of changes to your Puppetmaster, you just have to sync the working copy on the server with the code in the version control repository.
 
###  Make use of Environments:

Environments are isolated groups of Puppet agent s nodes. A Puppet master server can serve each environment with completely different main manifests and module paths. Puppet has this concept of Environment that helps in applying your configuration changes on less critical servers first and after the changes has been tested and ready, then promoting those changes to production. This frees you to use different versions of the same modules for different populations of nodes, which is useful for testing changes to your Puppet code before implementing them on production machines.
Staging and production are two Puppet environments that are used where Staging environment is used at an initial provisioning of a server to all pre-production boxes and production servers make use of the Production environment. Both environments are tied to a specific branch in Git repository i.e., **master** branch in Git is production and **staging** branch is staging.
 
### Use dry-runs:
 
Even with the best precautions taken, sometimes your Puppet manifest doesn t do exactly what you expected. Things can get messy at times whenever you actually get to run the Puppet agent to apply your configuration updates on your servers. For example, if it would update a config file and restart a production service this could result in unplanned downtime. Also, sometimes manual configuration changes are made on a server which Puppet would overwrite.
To reduce the risk of problems, you can use Puppet s agent in **dry run** mode (also called noop mode, for no operation) using the following options:

	puppet agent […] –verbose –noop –test

By making use of this practice you will be able to see the difference for all files that would modify as well as validate things according to your expectation. It will help Puppet agent to only show what it would do, not what it did.
 
###  Managing Puppet modules with librarian-puppet:
 
Handling module dependencies can sometimes become a source of worries, especially when several numbers of people are working on Puppet code and each one demand to test it on their own computer. Librarian-puppet is a bundler for your puppet infrastructure that helps in providing sanity to the process by automatically managing your module dependencies.
Librarian-puppet manages your modules/ directory based on your **Puppetfile**. The tool will install, update or remove modules automatically when you run it, always matching what s specified in the Puppetfile. Your Puppetfile becomes the authoritative source for what modules you require and at what version, tag or branch.
Once using Librarian-puppet you should not modify the contents of your modules directory. The individual modules  repos should be updated, tagged with a new release and the version bumped in your Puppetfile. It will help resolving and installing modules  dependencies and become aware of compatibility issues.
Librarian-puppet simplify deployment of your Puppet infrastructure by automatically pulling in modules from the forge and git repositories with a single command thus saving you from installation and managing your modules manually. Deployment normally comes up with following two simple steps:
     
    Sync your main sources with your code repository (ex: git pull)
    Run librarian-puppet to synchronize your installed Puppet modules

Don t use Git dependencies with any version specifier
 
### Keep sensitive data safe:
 
Today Data security is the key concern for any organization. Some data always needs to be kept safe. For example, for maintaining the data security it may be required to put in your Puppet code in passwords, private keys, SSL certificates and so on.
Don t put Puppet code in version control unless you re absolutely aware of the risks you re taking while doing so.
If you re already a bit familiar with Hiera which is a Puppet tool, you know why it s a good idea to separate your data from your Puppet manifests. In case you aren t familiar, though, using Hiera leases you write and use reusable manifests and modules. After you store your organization-specific data in Hiera, Puppet classes can request the data they need from your Hiera data store.
Hiera allows you to store data about your servers and infrastructure in YAML or JSON files. From usage, you ll see that most data in Hiera files is not confidential in nature… so should we refrain from using version control for Hiera files just because of a few elements that are unsafe? CERTAINLY NOT!!!!!!!!!!!!
The trick is to use Hiera s ability to combine multiple data sources or backend
 
or

What you can do is split Hieradata files into 2 types: YAML files for your **main** Hieradata files and JSON files to store your **secured** data. Those JSON files are not to be put under version control and are stored securely on a single location i.e., **the Puppetmaster**. This way, very few people can actually see the contents of the sensitive files.
 
### Producing abstractions for you high-level classes:
 
Wrapping up all the uses of modules into a covering or let s say into a wrapper classes provides higher-up maintainability of the Puppet code in time. For example, Assume you want to setup a reverse proxy server using an existing Nginx module. Instead of directly assigning the ‘nginx  class on your nodes and setting all of the required stuff up, create instead a new class called, say, ‘proxy_server  with the attributes you want to consider for your proxy server as class parameters. Assigning the ‘proxy_server  class on your node not only better states your intent, but it also creates a nice little abstraction over what you consider as a **proxy server**. Later on, if you decide to go away from Nginx (highly improbable) or use another Nginx module (more probable!), then you ll probably just need to change the content of your **proxy_server** class, instead of a bunch of tangled node definitions.

In the End:

Puppet isn t the only Configuration management (CM) tool available, but it is the most mature, with a large community of active module developers. Implementing these practices will help you in getting the most out of the Puppet.


## best practices from puppet lab

### Part 1: Module structure. 

	Gary talks about what you need to make a good component module, and adds some **gotchas** he s noticed over time. For example, he recommends treating parameters as, essentially, the API to your module. Gary also recommends keeping component modules generic, and not adding Hiera calls inside them. There s a lot more in this post, including strong advice to store all your modules in version control.

#### It all starts with the component module

The first piece of a functional Puppet deployment starts with what we call ‘component modules . Component modules are the lowest level in your deployment, and are modules that configure specific pieces of technology (like apache, ntp, mysql, and etc…). Component modules are well-encapsulated, have a reasonable API, and focus on doing small, specific things really well (i.e. the *nix way).

I don t want to write thousands of words on building component modules because I feel like others have done this better than I. As examples, check out RI s Post on a simple module structure, Puppet Labs  very own docs on the subject, and even Alessandro s Puppetconf 2012 session. Instead, I d like to provide some pointers on what I feel makes a good component module, and some ‘gotchas  we ve noticed.


#### Parameters are your API

In the current world of Puppet, you MUST define the parameters your module will accept in the Puppet DSL. Also, every parameter MUST ultimately have a value when Puppet compiles the catalog (whether by explicitly passing this parameter value when declaring the class, or by assuming a default value). Yes, it s funny that, when writing a Puppet class, if you typo a VARIABLE Puppet will not alert you to this (in a NON use strict-ian sort of approach) and will happily accept a variable in an undefined state, but the second you don t pass a value to your class parameter you re in for a rude compilation error. This is the way of Puppet classes at the time of this writing, so you re going to see Puppet classes with LINES of defined parameters. I expect this to change in the future (please let this change in the near future), but for now, it s a necessary evil.

The parameters you expose to your top-level class (i.e. given class names like apache and apache::install, I m talking specifically about apache) should be treated as an API to your module. IDEALLY, they re the ONLY THING that a user needs to modify when using your module. Also, whenever possible, it should be the case that a user need ONLY interact with the top-level class when using your module (of course, defined resource types like apache::vhost are used on an ad-hoc basis, and thus are the exception here).

#### Inherit the ::params class

We re starting to make enemies at this point. It s been a convention for modules to use a ::params class to assign values to all variables that are going to be used for all classes inside the module. The idea is that the ::params class is the one-stop-shop to see where a variable is set. Also, to get access to a variable that s set in a Puppet class, you have to declare the class (i.e. use the include() function or inherit from that class). When you declare a class that has both variables AND resources, those resources get put into the catalog, which means that Puppet ENFORCES THE STATE of those resources. What if you only needed a variable s value and didn t want to enforce the rest of the resources in that class? There s no good way in Puppet to do that. Finally, when you inherit from a class in Puppet that has assigned variable values, you ALSO get access to those variables in the parameter definition section of your class (i.e. the following section of the class:

	class apache (
		$port = $apache::params::port,
		$user = $apache::params::user,
	) inherits apache::params {

See how I set the default value of $apache::port to $apache::params::port? I could only access the value of the variable $apache::params::port in that section by inheriting from the apache::params class. I couldn t insert include apache::params below that section and be allowed access to the variable up in the parameter defaults section (due to the way that Puppet parses classes).

FOR THIS REASON, THIS IS THE ONLY RECOMMENDED USAGE OF INHERITANCE IN PUPPET!

We do NOT recommend using inheritance anywhere else in Puppet and for any other reason because there are better ways to achieve what you want to do INSTEAD of using inheritance. Inheritance is a holdover from a scarier, more lawless time.

NOTE: Data in Modules – There s a ‘Data in Modules  pattern out there that attempts to eliminate the ::params class. I wrote about it in a previous post, and I recommend you read that post for more info (it s near the bottom).
Do NOT do Hiera lookups in your component modules!

This is something that s really only RECENTLY been pushed. When Hiera was released, we quickly recognized that it would be the answer to quite a few problems in Puppet. In the rush to adopt Hiera, many people started adding Hiera calls to their modules, and suddenly you had ‘Hiera-compatible  modules out there. This caused all kinds of compatibility problems, and it was largely because there wasn t a better module structure and workflow by which to integrate Hiera. The pattern that I ll be pushing DOES INDEED use Hiera, BUT it confines all Hiera calls to a higher-level wrapper class we call a ‘profile . The reasons for NOT using Hiera in your module are:

*    By doing Hiera calls at a higher level, you have a greater visibility on exactly what parameters were set by Hiera and which were set explicitly or by default values.
*    By doing Hiera calls elsewhere, your module is backwards-compatible for those folks who are NOT using Hiera

Remember – your module should just accept a value and use it somewhere. Don t get TOO smart with your component module – leave the logic for other places.
Keep your component modules generic

We always get asked **How do I know if I m writing a good module?** We USED to say **Well, does it work?** (and trust me, that was a BIG hurdle). Now, with data separation models out there like Hiera, I have a couple of other questions that I ask (you know, BEYOND asking if it compiles and actually installs the thing it s supposed to install). The best way I ve found to determine if your module is ‘generic enough  is if I asked you TODAY to give me your module, would you give it to me, or would you be worried that there was some company-specific data locked in there? If you have company-specific data in your module, then you need to refactor the module, store the data in Hiera, and make your module more generic/reusable. Also, does your module focus on installing one piece of technology, or are you declaring packages for shared libraries or other components (like gcc, apache, or other common components)? You re not going to win any prizes for having the biggest, most monolithic module out there. Rather, if your module is that large and that complex, you re going to have a hell of a time debugging it. Err on the side of making your modules smaller and more task-specific. So what if you end up needing to declare 4 classes where you previously declared 1? In the roles and profiles pattern we will show you in the next blog post, you can abstract that away ANYHOW.
Don t play the **what if** game

I ve had more than a couple of gigs where the customer says something along the lines of **What if we need to introduce FreeBSD/Solaris/etc… nodes into our organization, shouldn t I account for them now?** This leads more than a few people down a path of entirely too-complex modules that become bulky and unwieldy. Yes, your modules should be formatted so that you can simply add another case in your ::params class for another OS s parameters, and yes, your module should be formatted so that your ::install or ::config class can handle another OS, but if you currently only manage Redhat, and you ve only EVER managed Redhat, then don t start adding Debian parameters RIGHT NOW just because you re afraid you might inherit Ubuntu machines. The goal of Puppet is to automate the tasks that eat up the MAJORITY of your time so you can focus on the edge cases that really demand your time. If you can eventually automate those edge cases, then AWESOME! Until then, don t spend the majority of your time trying to automate the edge cases only to drown under the weight of deadlines from simple work that you COULD have already automated (but didn t, because you were so worried about the exceptions)!
Store your modules in version control

This should go without saying, but your modules should be stored in version control (a la git, svn, hg, whatever). We tend to prefer git due to its lightweight branching and merging (most of our tooling and solutions will use git because we re big git users), but you re free to use whatever you want. The bigger question is HOW to store your modules in version control. There are usually two schools of thought:

*    One repository per module
*    All modules in a single repository

Each model has its pros and cons, but we tend to recommend one module per repository for the following reasons:

*    Individual repos mean individual module development histories
*    Most VCS solutions don t have per-folder ACLs for a single repositories; having multiple repos allows per-module security settings.
*    With the one-repository-per-module solution, modules you pull down from the Forge (or Github) must be committed to your repo. Having multiple repositories for each module allow you to keep everything separate

NOTE: This becomes important in the third blog post in the series when we talk about moving changes to each Puppet Environment, but it s important to introduce it NOW as a ‘best practice . If you use our recommended module/environment solution, then one-module-per-repo is the best practice. If you DON T use our solution, then the single repository per for all modules will STILL work, but you ll have to manage the above issues. Also note that even if you currently have every module in a single repository, you can STILL use our solution in part 3 of the series (you ll just need to perform a couple of steps to conform).

#### Best practices are shit

In general, ‘best practices  are only recommended if they fit into your organizational workflow. The best and worst part of Puppet is that it s infinitely customizable, so ‘best practices  will invariably be left wanting for a certain subset of the community. As always, take what I say under consideration; it s quite possible that I could be entirely full of shit.

	
	
	
### Part 2: Roles and profiles. 

	You should be able to classify a node with just a single role, Gary says, and include profiles in roles. There s a lot of specific advice in this post on using roles & profiles to classify servers, including how to distinguish two machines that are very similar, but not the same.

People usually stop once they have a library of component modules (whether hand-written, taken from Github, or pulled from The Forge). The idea is that you can classify all of your nodes in site.pp, the Puppet Enterprise Console, The Foreman, or with some other ENC, so why not just declare all your classes for every node when you need them?

Because that s a lot of extra work and opportunities for fuckups.

People recognized this, so in the EARLY days of Puppet they would create node blocks in site.pp and use inheritance to inherit from those blocks. This was the right IDEA, but probably not the best PLACE for it. Eventually, ‘Profiles  were born.

The idea of ‘Roles and Profiles  originally came from a piece that Craig Dunn wrote while he worked for the BBC, and then Adrien Thebo also wrote a piece that documents the same sort of pattern. So why am I writing about it a THIRD time? Well, because I feel it s only a PIECE of an overall puzzle. The introduction of Hiera and other awesome tools (like R10k, which we will get to on the next post) still make Roles and Profiles VIABLE, but they also extend upon them.

One final note before we move on – the terms ‘Roles  and ‘Profiles  are ENTIRELY ARBITRARY. They re not magic reserve words in Puppet, and you can call them whatever the hell you want. It s also been pointed out that Craig MIGHT have misnamed them (a ROLE should be a model for an individual piece of tech, and a PROFILE should probably be a group of roles), but, like all good Puppet Labs employees – we suck at naming things.
Profiles: technology-specific wrapper classes

A profile is simply a wrapper class that groups Hiera lookups and class declarations into one functional unit. For example, if you wanted Wordpress installed on a machine, you d probably need to declare the apache class to get Apache setup, declare an apache::vhost for the Wordpress directory, setup a MySQL database with the appropriate classes, and so on. There are a lot of components that go together when you setup a piece of technology, it s not just a single class.

Because of this, a profile exists to give you a single class you can include that will setup all the necessary bits for that piece of technology (be it Wordpress, or Tomcat, or whatever).

Let s look at a simple profile for Wordpress:
profiles/manifests/wordpress.pp

	class profiles::wordpress {
	
		## Hiera lookups
		$site_name               = hiera( profiles::wordpress::site_name )
		$wordpress_user_password = hiera( profiles::wordpress::wordpress_user_password )
		$mysql_root_password     = hiera( profiles::wordpress::mysql_root_password )
		$wordpress_db_host       = hiera( profiles::wordpress::wordpress_db_host )
		$wordpress_db_name       = hiera( profiles::wordpress::wordpress_db_name )
		$wordpress_db_password   = hiera( profiles::wordpress::wordpress_db_password )
		$wordpress_user          = hiera( profiles::wordpress::wordpress_user )
		$wordpress_group         = hiera( profiles::wordpress::wordpress_group )
		$wordpress_docroot       = hiera( profiles::wordpress::wordpress_docroot )
		$wordpress_port          = hiera( profiles::wordpress::wordpress_port )

		## Create user
		group {  wordpress :
			ensure => present,
			name   => $wordpress_group,
		}
		user {  wordpress :
			ensure   => present,
			gid      => $wordpress_group,
			password => $wordpress_user_password,
			name     => $wordpress_group,
			home     => $wordpress_docroot,
		}

	## Configure mysql
	class {  mysql::server :
		root_password => $wordpress_root_password,
	}

	class {  mysql::bindings :
		php_enable => true,
	}

	## Configure apache
	include apache
	include apache::mod::php
	apache::vhost { $::fqdn:
		port    => $wordpress_port,
		docroot => $wordpress_docroot,
	}

	## Configure wordpress
	class {  ::wordpress :
		install_dir => $wordpress_docroot,
		db_name     => $wordpress_db_name,
		db_host     => $wordpress_db_host,
		db_password => $wordpress_db_password,
	}	
	}	

#### Name your profiles according to the technology they setup

Profiles are technology-specific, so you ll have one to setup wordpress, and tomcat, and jenkins, and…well, you get the picture. You can also namespace your profiles so that you have profiles::ssh::server and profiles::ssh::client if you want. You can even have profiles::jenkins::tomcat and profiles::jenkins::jboss or however you need to namespace according to the TECHNOLOGIES you use. You don t need to include your environment in the profile name (a la profiles::dev::tomcat) as the bits of data that make the dev environment different from production should come from HIERA, and thus aren t going to be different on a per-profile basis. You CAN setup profiles according to your business unit if multiple units use Puppet and have different setups (a la security::profiles::tomcat versus ops::profiles::tomcat), but the GOAL of Puppet is to have one main set of modules that every group uses (and the Hiera data being different for every group). That s the GOAL, but I m pragmatic enough to understand that not everywhere is a shiny, happy ‘DevOps Garden. 

#### Do all Hiera lookups in the profile

You ll see that I declared variables and set their values with Hiera lookups. The profile is the place for these lookups because the profile collects all external data and declares all the classes you ll need. In reality, you ll USUALLY only see profiles looking up parameters and declaring classes (i.e. declaring users and groups like I did above will USUALLY be left to component classes).

I do the Hiera lookups first to make it easy to debug from where those values came. I don t rely on ‘Automatic Parameter Lookup  in Puppet 3.x.x because it can be ‘magic  for people who aren t aware of it (for people new to Puppet, it s much easier to see a function call and trace back what it does rather than experience Puppet doing something unseen and wondering what the hell happened).

Finally, you ll notice that my Hiera lookups have NO DEFAULT VALUES – this is BY DESIGN! For most people, their Hiera data is PROBABLY located in a separate repository as their Puppet module data. Imagine making a change to your profile to have it lookup a bit of data from Hiera, and then imagine you FORGOT to put that data into Hiera. What happens if you provide a default value to Hiera? The catalog compiles, that default value gets passed down to the component module, and gets enforced on disk. If you have good tests, you MIGHT see that the component you configured has a bit of data that s not correct, but what if you don t have a great post-Puppet testing workflow? Puppet will correctly set this default value, according to Puppet everything is green and worked just fine, but now your component is setup incorrectly. That s one of the WORST failures – the ones that you don t catch. Now, imagine you DON T provide a default value. In THIS case, Puppet will raise a compilation error because a Hiera lookup didn t return a value. You ll catch your error before anything gets pushed to Production and you can catch the screwup. This is a MUCH better solution.

#### Use parameterized class declarations and explicitly pass values you care about

The parameterized class declaration syntax can be dangerous. The difference between the include function and the parameterized class syntax is that the include function is idempotent. You can do the following in a Puppet manifest, and Puppet doesn t raise an error:

	include apache
	include apache
	include apache

This is because the include function checks to see if the class is in the catalog. If it ISN T, then it adds it. If it IS, then it exits cleanly. The include function is your pal.

Consider THIS manifest:

	class {  apache : }
	include apache
	include apache

Does this work? Yep. The parameterized class syntax adds the class to the catalog, the include function detects this and exits cleanly twice. What about THIS manifest:

	include apache
	class {  apache : }
	include apache

Does THIS work? Nope! Puppet raises a compilation error because a class was declared more than once in a catalog. Why? Well, consider that Puppet is ‘declarative …all the way up until it isn t. Puppet s PARSER reads from the top of the file to the bottom of the file, and we have a single-pass parser when it comes to things like setting variables and declaring classes. When the parser hits the first include function, it adds the class to the catalog. The parameterized class syntax, however, is a honey badger: it doesn t give a shit. It adds a class to the catalog regardless of whether it already exists or not. So why would we EVER use the parameterized class declaration syntax? We need to use it because the include function doesn t allow you to pass parameters when you declare a class.

So wait – why did I spend all this time explaining why the parameterized class syntax is more dangerous than the include function ONLY to recommend its use in profiles? For two reasons:

*    We need to use it to pass parameters to classes
*    We re wrapping its use in a class that we can IN TURN declare with the include function

Yes, we can get the best of BOTH worlds, the ability to pass parameters and the use of our pal the include function, with this wrapper class. We ll see the latter usage when we come to roles, but for now let s focus on passing parameter values.

In the first section, we set variables with Hiera lookups, now we can pass those variables to classes we re declaring with the parameterized class syntax. This allows the declaration of the class to be static, but the parameters we pass to that class to change according to the Hiera hierarchy. We ve explicitly called the hiera function, so it makes it easier to debug, and we re explicitly passing parameter values so we know definitively which parameters are being passed (and thus are overriding default values) to the component module. Finally, since our component modules do NOT use Hiera at all, we can be sure that if we re not passing a parameter that it s getting its value from the default set in the module s ::params class.

Everything we do here is meant to make things easier to debug when it s 3am and things aren t working. Any asshole can do crazy shit in Puppet, but a seasoned sysadmin writes their code for ease of debugging during 3am pages.

#### An annoying Puppet bug – top-level class declarations and profiles

Oh, ticket 2053, how terrible are you? This is one of those bug numbers that I can remember by heart (like 8040 and 86). Puppet has the ability to do ‘relative namespacing , which allows you to declare a variable called $port in a class called $apache and refer to it as $port instead of fully-namespacing the variable, and thus having to call it $apache::port inside the apache class. It s a shortcut – you can STILL refer to the variable as $apache::port in the class – but it comes in handy. The PROBLEM occurs when you create a profile, as we did above, called profiles::wordpress and you try to declare a class called wordpress. If you do the following inside the profiles::wordpress class, what class is being declared:

	include wordpress

If you think you re declaring a wordpress class from within a wordpress module in your Puppet modulepath, you would be wrong. Puppet ACTUALLY thinks you re trying to declare profiles::wordpress because you re INSIDE the profiles::wordpress class and it s doing relative namespacing (i.e. in the same way you refer to $port and ACTUALLY mean $apache::port it thinks you re referring to wordpress and ACTUALLY mean profiles::wordpress.

Needless to say, this causes LOTS of confusion.

The solution here is to declare a class called ::wordpress which tells Puppet to go to the top-level namespace and look for a module called wordpress which has a top-level class called wordpress. It s the same reason that we refer to Facter Fact values as $::osfamily instead of $osfamily in class definitions (because you can declare a local variable called $osfamily in your class). This is why in the profile above you see this:

	class {  ::wordpress :
		install_dir => $wordpress_docroot,
		db_name     => $wordpress_db_name,
		db_host     => $wordpress_db_host,
		db_password => $wordpress_db_password,
	}

When you use profiles and roles, you ll need to do this namespacing trick when declaring classes because you re frequently going to have a profile::**sometech** that will declare the **sometech** top-level class.

#### Roles: business-specific wrapper classes

How do you refer to your machines? When I ask you about that cluster over there, do you say **Oh, you mean the machines with java 1.6, apache, mysql, etc…**? I didn t think so. You usually have names for them, like the **internal compute cluster** or **app builder nodes** or **DMZ repo machines** or whatever. These names are your Roles. Roles are just the mapping of your machine s names to the technology that should be ON them. In the past we had descriptive hostnames that afforded us a code for what the machine ‘did  – roles are just that mapping for Puppet.

Roles are namespaced just like profiles, but now it s up to your organization to fill in the blanks. Some people immediately want to put environments into the roles (a la roles::uat::compute_cluster), but that s usually not necessary (as MOST LIKELY the compute cluster nodes have the SAME technology on them when they re in dev versus when they re in prod, it s just the DATA – like database names, VIP locations, usernames/passwords, etc – that s different. Again, these data differences will come from Hiera, so there should be no reason to put the environment name in your role). You still CAN put the environment name in the role if it makes you feel better, but it ll probably be useless.

#### Roles ONLY include profiles

So what exactly is in the role wrapper class? That depends on what technology is on the node that defines that role. What I can tell you for CERTAIN is that roles should ONLY use the include function and should ONLY include profiles. What does this give us? This gives us our pal the include function back! You can include the same profile 100 times if you want, and Puppet only puts it in the catalog once.
Every node is classified with one role. Period.

The beautiful thing about roles and profiles is that the GOAL is that you should be able to classify a node with a SINGLE role and THAT S IT. This makes classification simple and static – the node gets its role, the role includes profiles, profiles call out to Hiera for data, that data is passed to component modules, and away we go. Also, since classification is static, you can use version control to see what changes were introduced to the role (i.e. what profiles were added or removed). In my opinion, if you need to apply more than one role to a node, you ve introduced a new role (see below).

#### Roles CAN use inheritance…if you like

I ve seen people implement roles a couple of different ways, and one of them is to use inheritance to build a catalog. For example, you can define a base roles class that includes something like a base security profile (i.e. something that EVERY node in your infrastructure should have). Moving down the line, you COULD namespace according to function like roles::app for your application server machines. The roles::app class could inherit from the roles class (which gets the base security profile), and could then include the profiles necessary to setup an application server. Next, you could subclass down to roles::app::site_foo for an application server that supports some site in your organization. That class inherits from the roles::app class, and then adds profiles that are specific to that site (maybe they use Jboss instead of Tomcat, and thus that s where the differentiation occurs). This is great because you don t have a lot of repeated use of the include function, but it also makes it hard to definitively look at a specific role to see exactly what s being declared (i.e. all the profiles). You have to weigh what you value more: less typing or greater visibility. I will err on the side of greater visibility (just due to that whole 3am outage thing), but it s up to you to decide what to optimize for.
A role similar, yet different, from another role is: a new role

EVERYBODY says to me **Gary, I have this machine that s an AWFUL LOT like this role over here, but…it s different.** My answer to them is: **Great, that s another role.** If the thing that s different is data (i.e. which database to connect to, or what IP address to route traffic through), then that difference should be put in HIERA and the classification should remain the same. If that difference is technology-specific (i.e. this server uses JBoss instead of Tomcat) then first look and see if you can isolate how you know this machine is different (maybe it s on a different subnet, maybe it s at a different location, something like that). If you can figure that out and write a Fact for it (or use similar conditional logic to determine this logically), then you can just drop that conditional logic in your role and let it do the heavy lifting. If, in the end, this bit of data is totally arbitrary, then you ll need to create another role (perhaps a subclass using the above namespacing) and assign it to your node.

The hardest thing about this setup is naming your roles. Why? Every site is different. It s hard for me to account for differences in your setup because your workplace is dysfunctional (seriously).
Review: what does this get you?

Let s walk through every level of this setup from the top to the bottom and see what it gets you. Every node is classified to a single role, and, for the most part, that classification isn t going to change. Now you can take all the extra work off your classifier tool and put it back into the manifests (that are subject to version control, so you can git blame to your heart s content and see who last changed the role/profile). Each role is going to include one or more profile, which gives us the added idempotent protection of the include function (of course, if profiles have collisions with classes you ll have to resolve those. Say one or more profiles tries to include an apache class – simply break that component out into a separate profile, extract the parameters from Hiera, and include that profile at a higher level). Each profile is going to do Hiera lookups which should give you the ability to provide different data for different host types (i.e. different data on a per-environment level, or however you lay out your Hiera hierarchy), and that data will be passed directly to class that is declared. Finally, each component module will accept parameters as variables internal to that module, default parameters/variables to sane values in the ::params class, and use those variables when declaring each resource throughtout its classes.

*    Roles abstract profiles
*    Profiles abstract component modules
*    Hiera abstracts configuration data
*    Component modules abstract resources
*    Resources abstract the underlying OS implementation

Choose your level of comfortability

The roles and profiles pattern also buys you something else – the ability for less-skilled and more-skilled Puppet users to work with the same codebase. Let s say you use some GUI classifier (like the Puppet Enterprise Console), someone who s less skilled at Puppet looks and sees that a node is classified with a certain role, so they open the role file and see something like this:

	include profiles::wordpress
	include profiles::tomcat
	include profiles::git::repo_server

That s pretty legible, right? Someone who doesn t regularly use Puppet can probably make a good guess as to what s on the machine. Need more information? Open one of the profiles and look specifically at the classes that are being declared. Need to know the data being passed? Jump into Hiera. Need to know more information? Dig into each component module and see what s going on there.

When you have everything abstracted correctly, you can have developers providing data (like build versions) to Hiera, junior admins grouping nodes for classification, more senior folk updating profiles, and your best Puppet people creating/updating component modules and building plugins like custom facts/functions/whatever.
Great! Now go and refactor…

If you ve used Puppet for more than a month, you re probably familiar with the **Oh shit, I should have done it THAT way…let me refactor this** game. I know, it sucks, and we at Puppet Labs haven t been shy of incorporating something that we feel will help people out (but will also require some refactoring). This pattern, though, has been in use by the Professional Services team at Puppet Labs for over a year without modification. I ve used this on sites GREAT and small, and every site with which I ve consulted and implemented this pattern has been able to both understand its power and derive real value within a week. If you re contemplating a refactor, you can t go wrong with Roles and Profiles (or whatever names you decide to use).
	
	
### Part 3: Dynamic environments with R10k. 

	Gary discusses the value of dynamic environments and R10k for iterating on your Puppet modules. If you haven t heard about R10k, it s an invention by another Puppet employee, Adrien Thebo. You can read about it straight from the original source right here.

see http://garylarizza.com/blog/2014/02/18/puppet-workflow-part-3/

## Best practices from cake solutions 

Why Puppet

Why Puppet, one may be tempted to fairly ask. Adopting Puppet as the configuration management technology of choice at Cake and sticking with it was a successful process. We have experimented and used alternative technologies, like Chef, or even Ansible as well.

We do think Puppet is a great tool that enables DevOps teams to follow CAMS (http://itrevolution.com/devops-culture-part-1) DevOps principles. Puppet is an "Automation" enabler and when it comes to "Sharing", PuppetForge is a very reliable source of open source modules, many of them adopted by PuppetLabs, fact that stands as a true badge of confidence in such modules.

Puppet Forge https://forge.puppetlabs.com is also a very rich repository with mature open source modules that can be used to implement enterprise solutions.

### Masterful or Masterless

Masterful or Masterless?

The debate deserves a blog post on its own. We use Puppet Masterless which leaves more room for Puppet’s friends, as per title.

### Puppet best friends

Facter

Even though for most of you readers, Facter comes without saying, we still think it is worth mentioning it as one of the core tools in configuration management with Puppet. Distribution of configuration is a unique method that has to be parameterized so the right configuration ends up on the appropriate node. We use Facter’s custom facts to achieve this. Facter can be used more extensively, but we prefer to keep it simple. The less custom facts we end up using, the more meaningful these facts tend to be.

### Puppet Patterns

###  Roles and Profiles

The distribution of configuration to nodes we consider as one of the defining aspects of configuration management with Puppet. We made use of the Roles/Profiles Pattern and the Roles/Profiles Pattern became the backbone of all our provisioning code repositories.

There is a good Craig Dunne video on the Roles/Profiles pattern http://puppetlabs.com/presentations/designing-puppet-rolesprofiles-pattern and I shall not go too deep into describing what it is, but I will rather expose what we learned while using it, through a set of best practices that we came up with amongst our team.

*    First, keep the roles simple, only include profiles and ordering inside a role.
*    Second, keep the profiles simple, reusable. Only include modules within a profile and minimal additional required logic. The profile name must be defining for the function of what the profile is used for.
*    Third, split a profile into more reusable profiles when it begins to grow too much.

Keep it clear, keep it simple!

Puppet role example:

blog.pp

	class role::blog {
		include ::profile::base
		include ::profile::a
		include ::profile::b
		include ::profile::blog

		Class[ profile::base ] ->
		Class[ profile::a ] ->
		Class[ profile::b ] ->
		Class[ profile::blog ]
	}

 

Puppet profile example:

	blog.pp

	class profile::blog( $blog_title = "default", $blog_serie_title = "default" ) {
		notify{ "Blog serie title: ${blog_serie_title}": }
		notify{ "Blog title: ${blog_title}": }
	}

Minimalistic roles and profiles proved to be the winner for us. A good design will keep many challenges away. A good design enables contributors to a project to "speak the same language" so that teams perform better. One process that we use as standard, code active reviewing, benefits as well from adhering to a good design. Active reviewing becomes faster and it trully boosts productivity.


### Configuration data

### Hiera

We like Hiera. We endorse Puppetlabs answer to the question 

Q: **Why Hiera?**, 
A: **To make Puppet better**, as detailed here https://docs.puppetlabs.com/hiera/1/

Hierarchy is equally important as the Roles/Profiles pattern. Hiera allows separation of your configuration data from your code, therefore allowing your Puppet code to be reused without significant or any rework.

Hiera enables the usage of different backends for storing the data, from yaml files to database. In case you are thinking about encrypting the data, you can use the hiera-eyaml backend, available as ready to use packages, https://github.com/TomPoulton/hiera-eyaml, or the hiera-gpg backend https://github.com/crayfishx/hiera-gpg.

This is how the hiera main configuration file would look for a basic usage:

	hiera.yaml

	---
	:backends:
		- yaml
	:yaml:
		:datadir: /etc/puppet/hieradata
	:hierarchy:
		- "%{::fact_serie}/%{::fact_role}"
		- "%{::fact_role}"
		- "%{::fact_serie}"
		- common



### Puppet modules dependency management

There are currently two main candidates for solving the dependency management job with Puppet: Librarian-Puppet and r10k. Let’s see now where these methods excel and where they can compliment each other.

The best thing about the two options at hand is they both use the same configuration format, Puppetfile.

	Puppetfile.blog

	forge "https://forgeapi.puppetlabs.com"

	mod "saz/timezone", "3.0.1"
	mod "cakesolutions/base",
		:git => "git@github.com:cakesolutions/puppet-base.git",
		:ref => "1.0.0"


#### Librarian-puppet

We prefer librarian many times for its dependency resolution mechanism. Librarian uses Puppetfiles for defining module dependencies.

Pros: Dependency Resolution, Mature product

Cons: Slow

#### r10k

r10k is prefered when speed of bringing up an environment is key. Switching from Librarian-Puppet to r10k during the lifecycle of a product is an option to consider, especially considering that both methods can use the same configuration format, the Puppetfile.

Pros: Fast, Makes use of Puppetfile format.

Cons: Missing Dependency Resolution

r10k author’s own blog on r10k which I found a good read: http://somethingsinistral.net/blog/rethinking-puppet-deployment/


### Testing

Configuration Management code requires testing as well, there is no escape.

To start with, style checking can be achieved with Puppet-lint http://puppet-lint.com

Functional integrated testing can be achieved by smoke testing environments. There are different tools that enable such mechanisms.


### Mixing it all together

Knowing what is in a name, having the tools and a set of best practices on using CloudFormation and applying the configuration management tools, described in this post makes environments provisioning a graceful and repeatable process.

Here is a very basic skeleton to best exemplify how the provisioning structure using Puppet, Hiera, Facter should look like: https://github.com/cornelf/puppet-and-friends.


### Conclusion

Configuration Management can be approached with a set of tools that consitute themselves into a common method that can be applied consistently. Provisioning code consistency can be achieved only by adhering to good patterns and by using the elementary tools that always deliver value and lead the way to the solution from early stages in a project.



## Common trap

 
