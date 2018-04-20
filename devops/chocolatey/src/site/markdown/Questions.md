# Questions and best practices 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


## Questions :

* internalisation of packages (automation)
* dependencies management (via nuget)
  
* license management of software installed 
* roll back strategy and update
* security (checkshum is enough ?)
* anti virus 
* custom configuration 
* packages storage
* syncrhonisation expected / real installed
* 32 / 64 bits packages
* Powershell V2 ? 

* test application removal 
	- check environment variables
	- check registry
	- check folder
	- what about application data ? 
* chocolatey for what ?
	* installation
	* registry
	* sheduled task / crontab
	* environment variables
	* user en groups
	* services
	
	
* installation order
	-ex : update jdk8, remove jdk7 (32b et 64b), restart all applications using java ? 

	
## Chocolatey and nexus



Here is how you do it:

* Create a NuGet proxy repository to points to https://chocolatey.org/api/v2/. This will allow you to cache packages from chocolatey.org which can come in handy if it goes down during a deployment.
*    Create a NuGet hosted repository. This is where you will publish your private packages to.
*    Create a group repository that contains the above repositories. This what you will set --source flag to when installing package.

The NuGet tab under hosted repo created above will have the Package Source and Personal API Key.

So if hosted repo has:

	Package Source = "https://example.com/nexus/service/local/nuget/choco-releases/" Personal API Key = "d8471cc1-d350-3e45-a0c2-95d0b938e1d9"

The call to package and publish your private mypackage package would look like this:

	choco pack
	choco push --source "'https://example.com/nexus/service/local/nuget/choco-releases/'" -k="'d8471cc1-d350-3e45-a0c2-95d0b938e1d9'"

To install packages from both private and public sources, use the group repository as the source. The NuGet tab under group repo created above will have the Package Source to use.

So if group repo has:

	Package Source = "https://example.com/nexus/service/local/nuget/choco-all/"

Then the call to install both your private and publicly available packages would look something like this:

	choco install jdk8 mypackage --source "'https://example.com/nexus/service/local/nuget/choco-all/'" 

Where jdk8 package is not in the private repo, so Nexus will pull it from chocolatey.org, cache it in the proxy repo, then send it on to where choco install is called.
	
	
## Common traps

### passing arguments from puppet to chocolatey 

need to define an array (common powershell trap) : 

	install_options => ['-InstallArguments', '"/Features:\'Blend','LightSwitch','VC_MFC_Libraries', 'OfficeDeveloperTools', 'SQL', 'WebTools', 'Win8SDK', 'SilverLight_Developer_Kit', 'WindowsPhone80\'', '/ProductKey:AAAA-AAAA-AAAA-AAAA-AAAA"'],

#### detailed example
	
The underlying installer may need quotes passed to it. This is possible, but not as intuitive. The example below covers passing **/INSTALLDIR="C:\Program Files\somewhere"**.

For this to be passed through with Chocolatey, you will need a set of double quotes surrounding the argument and two sets of double quotes surrounding the item that must be quoted (see how to pass/options/switches). This makes the string look like **-installArgs "/INSTALLDIR=""C:\Program Files\somewhere"""** for proper use with Chocolatey.

Then for Puppet to handle that appropriately, we must split on every space. Yes, on every space we must split the string or the result will come out incorrectly. So this means it will look like the following:

	install_options => ['-installArgs',  '"/INSTALLDIR=""C:\Program', 'Files\somewhere"""']

Make sure you have all of the right quotes - start it off with a single double quote, then two double quotes, then close it all by closing the two double quotes and then the single double quote or a possible three double quotes at the end.

	package {'mysql':
		ensure          => latest,
		provider        => 'chocolatey',
		install_options => ['-override', '-installArgs', '"/INSTALLDIR=""C:\Program', 'Files\somewhere"""'],
	}
	
	
