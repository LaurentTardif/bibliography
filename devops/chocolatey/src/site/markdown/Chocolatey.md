# Chocolatey

## Introduction to Chocolatey

Chocolatey is a software management tool that is also a package manager. It functions fantastically well when the runtime software is all included in the package and it doesn't make use of native installers. However to approach the Windows ecosystem a package manager also needs to know how to manage actual software installations, thus why Chocolatey does that as well. For publicly available packages, copyright keeps from having binaries embedded in packages, so Chocolatey is able to download from distribution points and checksum those binaries.

## Nuspec

The Chocolatey Windows package manager uses the same infrastructure as NuGet, the Visual Studio package manager by Outercurve Foundation (sponsored by Microsoft). Therefore packages are based on the same principles. One of those is a package description (specification) in xml format, known as the Nuspec.

The Nuspec contains basic information such as the version, license, maintainer, and package dependencies. Chocolatey includes additional optional functionality on top of NuGet's Nuspec format - the best way to determine currently supported features is to create a test package, and look at the generated nuspec file.

 choco new testpackage

Note: If your package uses recently introduced functionality, you might want to include chocolatey as a dependency with the version being the lowest version that has the introduced functionality. Otherwise the installation could fail for users with an older version of Chocolatey installed.

You can indicate the Chocolatey dependency like any other dependency. E.g.:

```{}
<?xml version="1.0" encoding="ISO-8859-1"?>
    <dependencies>
      <dependency id="chocolatey" version="0.9.8.21" />
    </dependencies>

```

Logically, the version is based on the lowest compatible version. But if you don't know and used a lot of sorcery in your package, depend on the version of Chocolatey that you succesfully tested your package on.

## Summary to create packages

See What are Chocolatey Packages? first.

Note: When you host internal packages, those packages can embed software and/or point to internal shares. You are not subject to software distribution rights like the packages on the community feed, so you can create packages that are more reliable and secure.

First you should determine if you are making a self-contained package or (also) using automation scripts. You should also consider creating automatic packages for the best maintainability over time.

    Run choco new -h to get a feel for what you can pass e.g choco new bob (to create a package named bob)
    Note all the rules below. This will help you, especially in publishing to the community feed (https://chocolatey.org)

## Self-Contained?

If you have a self-contained package, you can remove the automation scripts
entirely and just include the runtime executables, they will automatically get shimmed,
which puts them on the path. Ensure you have the legal right to distribute
the application though. You should read up on the Shim Generation section
below though to familiarize yourself on what to do with GUI applications
and/or ignoring shims (also known as batch redirects).
Including the Software Installer in the Package

Sometimes called embedding the binaries, there are functions in the automation scripts you can call that will use the installers directly from the package instead of downloading first. This makes for the most reliable and deterministic packages, but ensure you have the legal right to distribute the software first if publishing to a public location (like the community package repository).

## Automation Scripts

You have a powerful use of Chocolatey, as you are using PowerShell. So you
can do just about anything you need. Choco has some very handy built-in functions that you can use, these are sometimes called
helpers.

## Quick Start guide

If you think you got what it takes and just want to know the basic steps to get a package out, there is a special Quick Start Guide for you. NOTE: This doesn't exempt you from observing the rules, requirements and guidelines (noted below).
Rules to be observed before publishing packages

There are a few rules that you have to follow before pushing packages to chocolatey.org:

* **Don't package illegal software.** Packages of software that is illegal in most countries in the world are prohibited to publish on Chocolatey.org. This applies in particular to software that violates the copyright, pirated software and activation cracks. Remember that this also affects software that is especially designed to accomplish software piracy.
* **Do not include software in a package that you don't have the right to distribute.** Please see Distribution Rights for more information. Any package found not in compliance with this will be removed immediately. Instead you can download binaries from official distribution locations at runtime.
* **Packaging commercial or trial software?** Clearly state this in the package description. Does it require an activation key? Is there a trial period if you don't have a key? How long is this trial period?
* **Only post publicly relevant packages.** You should consider whether this package is useful for others. If that is not the case, it shouldn't be published on Chocolatey.org. Reasons for that can be if the package would require a very customized configuration. You can host your personal packages on MyGet and still be able to install them with Chocolatey using the -source parameter.
* **Do not publish junk or malware packages**.
* **Don't package software containing malware**. Packages of software that comes with bundled adware, spyware or other unrelated software that installs even in silent mode, are not allowed. But if you can figure out how to install the desired software without this unrelated software, it is allowed to publish the package. That can be accomplished for example with additional command line switches or by adding specific values to the registry. Examples of packages which make use of this are PDFCreator and CCleaner.
* **Don't package software that is already packaged**. Use the search function in the Chocolatey.org gallery and look if there is already a package for the desired software. If you would like to improve the already existing package or if you have suggestions, just contact the package maintainer or open a pull request at the maintainer's package repository.
* **Don't include other required software** if there's a package of it. If a package requires other software of which there is already a package, the already existing package should be used as dependency instead.
* **Split dependencies into multiple packages**. Try to split up packages as much as possible. If for example a program comes with additional modules/installers that are optional, make different packages for them instead of including all the things into one package. This idea is already widely applied for Linux packages, because it leads to a more lightweight system and reduces potential issues and conflicts.
* **Use a simple intuitive lowercase name for the package**. See the package naming guidelines for details. (If you are a reviewer/moderator, this is considered a guideline).

Is your package unqualified for the Chocolatey feed, but you like to be able to install it through Chocolatey? Don't worry, you can always host your package for free on MyGet. See Hosting Chocolatey Packages on MyGet.

## Character encoding

Use the UTF–8 character encoding for the *.nuspec and *.ps1 files. If you don't respect this rule, some characters are not displayed correctly in the Gallery on Chocolatey.org, because the Gallery assumes UTF-8.

Do not save your *.nuspec files with a Byte Order Mark (BOM). A BOM is neither required nor recommended for UTF-8, because it can lead to several issues.

PowerShell scripts need to be saved in UTF–8 with BOM. PowerShell is ignoring the standards and needs a BOM in order to recognize scripts as UTF-8. Otherwise it processes non ASCII characters incorrectly.

Don't use the default Windows Editor. In addition to its lack of features, it can't even save UTF-8 files without BOM. Alternatives:

* Visual Studio Code
* Notepad++
* Geany

Use this XML declaration:

```{}
<?xml version="1.0" encoding="ISO-8859-1"?>
or
<?xml version="1.0" encoding="utf-8"?>.

```

Note: There is a lot of confusion in the world of character encodings: For example, ANSI is an incorrect term for the internal Windows character encodings, e.  g. Windows-1252. But you should not use this encoding family anyway.

## What version of the software should I package?

The main release of a product versions are usually sufficient. If there are also beta versions available and you would rather have that, then please create both the official release and the beta (and set the beta as a prerelease when pushing the item to chocolatey.org). Regular users of packages may want to use official releases only and not betas. NOTE: Both of these have the SAME package id, just different versions.

## Okay, how do I create packages?

There are three main elements to a Chocolatey package. Only the nuspec is required (#1 below).

* Nuspec
* chocolateyInstall.ps1 - check out the helper reference
* any application files to include (it is highly suggested that you are the author in this case or you have the right to distribute files). EXE files in the package/downloaded to package folder from chocolateyInstall.ps1 will get a link to the command line.
* chocolateyUninstall.ps1, for uninstalling your package. See helper reference for functions available in your script.

Note: Please maintain compatibility with Posh v2. Not every OS we support is on Posh v2 (nor comes OOB with Posh v3+). It's best to work with the widest compatibility of systems out there.

There is a video showing the creation of a package: http://www.youtube.com/watch?v=Wt_unjS_SUo
The video is a bit outdated in showing the contents of the chocolateyInstall.ps1. Have a look at what the chocolateyInstall.ps1 looks like now:

	$packageName = 'windirstat'
	$fileType = 'exe'
	$url = 'http://prdownloads.sourceforge.net/windirstat/windirstat1_1_2_setup.exe'
	$silentArgs = '/S'

	Install-ChocolateyPackage $packageName $fileType $silentArgs $url

During which scenarios will my custom scripts be triggered?

The table below shows which scripts are available, and which command(s) will cause them to be run.

	Script Name 						Install 	Upgrade 	Uninstall
	chocolateyBeforeModify.ps1 						Yes 		Yes
	chocolateyInstall.ps1 				Yes 			Yes
	chocolateyUninstall.ps1 										Yes

Note: In the upgrade scenario, the chocolateyInstall.ps1 script will be the one included in the new package. The chocolateyBeforeModify.ps1 script will be the one from the previously installed package.

The chocolateyBeforeModify.ps1 script will only be executed if using choco version 0.9.10 or later.

## Nuspec

The Chocolatey Windows package manager uses the same infrastructure as NuGet, the Visual Studio package manager by Outercurve Foundation (sponsored by Microsoft). Therefore packages are based on the same principles. One of those is a package description (specification) in xml format, known as the Nuspec.

The Nuspec contains basic information such as the version, license, maintainer, and package dependencies. Chocolatey includes additional optional functionality on top of NuGet's Nuspec format - the best way to determine currently supported features is to create a test package, and look at the generated nuspec file.

## choco new testpackage

Note: If your package uses recently introduced functionality, you might want to include chocolatey as a dependency with the version being the lowest version that has the introduced functionality. Otherwise the installation could fail for users with an older version of Chocolatey installed.

You can indicate the Chocolatey dependency like any other dependency. E.g.:

```{}
<?xml version="1.0" encoding="ISO-8859-1"?>

    <dependencies>
      <dependency id="chocolatey" version="0.9.8.21" />
    </dependencies>

```

Logically, the version is based on the lowest compatible version. But if you don't know and used a lot of sorcery in your package, depend on the version of Chocolatey that you succesfully tested your package on.

See also: NuGet Version Reference

## But for real, how do I create a package?

    Generate new package:
        choco new -h will get you started seeing options available to you.
        Once you figured out all of your options, you should move forward with generating your template.

## Install Only On Some Versions of Windows

Right now if the software the package installs is only supported on particular versions of Windows, you should absolutely fail the package. An installed package indicates success. If you pass a warning message but don't also throw an error, that means the package installed successfully. Folks using the package are going to be confused because they will then expect that the underlying software is also installed. The software itself may throw a cryptic error, which will lead to questions from the community about why it is broken (when it is just unsupported). Do yourself a favor and check the version of Windows and throw an error if it is not a supported version. Under no circumstances should you bypass with a warning, because a warning is still a success.

There is at least one noted exception to this and that is low-level packages that are meant as dependencies that need to be present even if they do not install anything. These are things like KBs that only need to be installed on some versions of Windows. If the package failed and it was a dependency of a higher level package that installed software, it would cause issues attempting to install that software on different versions of Windows. Since about 5% of the packages apply to this exception, stick with the above thoughts for packages.

NOTE: We will ultimately enhance the nuspec and take care of this for you automatically. Until we get there, follow the above avenue.

## Installation Paths

As the package maintainer, you decide where the packaged application is installed or extracted to. Depending on your type of application (see "What distinction does Chocolatey make between an installable and a portable application?" at the bottom of the FAQ) there are a couple of suitable locations (not listed in any particular order):

### 1. The default installation path of your .msi/.exe setup file

The original creator probably had a reason for choosing a specific default installation path.
If you think, the user should be able to customize this path and you, the package maintainer, know how to pass a custom path on to the installer, then you should use %ChocolateyBinRoot%.

### 2. The package directory in %ChocolateyInstall%\lib\mypackage

You can extract the application within the package directory itself (or even ship an extracted version with the package). This allows Chocolatey to automatically find executables and put those on %path%.

### 3. Path provided by the Get-BinRoot helper - will be deprecated later (closer to v1) but okay to use for now

The path returned by the helper Get-BinRoot can be used as the parent directory for the installation. Get-BinRoot will return the value of the environment variable %ChocolateyBinRoot%. If the value does not contain a drive reference, the system drive will be prepended. If the environment variable is not set, the default path (C:\Tools C:\Chocolatey\bin) will be returned.

As an example, MinGW uses %ChocolateyBinRoot%. If the environment variable is not set, it will be set to c:\tools and MinGW will install to C:\Tools\MinGW by default. If %ChocolateyBinRoot% is set to "C:\Common\bin", MinGW installs to C:\Common\bin\MinGW.

%ChocolateyBinRoot% gives the Chocolatey user a way of controlling where packages are installed. If you want to allow customizing the installation path, then this is currently the way to go.
Make it clear in the package description

No matter how you decide, you are advised to state the default installation directory in your package description. This prevents confusion about where the application will end up being installed.

If you allow customizing the installation path, then append instructions on how to do that, too.

## Upgrading

Prior to choco version 0.9.10, there is no dedicated automation script for upgrade scenarios. Instead, your chocolateyInstall.ps1 script should support installing/upgrading on top of any previous versions of your package.

More recent versions of choco (0.9.10+) give you the option of supplying a chocolateyBeforeModify.ps1 script.
If applicable, the version of this script from the currently installed package will be run before subsequent
chocolateyInstall or chocolateyUninstall scripts.

## Uninstalling

Uninstalling is handled by a chocolateyUninstall.ps1 script, which should be in your package's tools directory, next to chocolateyInstall.ps1. All the usual helper reference are available. If your package doesn't uninstall cleanly, people will get grumpy because they'll have to manually clean up after you. Be a good human being and write an uninstaller.

## Dependency Chaining

You can make packages that depend on other packages just by adding those dependencies to the nuspec. Take a look at ferventcoder.chocolatey.utilities nuspec.

## Avoid folders named "content"

Do not use a folder named "content" in your package. NuGet attaches a special meaning to this folder and will not allow you to have dependencies on packages that have content folders without also having a content folder. It's turtles all the way down until we or NuGet removes this limitation.

## Naming your package

The title of your package (\<title\> tag in the nuspec) should be the same as the name of the application. Follow the official spelling, use upper and lower case and don't forget the spaces. Examples of correct package titles are: Google  Chrome, CCleaner, PuTTY and FileZilla. The title will appear on the left side in the package list of the Chocolatey gallery, followed by the version.

There are some guidelines in terms of the package id (\<id\> tag in the nuspec):

* Use only lowercase letters, even if you used uppercase letters in the package title. (This is considered a guideline because it is correctable in other ways). Once a package is submitted (even prior moderation), the Gallery will always show the id with the casing of the first package version. In addition, changing the casing of the package id may have negative side effects on dependencies (note: this last statement needs verified).
* If the original application name consists of compound words without spaces (CamelCase), just as MKVToolNix, TightVNC and VirtualBox, the package id's are simply the same (but lowercase of course): mkvtoolnix, tightvnc, and virtualbox.
* If the name of the application contains multiple words separated by spaces, such as MusicBrainz  Picard or Adobe Reader, replace the spaces with the hyphen-minus character "-" (U+002D) or just omit them. Don't use dots. They should be used only if the original application name contains dots (e.  g. Paint.NET). Hence the correct id's of the previously mentioned applications can be musicbrainz-picard or adobereader. It is highly suggested to use the hyphen method when there are long package names, because that increases readability.
* For sub-packages, use the hyphen-minus character "-" (U+002D) as separator, not a dot. Sub-packages are intended for separate packages that include extensions, modules or additional features/files for other applications. Therefore keepass-langfiles is a proper package id, because it adds the language files for the main application which in this case is KeePass. Another example is libreoffice-help for the help pack for LibreOffice, the open source office suite.
* If you want to package an open source application, look on the repositories of some popular Linux distributions, such as Debian, Ubuntu and Arch Linux if they already have a package of it. If that is the case, use the same package id. This will make it easier for Linux and Windows users, because then they don't have to remember and use different package names.

These guidelines are already commonly applied on packages for all major Linux distributions, because they lead to a more consistent look of software repositories, easier to remember package id's and less considerations about the naming for package creators.

Note that a lot of packages in the Chocolatey Gallery don't follow these guidelines. The simple reason is that the affected packages were created before the introduction of these guidelines.

If you are going to offer a package that has both an installer and an archive (zip or executable only) version of the application, create three packages   - see Portable vs Installable and Install, Portable, and Meta/Virtual Packages

## Package description and release notes

The \<description\> of the package should contain a short text or at least a few words about the software for which the package is made. Here are a few things that should be respected:

    The description should always be written in English, even if the packaged software does not provide an UI in English. You can also include the software's description in its original language, but insert it after the English description.
    The description should not just contain a repetition of the package name.
    It should not just consist of a link where more information can be found. For that purpose there's already <projectUrl>.
    The contents of <description> and also <releaseNotes> are parsed as Markdown, so don't insert line breaks in the middle of sentences. Remember to add empty lines to separate paragraphs and add an empty line before a list.

## Versioning Recommendations

Versioning can be both simple and complicated. The best recommendation is to use the same versioning that the installable/portable application uses. With Chocolatey you get four version segments. If the application only uses 1, 2 or 3 version segments, follow suit.

If the 4th segment is used, some folks like to drop the segment altogether and use that as only the package fix notation using one of the notations in the next section. There is no recommendations at this time.

### Package Fix Version Notation

Package fix version notation ONLY applies when you are making a fix to the package because the existing version of a package is incorrect in some way. So if the software is 1.1.0, in a normal scenario the package version should be 1.1.0. If you find that the 1.1.0 package has an issue and you need to fix the package but keep the same version of the software, that is where package fix version notation comes into play. You would end up with both a 1.1.0 package and a 1.1.0.YYYYMMDD version of the package.

NOTE: This doesn't apply to packages on the community feed (aka https://chocolatey.org/packages) that are still under review (not yet approved). Please read the instructions given in email for resubmitting the same version.

If you need to fix an approved package for some reason, you can use the fourth version element (aka segment) for a package fix notation. There are two recommended methods of package fix version notation:

    Date (Year/Month/Day) - Some folks use year month day package fix notation (yyyyMMdd as in 20120627 seen as 1.2.0.20120627)
    Sequential - Not recommended - Some folks use sequential numbering (0, then 1, etc as in 0 for no fix, 1 for first fix and so on seen as 1.2.0.0 and 1.2.0.1).

Date Package Fix Version Notation is recommended because one can ascertain what it is immediately upon seeing it, where sequential is not obvious on sight whether it is part of the software versioning or something special with the package.

Package fix version notation is only acceptable in the fourth segment. Do not use any of the other segments for package fix notation. If an application only uses 1 or 2 version segments, add zeros into the other segments until you get to the 4th segment (i.e.  1.0.0.20120627).

When the fourth segment is already used, it is recommended to add two zeroes (00) to the end of the version. Then when you need to fix, you just increment that number. So if the package was ruby and the version was 2.0.0-p353, the package is 2.0.0.35300 (adding the two zeroes at the end). Then a fix would be 2.0.0.35301 and so on. WARNING: If you decide to add a secondary set of numbers to the fourth segment, you MUST ALWAYS include that secondary set of numbers while the other three version segments are the same. The reason - if you fix .1 to .100, then release .2, .100 is greater than .2 because versioning doesn't look at the ".", only the number in the element. So 100 is greater than 2.

## Internationalization and localization of packages

For Chocolatey, internationalization and localization of packages is very important, because it has users from all over the world. Many applications support multiple languages, but they use several different methods to achieve that. Therefore, there is no standard how internationalization/localization has to be integrated into packages. However, here are a few examples of packages that use various techniques. You can use them as inspiration for new packages:

    The ideal situation is when an application determines the user's system language and automatically installs with that language. Then you don't have to take any action relating to localization, because the application already handles that. Examples of such applications are VLC Media Player and LibreOffice.
    When an application provides different installers for different languages, you should determine the system language and download the appropriate installer. The package for Mozilla Firefox (source code) uses this method.
    Sometimes an application installer or executable has already integrated all supported languages, but doesn't automatically select the system language during a silent install. Often you can pass an additional install parameter to select the desired language. This is used for example in the CCleaner package (source code).
    Some application use separate language files which must be downloaded separately and put somewhere in the application directory. It is best when you create a separate package for the language files. If your package id is packageid, then call it packageid-langfiles. The language files package for KeePass is an example how this can be achieved.

## Package icon guidelines

If there is an icon which is suitable for your package, you can specify it in the \<iconUrl\> tag in the nuspec. But there are a few things you should consider:

* Avoid hotlinking icons from sites where you don't have control over the file. If you have a packages repository (recommended), use your own copy of the icon and put it there.
* For the icon URLs it is recommended to use https://cdn.rawgit.com/ (production links). Rawgit is a CDN service that permits you to serve files hosted in a repository on GitHub. Keep in mind that cdn.rawgit.com caches files permanently. Therefore it's best to use a specific tag or commit URL, not a branch URL.
* Avoid using GitHub raw links (https://raw.githubusercontent.com/...). They are not intended to be used as CDN.
* Use the software's icon if one is available, not the logo. This blog post explains the difference between logos and icons: http://blog.designcrowd.com/article/353/differences-between-logos-and-icons. If the software of your package doesn't have an icon, but a logo with text and an image, you can extract the image with your favorite image editor and use that as package icon. An example is MySQL's dolphin.
* Use package icons with at least 128 pixels in width or height if available. However, avoid very high resolutions, because this would only unnecessarily increase the page load time. If there are only icons with less than 128 pixels available, choose the icon with the highest resolution you can find without upscaling it. Don't use low resolution favicons as package icons.
* Use icons with transparent background if available.
* Don't use distorted or blurry icons.
* The package list in the gallery shows the icons with a maximum of 48 pixels in width/height. Application logos with very detailed graphics that are barely visible at that dimension are not suitable as package icons.
* PNG is the preferred format for raster package icons. Avoid ICO, GIF and JPEG graphics.
* Good sources for package icons are the official desktop icons of the corresponding application you want to make a package of. The icons can be extracted from the app executables using tools like BeCyIconGrabber. Remember to take the icon with 128  px or more and save it as PNG file.

## How do I exclude executables from getting shims?

If you have executables in the package or brought into the package folder during PowerShell run and you want to exclude them you need to create an empty file named exactly like (case sensitive) the executable with .ignore suffixed on the end in the same directory where the executable is or will be.

Example: In the case of Bob.exe you would create a file named Bob.exe.ignore and that file would not get a redirect batch link. The Chocolatey package has an example of that. To further expand, bob.exe.ignore would not work because it doesn't have the correct casing.

Here's a great programmatic example:

	$files = get-childitem $installDir -include *.exe -recurse

	foreach ($file in $files) {
		#generate an ignore file
		New-Item "$file.ignore" -type file -force | Out-Null
	}

## How do I set up shims for applications that have a GUI?

If you don't want to see a hanging window when you open an application from the command line that was set up with Chocolatey, you want to create a file next to the executable that is named exactly the same (case sensitive) with .gui suffixed on the end.

Example: In the case of Bob.exe you would create a file named Bob.exe.gui and that file would be set up as a GUI application so the window will call it and then move on without waiting for it to finish. Again, bob.exe.gui would not work because it doesn't have the correct casing.

## Build Your Package

Open a command line in the directory where the nuspec is and type choco pack. That's it.


## Testing Your Package

NOTE: We strongly suggest the following should be performed in a VM and not on your machine.

NOTE: Testing your package can be done in the same way as the verifier - take a look at Chocolatey Verifier Testing.

To test the package you just built, open a command line shell and navigate to the directory where the *.nupkg file is located. Then type:

	choco install packageName -dv -s .

This will install the package right out of your source. As you find things you may need to fix, using --force (-f) will remove and reinstall the package from the updated *.nupkg. If you are specifically testing chocolateyBeforeModify.ps1, you need to be testing upgrade and uninstall scenarios. You need to install a version of the package with this file first as before modify is like uninstall, it runs from the installed package, not the package you are installing (like chocolateyInstall.ps1 does).

NOTE: Using Force --force (-f) should only be done in subsequent testing where you are reinstalling the same package that you've changed and should NOT be used in regular use scenarios. It should definitely not be in scripts.

NOTE: If you are using a Semver dash in your package version (such as 1.0.0-beta), you will need to use the -pre switch or else you will get Unable to find package errors from choco install. You can also specify -version 1.0.0-beta to try to install that exact version.

. points to the current directory. You can specify multiple directories separated by a semicolon;

When your nuspec specifies dependencies that are not in your source, you should add their paths to the source directory. E.g. in the case of Chocolatey itself:

    <dependencies>
      <dependency id="chocolatey" version="0.9.8.20" />
    </dependencies>

You'll need to append the API path like so:
-source "'.;https://chocolatey.org/api/v2/'" (note the double quotes bookending the apostrophes here, use %cd% in cmd.exe or $pwd in Powershell.exe if . doesn't resolve). See passing options with quotes. Note: If you need to do this, please ensure you run choco pack first. This method of passing a source won't work calling a nuspec or nupkg directly as it will override the source passed to the local folder.

You can also use the -debug switch on choco install to provide more information.

NOTE: Do not call install with .nupkg - pointing to a file explicitly overrides source. You must call your install with the package name, not the nupkg file and location. You've already specified for choco to look in a local source with -s "'.;https://chocolatey.org/api/v2/'". Call choco install dude -s "'.;https://chocolatey.org/api/v2/'", not choco install .\dude.nupkg -s "'.;https://chocolatey.org/api/v2/'".

### Alternative testing strategy

You can also type choco install -fdv path/to/nuspec and choco will build the nupkg and attempt to install it.

NOTE: This is not recommended if you are passing install arguments or package parameters due to some weirdness, and definitely does not work with passed sources as it need to override that with the local folder once it builds the package. Most likely you will want to stick with the recommended strategy.

### Push Your Package

To push your package after you have built and tested it, you type choco push packageName.nupkg -s sourceLocation where packageName.nupkg is the name of the nupkg that was built with a version number as part of the package name and sourceLocation is the location of the source you want to push to (e.g. -s https://chocolatey.org/ for chocolatey's community feed). You must have an api key for https://chocolatey.org/ set. Take a look at choco push

You can also log into chocolatey.org and upload your package from there (not recommended for packages over 2MB).
