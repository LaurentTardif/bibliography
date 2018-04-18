# Automtation of internalisation  

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


## Internalisation  

If an organization is serious about using Chocolatey to manage its packages, then it really should host its own package repository. Of course, you could use the public Chocolatey repository as your source for packages, but this is a bad idea for a couple of reasons. First, you would have to trust the maintainers of those packages completely, and second, you would need to reach out to the internet to install the packages.

Fortunately, Chocolatey for Business (C4B) allows you to recompile (or internalize) public packages easily, which then enables you to push these packages to your own repository. Internalizing refers to taking remote installers that public packages may call, downloading them, and then embedding them in your NuGet package or another location such as a CIFS share.

When I created my own repository, one of the first tasks I looked at automating was internalizing public packages—especially since I manage hundreds of packages. Doing this has saved me countless hours creating my own packages. The initial internalizing of packages from Chocolatey is actually quite simple, as I will show below, but what happens when a new package version is released? This is where PowerShell comes in handy.

In this article, I will demonstrate how to check for updated Chocolatey packages, test installation, and internalize them automatically. Keep in mind this is done with the Chocolatey Business version, which significantly eases the process.
How Chocolatey internalizing works

Internalizing essentially means Chocolatey will take any remote resources that a public package uses (such as installers), and it will download them locally to create another NuGet package you can use. By default, the resources are relocated into the tools directory of the NuGet package. Here I will show a simple example of internalizing the ownCloud client. As you can see in the output, the recompiled package is available in the C:\Example directory:

	PS C:\Example> choco download owncloud-client --internalize --ignore-dependencies
	Chocolatey v0.10.8 Business
	Downloading existing package(s) to C:\Example\download
	Progress: Downloading owncloud-client 2.3.3.8250... 100%

	owncloud-client v2.3.3.8250
	Found Install-ChocolateyPackage. Inspecting values for remote resources.
	Updating chocolateyInstall.ps1 with local resources.
	Recompiling package.

	Recompiled package files available at 'C:\Example\download\owncloud-client'
	Recompiled nupkg available in 'C:\Example'


	
	PS C:\Example> choco download owncloud-client --internalize --ignore-dependencies
	Chocolatey v0.10.8 Business
	Downloading existing package(s) to C:\Example\download
	Progress: Downloading owncloud-client 2.3.3.8250... 100%
 
	owncloud-client v2.3.3.8250
	Found Install-ChocolateyPackage. Inspecting values for remote resources.
	Updating chocolateyInstall.ps1 with local resources.
	Recompiling package.
 
	Recompiled package files available at 'C:\Example\download\owncloud-client'
	Recompiled nupkg available in 'C:\Example'

Now, if we compare the ChocolateyInstall.ps1 file from the public repository to the internalized package we just created, we can see the difference. The internalized package points to the local installer instead of the ownCloud URL, but the checksum is the same:

	Public package:
	$ErrorActionPreference = 'Stop'

	$packageArgs = @{
		packageName    = 'owncloud-client'
		fileType       = 'exe'
		softwareName   = 'ownCloud'

		checksum       = '6d347803c779377cf8e66d5235f5a9390b4e328b8d2c7236d5d47f029622c90b'
		checksumType   = 'sha256'
		url            = 'https://download.owncloud.com/desktop/stable/ownCloud-2.3.3.8250-setup.exe'

		silentArgs     = '/S'
		validExitCodes = @(0)
	}
	
	$ErrorActionPreference = 'Stop'
 
	$packageArgs = @{
		packageName    = 'owncloud-client'
		fileType       = 'exe'
		softwareName   = 'ownCloud'
 
		checksum       = '6d347803c779377cf8e66d5235f5a9390b4e328b8d2c7236d5d47f029622c90b'
		checksumType   = 'sha256'
		url            = 'https://download.owncloud.com/desktop/stable/ownCloud-2.3.3.8250-setup.exe'
 
		silentArgs     = '/S'
		validExitCodes = @(0)
	}	

Internalized package:

	$ErrorActionPreference = 'Stop'

	$packageArgs = @{
		packageName    = 'owncloud-client'
		fileType       = 'exe'
		softwareName   = 'ownCloud'

		checksum       = '6d347803c779377cf8e66d5235f5a9390b4e328b8d2c7236d5d47f029622c90b'
		checksumType   = 'sha256'
		url            = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)\files\ownCloud-2.3.3.8250-setup.exe"

		silentArgs     = '/S'
		validExitCodes = @(0)
	}
		
	$ErrorActionPreference = 'Stop'
 
	$packageArgs = @{
		packageName    = 'owncloud-client'
		fileType       = 'exe'
		softwareName   = 'ownCloud'
		
		checksum       = '6d347803c779377cf8e66d5235f5a9390b4e328b8d2c7236d5d47f029622c90b'
		checksumType   = 'sha256'
		url            = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)\files\ownCloud-2.3.3.8250-setup.exe"
	
		silentArgs     = '/S'
		validExitCodes = @(0)
	}

Internalizing multiple Chocolatey packages

Let’s say I have a group of packages I want to internalize. In this example, they’re Google Chrome, Git, LastPass, and Notepad++. A recent release of the Chocolatey Business version allows you to pass multiple packages with choco download:
	
	choco download googlechrome git lastpass notepadplusplus --internalize

	
	choco download googlechrome git lastpass notepadplusplus --internalize

Cool! Now we can just push the packages to our hosted repository:
choco push googlechrome.nupkg --source --api-key='yourapikey'

	choco push googlechrome.nupkg --source --api-key='yourapikey'

Automating the checking and internalizing for new package versions

Now that we have a set of packages internalized, I want to take this a step further and automate internalizing any new version of a package if it is released on the Chocolatey repository, creating a very basic pipeline. There are a few ways to do this, but what I prefer is to use a virtual machine (VM) that will act as a testing machine for checking, installing, and internalizing new public packages. Set at a certain time interval, such as every 4 hours, this VM will run my script, which includes the command choco outdated. This command will see if any Chocolatey packages the machine installed are outdated based on the Chocolatey repository. If the VM finds certain packages have updated versions available, it will first internalize them, then install them to itself, and finally push them to our hosted NuGet server. Pretty cool!

To combine all of this, I created a PowerShell function Add-ChocoInternalizedPackage, which orchestrates this process. Here I will step through some of the code to show you what is happening. Please keep in mind this is a work in progress and currently does not work perfectly.

The parameter $APIKeyPath indicates that you have an encrypted file that has the API key of your internal Chocolatey repository. The first thing the function does is grab this key to use later on in choco push:

	#Convert API Key Path for use in choco push
	$PushAPIKey = Get-Content $APIKeyPath | ConvertTo-SecureString
	$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($PushAPIKey )
	$ApiKey = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
	
	#Convert API Key Path for use in choco push
	$PushAPIKey = Get-Content $APIKeyPath | ConvertTo-SecureString
	$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($PushAPIKey )
	$ApiKey = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

Here, we run choco outdated -r to get any out-of-date packages on our local machine. If any are found, we create a custom PowerShell object. The output of choco outdated needs to be parsed and formatted a bit since “|” is the delimiter, so we use the split method and place them into their own properties. Now, $NewPackages can be processed by the rest of the script.

    #Get outdated packages
    $OutdatedPackages = (choco outdated -r)
    #If no updated packages are available then exit
    if (!$OutdatedPackages)
    {
        Write-Warning -Message 'No new packages available. Exiting'
        Exit
    }
    else
    {
        $NewPackages = foreach ($NewPackage in $OutdatedPackages) 
        {
            [PSCustomObject]@{
                Name = $NewPackage.Split('|')[0]
                CurrentVersion = $NewPackage.Split('|')[1]
                NewVersion = $NewPackage.Split('|')[2]
                Pinned = $NewPackage.Split('|')[3]
            }
        }
	
    #Get outdated packages
    $OutdatedPackages = (choco outdated -r)
    #If no updated packages are available then exit
    if (!$OutdatedPackages)
    {
        Write-Warning -Message 'No new packages available. Exiting'
        Exit
    }
    else
    {
        $NewPackages = foreach ($NewPackage in $OutdatedPackages) 
        {
            [PSCustomObject]@{
                Name = $NewPackage.Split('|')[0]
                CurrentVersion = $NewPackage.Split('|')[1]
                NewVersion = $NewPackage.Split('|')[2]
                Pinned = $NewPackage.Split('|')[3]
            }
        }

We loop through the packages that need updating, but skip any packages with the name *.install that is also contained in the $NewPackages.Name object. The reason for this is that many packages such as “git” are actually virtual packages that just call *.install (example git.install). When using choco download, all packages including virtual and dependency packages are downloaded to be internalized, so “git.install” will still be internalized when the “git” package is processed. Skipping *.install packages will save us from redundancy. The $DownloadTime variable is used to filter all packages downloaded after this time, as you will see later.

	foreach ($InstallPackage in $NewPackages)
	{
     #Check to see if need to skip *.install packages due to redundancy in virtual package
     if ($InstallPackage.Name -like "*.install" -and $NewPackages.Name -contains ($InstallPackage.Name.Replace('.install','')))
     {
          Write-Warning ($InstallPackage.Name + ' skipping due to existing virtual package')  
          Continue  
     }
     #Get time to use with choco push
     $DownloadTime = Get-Date

	
	foreach ($InstallPackage in $NewPackages)
	{
     #Check to see if need to skip *.install packages due to redundancy in virtual package
     if ($InstallPackage.Name -like "*.install" -and $NewPackages.Name -contains ($InstallPackage.Name.Replace('.install','')))
     {
          Write-Warning ($InstallPackage.Name + ' skipping due to existing virtual package')  
          Continue  
     }
     #Get time to use with choco push
     $DownloadTime = Get-Date

This block of code attempts to internalize the package locally and save it to whatever path is in the $WorkingDirectory variable. Notice we check the $LASTEXITCODE variable to ensure that there were no errors in the choco download command. If $LASTEXITCODE is anything but zero (success), we add it to the $Failure array and move to the next package.
 
	choco download $InstallPackage.Name --internalize --no-progress
	if ($LASTEXITCODE -ne 0)
	{
      Write-Warning ($InstallPackage.Name + ' internalize failed')
      $Failure.Add($InstallPackage.Name) | Out-Null
      Continue
	}
	
	choco download $InstallPackage.Name --internalize --no-progress
	if ($LASTEXITCODE -ne 0)
	{
      Write-Warning ($InstallPackage.Name + ' internalize failed')
      $Failure.Add($InstallPackage.Name) | Out-Null
      Continue
	}

If internalization is successful, it attempts to install that package from the local Chocolatey package. And if the package installs correctly, it pushes all internalized packages created after the $DownloadTime variable (which again includes dependency and virtual packages) to your hosted repository with choco push, using the value of $RepositoryURL for your feed.

    choco upgrade $InstallPackage.Name --source=$WorkingDirectory --no-progress -y
    #If failure detected in output continue to next package
    if ($LASTEXITCODE -ne 0)
            {
                Write-Warning ($InstallPackage.Name + ' install failed')
                $Failure.Add($InstallPackage.Name) | Out-Null
                Continue
            }
            #If no failure detected than push to hosted repository
            else 
            {
                #Get package and all dependency package paths for push
                $DownloadedPackages = Get-ChildItem -Path $WorkingDirectory | Where-Object {$_.Extension -eq '.nupkg' -AND $_.LastWriteTime -gt $DownloadTime} | Select-Object -ExpandProperty FullName
                foreach ($DownloadedPackage in $DownloadedPackages) 
                {
                    Write-Output ("Pushing " + $DownloadedPackage)
                    choco push $DownloadedPackage --source=$RepositoryURL -k=$ApiKey
                    if ($LASTEXITCODE -ne 0)
                    {
                        Write-Warning -Message "$DownloadedPackage failed push"
                        $Failure.Add((Split-Path -Path $DownloadedPackage -Leaf)) | Out-Null
                    }
                    else 
                    {
                       Write-Output "$DownloadedPackage successfully pushed"
                       $Success.Add((Split-Path -Path $DownloadedPackage -Leaf)) | Out-Null
                    }
                }
            }
	
            choco upgrade $InstallPackage.Name --source=$WorkingDirectory --no-progress -y
            #If failure detected in output continue to next package
            if ($LASTEXITCODE -ne 0)
            {
                Write-Warning ($InstallPackage.Name + ' install failed')
                $Failure.Add($InstallPackage.Name) | Out-Null
                Continue
            }
            #If no failure detected than push to hosted repository
            else 
            {
                #Get package and all dependency package paths for push
                $DownloadedPackages = Get-ChildItem -Path $WorkingDirectory | Where-Object {$_.Extension -eq '.nupkg' -AND $_.LastWriteTime -gt $DownloadTime} | Select-Object -ExpandProperty FullName
                foreach ($DownloadedPackage in $DownloadedPackages) 
                {
                    Write-Output ("Pushing " + $DownloadedPackage)
                    choco push $DownloadedPackage --source=$RepositoryURL -k=$ApiKey
                    if ($LASTEXITCODE -ne 0)
                    {
                        Write-Warning -Message "$DownloadedPackage failed push"
                        $Failure.Add((Split-Path -Path $DownloadedPackage -Leaf)) | Out-Null
                    }
                    else 
                    {
                       Write-Output "$DownloadedPackage successfully pushed"
                       $Success.Add((Split-Path -Path $DownloadedPackage -Leaf)) | Out-Null
                    }
                }
            }

In terms of adding packages to the $Success and $Failure arrays, I chose to use Split-Path and remove the $WorkingDirectory from the value since that is not necessary for the user to see.

Finally, we write the contents of the successful and failed packages to the console. Optionally, you could put this into Send-MailMessage and email the results.

    #Write successes and failures to console
    Write-Output "Successful packages:"
    $Success
    Write-Output "Failed packages:"
    $Failure

	#Write successes and failures to console
    Write-Output "Successful packages:"
    $Success
    Write-Output "Failed packages:"
    $Failure
	
	