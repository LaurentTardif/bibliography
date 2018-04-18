# PRO version


<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Package throttle


By default, Chocolatey downloads packages and any resources the packages use as fast as it can. In most cases, this is exactly the behavior you want. However, low bandwidth areas would be overwhelmed with this behavior and so you need to slow Chocolatey down. Package Throttle covers these low bandwidth scenarios automatically.

### Usage

Adding the option or config setting for maximum download rate will slow down the speed a package and any resources are downloaded to a particular machine. This is done in "bit per second" (b/s or bps), which is a common unit of data transmission rate (bandwidth) in computers. The symbol bps is often pronounced "bips."

A good way to find what you need is to translate from Kbps to bits: Google Kbps to bps or KB/s (bytes) to bits: Google KB/s to bps.
Requirements

    Chocolatey (chocolatey package) v0.10.7+.
    Chocolatey Licensed Edition.
    Chocolatey Licensed Extension (chocolatey.extension package) v1.10.0+.

## Virus 


### VirusTotal

We've teamed up with the amazing VirusTotal to offer a second opinion to your built-in anti-virus solution. When your packages download content from the internet, Chocolatey Pro will automatically check the executables prior to running any content. If a certain number of positives have been identified (configurable), Chocolatey will fail the install automatically (but can be overridden).

VirusTotal scans binaries against over 50 different anti-virus scanners. Chocolatey will use the information based on sending a SHA256 checksum of the binary to VirusTotal and determining what the scans have determined.

By default the virus scanner is already enabled and set to VirusTotal for Pro . Organizations usually are more wary about reaching out to the internet, so the feature is set to Generic and the virus scanner may not be enabled by default (because it needs configuration). If you need to configure the virus scanner to use VirusTotal, please run the following two commands:

    choco config set virusScannerType VirusTotal
    choco feature enable -n virusCheck

Because some scanners can be quite aggressive and may falsely identify a binary as a false positive for malware, Chocolatey doesn't flag a binary until a certain number of scanners have identified the binary as malware. This number defaults to 5 in the configuration. This means 5 AV scanners need to flag the binary for Chocolatey to stop and fail the install/upgrade. You can adjust this value in two ways, by adjusting the configuration and per choco command.

To adjust the configuration value, which will globally set the number, please run the following replacing ## with the value you want as a minimum:

    choco config set virusCheckMinimumPositives ##

You can also adjust the default number of positives prior to flagging a binary with the virusCheckMinimumPositives configuration value. By default it is set to 5, which means 5 AV scanners need to flag the binary for Chocolatey to stop and fail the install/upgrade.

If you need to override the minimum number of positives just for one install/upgrade, you can do that by passing --virus-positives-minimum=VALUE with your install or upgrade commands.


### Generic

If you are an organization and you do not want to reach out to the internet for virus scan verification, you will want to take advantage of hooking Chocolatey up to your existing AV solution.

Chocolatey will just need to know the full path to the AV command line executable, the arguments to pass, and valid exit codes (comma separated).
Then you will get the same benefit of runtime virus checking as an added benefit on top of the protection it is already providing.

If you need to configure the virus scanner to use your built-in antivirus solution, please run the following two commands:

    choco config set virusScannerType Generic
    choco config set genericVirusScannerPath "full path to av command line"
    choco config set genericVirusScannerArgs "[[File]]"
    choco config set genericVirusScannerValidExitCodes "0, ##"
    choco feature enable -n virusCheck

The genericVirusScannerPath should be the full path to the anti-virus command line executable. What we are looking for is the command line interface to the scanner.

In genericVirusScannerArgs, these are the arguments that Chocolatey will pass to the AV console. Chocolatey will automatically replace [[File]] with the full path to the binary that it is scanning.

In genericVirusScannerValidExitCodes, these are exit codes that indicate that a binary is okay. This defaults to 0, but you can configure this with as many exit codes as necessary by adding them to the configuration as comma separated values.

## CDN cache

### Important Information for Software Vendors

When our customers download content from you, we may cache a copy of that content on our customer CDN to ensure that later our licensed customers will be able to access that content again. This allows predictable and reliable installations of packages. Like most CDNs, no changes are made, we are simply ensuring the content stays available for our private customers. We are not modifying, selling, renting, leasing, lending, or sub-licensing this content in any way.

This private content cache is only available as a service for our licensed customers.

If you have questions or prefer to opt out, please contact us and select "Software Vendor Opt Out" (vendor requests only please).

### Usage

This is mostly automatic, but in case the feature is turned off, you may turn it on by setting the feature downloadCache (choco feature enable -n downloadCache).

To see if a package has content cached on the CDN, you can run choco info packagename and it will give you detailed information pertaining to a package.



## Package reducer 

### Automatic , Usage

When you normally create packages that embed or download resources, the impact on a system includes the following:

    A nupkg file size takes up some space - this is like a fancy zip file and may contain binaries
    Files extracted or downloaded to the package directory - this may include zips and installers
    The actual install location if using an installer
    MSI cached by Windows - Windows caches the complete MSI binaries

Typically with this in mind, the size of nupkg could account for an up to 4x impact on a system considering the above. So we typically recommend organizations set a comfortable threshold for package sizes (around 500MB) before splitting out binaries to downloaded resources.

Package Reducer nearly removes that need as it reduces the nupkg file to 5KB by removing all the embedded files. Then it takes it a step further and removes zip and installers from the package directory automatically (configurable by a feature switch, see Setup).

When you turn on Package Reducer, the first two items above no longer take up any significant space. This can reduce space usage in the order of GBs for some installations of Chocolatey.

With Package Reducer:

    nupkg file is reduced to 5KB or less, no matter the size on install
    zips / installers are automatically removed from the package directory if they are found

The following file extensions are removed automatically:

    7z / zip / rar / gz / tar / sfx
    iso
    msi / msu / msp
    exe files if they are detected to be an installer

So the space usage impact changes to what you'd normally experience outside of Chocolatey:

    the actual install location (package directory, Program Files, etc)
    MSI cache for MSIs

### Requirements

    Chocolatey (chocolatey package) v0.10.7+.
    Chocolatey Licensed Edition
    Chocolatey Licensed Extension (chocolatey.extension package) v1.12.0+.

###Setup

To turn on Package Reducer, you need to run the following:

    choco upgrade chocolatey.extension <options>
    choco feature enable -n reduceInstalledPackageSpaceUsage
    If you want to limit to just nupkg files being reduced and not automatically removing zips and installers, run the following: choco feature enable -n reduceOnlyNupkgSize

## Synchronize with Programs And Features (Licensed Editions Only)

Chocolatey maintains its own state of the world, while Windows maintains the state of Programs and Features. If an application is upgraded or uninstalled outside of Chocolatey, such as is the case with Google Chrome and its auto updating utility, Chocolatey doesn't know about the change. The synchronize feature keeps Chocolatey's state in sync with Programs and Features, removing possible system-installed state drift.

### Automatic Synchronize

In all licensed editions, Chocolatey syncs the state of already installed packages to the state of the software they are linked up with - so if the software is uninstalled or upgraded outside of Chocolatey, it removes the package or notes the upgrade. This behavior is known as automatic synchronization or autosync for short. There is also choco sync, which is different and is covered in Sync Command below.

There is an important distinction between packages and software to understand with Chocolatey. Chocolatey manages packages (nupkg files), and packages manage software. Many packages are what we call "installer packages" - packages that manage an installer and an installation into Programs and Features. Once that software is in Programs and Features, it could be manipulated outside of Chocolatey. If it is removed without the command going through Chocolatey, open source Chocolatey will still have the package installed (which is technically correct from a package manager state, but incorrect based on expectations). With open source you end up with a disconnect if you are not careful. Licensed editions of Chocolatey are able to see system state and correct this discrepancy with autosync by updating the state of the package based on what has occurred outside of Chocolatey.

For software management, which encompasses package management and above that for Windows, the state of the software is really what should be reflected with those installed packages. So if that software has been uninstalled from Programs and Features or upgraded outside of Chocolatey, autosync removes those packages to match that state or notes the upgrade (note it doesn't change the version of the package - that's much trickier to get correct).

As commercial editions lean more towards software management, they have much better system integration. Autosync is great for ensuring that Chocolatey matches the state of changes on the system for what the packages it is tracking.

## Chocolatey Agent Service (Business Editions Only)

Empower your users and give your IT folks the precious gift of time to invest into taking your organization to the next level!

The Chocolatey Agent service allows you to go further with your software management, bringing Chocolatey to desktop users in organizations that have controlled environments. This provides users in controlled environments more empowerment and instant turn around on required software. This frees up IT folks and admins time to spend on making the organization even better.

### Usage

The Chocolatey agent enables two simultaneous modes of operation, one as an agent for a central console (upcoming) and the other as a background service for use in controlled environments. You can configure one or both modes.
Requirements

*    Chocolatey (chocolatey package) v0.10.3+ - v0.10.4+ for better compatibility. Chocolatey v0.10.7+ is recommended and required in newer versions.
*    Chocolatey for Business (C4B) Edition
*    Chocolatey Licensed Extension (chocolatey.extension package) v1.8.4+. For chocolatey-agent v0.5.0+, licensed extension v1.9.0+. For chocolatey-agent v0.7.0+, licensed extension v1.11.0+.
*    Chocolatey Agent Service (chocolatey-agent package) - 0.7.0+ is recommended.

For use with Chocolatey GUI, you must be on Chocolatey v0.10.7+, Chocolatey Licensed Extension v1.11.0+, and Chocolatey Agent v0.7.0+.

### Setup

To install the Chocolatey agent service, you need to install the chocolatey-agent package. The Chocolatey Agent is only available for business edition customers to install from the licensed source (customers trialling the business edition will also be provided instructions on how to install).
Background Mode Setup

To set Chocolatey in background mode, you need to run the following:

*    choco upgrade chocolatey-agent \<options\> (see agent install options)
*    choco feature disable --name=showNonElevatedWarnings - requires Chocolatey v0.10.4+ to set.
*    choco feature enable --name=useBackgroundService
*    You also need to opt in sources in for self-service packages. See choco source (and --allow-self-service). You can also run choco source -? to get the help menu.
*    * OPTIONAL (not recommended): Alternatively, you can allow any configured source to be used for self-service by running the following: choco feature disable --name=useBackgroundServiceWithSelfServiceSourcesOnly (requires Chocolatey Extension v1.10.0+). We do not recommend this as it could be a security finding if you shut it off.
*    OPTIONAL (highly recommended): If you want self-service to apply only to non-administrators, run choco feature enable --name=useBackgroundServiceWithNonAdministratorsOnly (requires Chocolatey Extension v1.11.1+). Do understand this means that a real non-administrator, not an administrator in a non-elevated UAC context (that scenario will go the normal route and will not go through background mode).
*    OPTIONAL (varied recommendations): If you want to configure custom commands (not just install/upgrade), use something like choco config set backgroundServiceAllowedCommands "install,upgrade,pin,sync" (with the commands you want to allow, requires Chocolatey Extension v1.12.4+). See commands consideration below.
*    OPTIONAL (highly recommended): For use with Chocolatey GUI, you need Chocolatey Extension v1.12.4+, and at least Chocolatey GUI v0.15.0. Uninstall any version of the GUI you already have installed first, then run choco upgrade chocolateygui -y --allow-downgrade (you will also need at least .NET 4.5.2 installed)
*    DOES NOT WORK WITH UAC, DO NOT USE UNTIL FIX IS ANNOUNCED! OPTIONAL (recommended if you use installers that are not completely silent): If you want self-service to interactively manage installations, run choco feature enable --name=useBackgroundServiceInteractively (requires Chocolatey Extension v1.12.10+). This requires that you use the ChocolateyLocalAdmin account with the Chocolatey-managed password as passwords are not stored and the service would need to produce that at runtime. There are some security considerations and why this is not turned on by default. Please see interactive self-service consideration.

NOTE: Once you are all setup, please review the Common Errors and Resolutions section so you will be familiar if you run into any issues with working with sources.

###  An example script:

This carries our typical recommendations, but you could adjust from above.

	choco upgrade chocolatey-agent -y
	choco feature disable --name="'showNonElevatedWarnings'"
	choco feature enable --name="'useBackgroundService'"
	choco feature enable --name="'useBackgroundServiceWithNonAdministratorsOnly'"
	
	# TODO: opt in your sources with --allow-self-service - run choco source -? for details

### Best practices in scripts are noted here:

*        Use upgrade instead of install - upgrade is more making the script reusable when newer versions are available.
*        Always use -y to ensure nothing stops and prompts for more than 30 seconds.
*        When using options prefer a longer name (--name versus the short -n) to make the scripts more self-documenting
*        When using options that have a value passed, add an = between and surround the value with "''" (--name="'value'"). This ensures that the argument is not split between different versions/editions of Chocolatey. This also ensures that values like . and \\ are not escaped by PowerShell.

### Chocolatey Agent Install Options

Starting with Chocolatey Agent v0.8.0+, the service will install as a local administrative user ChocolateyLocalAdmin by default (and manage the password as well). However you can specify your own user with package parameters (or have it use LocalSystem). Using a local administrator account allows for more things to be installed without issues. It also will allow easier shortcuts and other items to be put back on the correct user (the original requestor). You can specify a domain account as well. Prior to v0.8.0, Chocolatey Agent would install as LocalSystem (SYSTEM) and would require additional customization.

NOTE: If you are using file shares for sources, you may want to ensure the account or computer has network access permissions for the file share(s).

###  Package Parameters:

*    /Username: - provide username - intead of using the default 'ChocolateyLocalAdmin' user.
*    /Password: - optional password for the user.
*    /EnterPassword - receive the password at runtime as a secure string
*    /UseDefaultChocolateyConfigUser - use the default username from Chocolatey's configuration. This may be LocalSystem.
*    /NoRestartService - do not shut down and restart the service. You will need to restart later to take advantage of new service information.

### Chocolatey Managed Password

When Chocolatey manages the password for a local administrator, it creates a very complex password:

*    It is 32 characters long.
*    It uses uppercase, lowercase, numbers, and symbols to meet very stringent complexity requirements.
*    The password is different for every machine.
*    Due to the way that it is generated, it is completely unguessable.
*    No one at Chocolatey Software could even tell you what the password is for a particular machine without local access.

See FAQ below for more discussion on security aspects.

### Command Customization Consideration

Starting with Chocolatey Licensed Extension v1.12.4, you are allowed to configure what commands can be routed through the background service. Please note that Chocolatey Licensed defaults to install and upgrade as that is the most secure experience. However you can add uninstall and some other commands as well. Uninstall does have some security considerations as it would allow a non-administrator to remove software that you may have installed, including the background service itself.

Available Commands:

*    info - do not add if you want sources hidden from non-admins
*    list/search - do not add if you want sources hidden from non-admins
*    outdated - do not add if you want sources hidden from non-admins
*    install - default
*    upgrade - default
*    uninstall - keep in mind there may be security implications for this
*    optimize
*    pin
*    sync

Blacklisted Commands:

*    config
*    feature
*    source
*    apikey

Chocolatey does not allow for configuration changing commands to be routed through the background service as that would allow users to be able to change configuration and that could be detrimental. For instance, a user could add a local source with a package they've created that promotes themselves to an administrator (escalation of privilege). As that constitutes a security issue, we do not allow it.

For the same reason, we do not recommend allowing sources you do not control to be allowed for self-service.
Interactive Self-Service Consideration

When using the self-service with useBackgroundServiceInteractively, it is similar to "Run As". Microsoft used to allow Windows services to interact with the desktop but removed the functionality (of "Allow Service to Interact with the Desktop") in Windows Vista and limited all services into what is known as Session 0 isolation. In Session 0, those services can access a desktop, but not the interactive user's desktop (at least it is very, very difficult to do so). Microsoft did this as a security consideration as allowing a privileged account to run executables in a non-administrative user context opens the potential to allow a non-admin to gain privileges to a system (known as escalation of privilege). Depending on what is allowed for folks to install, as long as those package installations do not open a command shell and wait for user input, the potential is quite low. However it is something to keep in mind and assess before turning on this feature (and why it is off by default).

However sometimes you work with installers that refuse to be silent. You might get them down to unattended (meaning they still have required input or window pop ups, but they are all handled and automatically closed), but they won't get to silent without a different option. Here are some options from most preferred to least preferred in getting those installers that are not fully silent to work:

*    (Silent) Find a portable version of the tool that doesn't run a badly behaved installer. Looked for a zipped up version, you can easily create a Chocolatey package from these.
*    (Silent) Find an alternative that does the same thing, but has silent installers. It's a consideration, but maybe not something you can or are willing to explore. Moving on...
*    (Silent) Use MSI repackaging to produce a completely silent installer. It works by recording everything that happens when you run the install and creates an MSI to do the same. This would need to be done for every version. There are tools out there that can do this, some of which are VERY expensive.
*    (Unattended/Interactive) Use AutoHotKey or AutoIT (or some other tool) to automate the key presses and values being sent to the interfaces. This can be brittle, but can typically be quickly implemented.

If you must run in the context of working with "unattended", non-silent installations, you need to take the above security consideration into account if you want to be able to manage those installations using the background service.
Chocolatey Background Service / Self-Service Installer

When an administrator installs the agent, they can configure Chocolatey to use background mode so that non-administrators can still perform installations of approved software as configured by an administrator.

Why this is desirable:

*    Users do not need to be administrators but are still empowered to install and upgrade software (functions are configurable with Chocolatey Extension v1.12.4+)
*    Users can only run install and upgrade in an administrative context by default. This is configurable to other commands as of Chocolatey Licensed Extension 1.12.4+..
*    Shortcuts, desktop icons, etc created through Chocolatey functions will end up with the proper user (still coming).
*    Users can only install approved software based on admin configured sources.
*    This frees up precious IT bandwidth to work on other engagements.
*    Empowers users, so they feel more in control.

This makes for happy users and happy admins as they are able to move quicker towards a better organization.
Self-Service Roadmap:

*    Admins will be able to schedule when upgrades occur (maintenance windows).
*    Admins will be able to configure what commands can be run through the background service. Completed with Chocolatey Extension v1.12.4.
*    Admins will have more granular control of what certain users can install.

### Chocolatey Central Console

NOTE: The console is estimated to be available by end of 2017. All notes here are what we expect, but the standard disclaimer of availability and features apply.

Chocolatey will have a central core console that will allow you to manage your environments. You will need the Chocolatey Agent installed on all machines you wish to manage centrally.
Chocolatey Central Console Roadmap

The console will allow:

*    Centralized Software Management for your entire organization.
*    Centralized reporting of software.
*    Know immediately what software is out of date and on what machines.
*    Know within seconds the entire estate of software and what versions are installed.
*        **Including zips and archives** that do not show up in Programs and Features
*       **Including internal software** that does not show up in Programs and Features
*    Adhoc reporting for a particular machine or set of machines
*    Run arbitrary Chocolatey commands against one or more machines
*    See how many machines you are actively managing in your organization
*    More...

When deployed through Chocolatey.

## Package Builder - Create Packages Automatically From Installers (Business Editions Only)

Chocolatey's Package Builder allows you to create fully ready to go software deployments for Windows in 5–10 seconds! There is nothing faster than Chocolatey when it comes to preparing software for an unattended deployment across your organization.

    Generate packages based on installers and zips
    Generate packages by reflecting over Programs and Features
    Generate packages from the Package Builder UI

Creating packages is a pretty quick process as compared to manually installing software on multiple machines. There is still some time involved to create a package. Even the best packagers take between 5–10 minutes to create a package. Chocolatey's Package Builder creates high quality packages in 5–10 seconds when pointed to native installers and zips! You can even point package builder to both a 32-bit and 64-bit url and seconds later you have a fully functioning package using all local/embedded resources!

Chocolatey for Business is able to inspect an installer and determine silent arguments and complete packaging components for you, saving you hours of time in packaging and maintaining software!

## Package Internalizer

Automatically Internalize/Recompile Existing Packages (Business and MSP Editions Only)

There are thousands of existing packages on the community repository that are a tremendous resource when it comes to creating packages that have software that can sometimes be tricky! Unfortunately you may be wary of using those packages without changes because many of those packages are subject to distribution rights and thus have an internet dependency (which creates both trust and control issues). There is a process for downloading and internalizing packages to use internal or embedded locations for that software that is called internalizing (also known as recompiling).

Chocolatey for Business is able to automatically download packages and resources, edit the scripts, and recompile packages to internalize and remove internet dependencies from those packages, saving you hours of time in manually internalizing/recompiling packages!

Usage

	Call choco download with options including the name of an existing package for Chocolatey for Business to download and download all existing remote resources.

## Package Audit (Business Editions Only)

    Know who installed what package and when.

Reporting is very important, and auditing not only when your installations occurred but who installed them can be critical. There is nothing that presents this kind of information as easily as you will be able to gather it with Chocolatey for Business (C4B) and Package Audit.

When calling choco list -lo, add --audit to see information on who installed and when.
Requirements

    Chocolatey (chocolatey package) v0.10.7+.
    Chocolatey for Business (C4B) Edition.
    Chocolatey Licensed Extension (chocolatey.extension package) v1.12.0+.
