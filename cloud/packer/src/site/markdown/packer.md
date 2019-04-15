# Packer

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->
## Introduction 




## Quick start & demo

From : 

 * http://www.packer.io/
 
 
## Quick introduction to packer

Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image.
A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. 

## Packer in action 

### Building image 


### First example.json 

```

{
  "variables": {
    "aws_access_key": "",
    "aws_secret_key": ""
  },
  "builders": [{
    "type": "amazon-ebs",
    "access_key": "{{user `aws_access_key`}}",
    "secret_key": "{{user `aws_secret_key`}}",
	#ensure you have permission in the good region
    "region": "us-east-1",
    "source_ami_filter": {
      "filters": {
        "virtualization-type": "hvm",
        "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
        "root-device-type": "ebs"
      },
      "owners": ["099720109477"],
      "most_recent": true
    },
    "instance_type": "t2.micro",
    "ssh_username": "ubuntu",
    "ami_name": "packer-example {{timestamp}}"
  }]
}
```

### Validate your first example


```
 #warning: there a packer command in /usr/sbin, so be sure to use the good one :)
 packer validate example.json
 ...
 Template validated successfully.
```

 ### Build your first image
 
```
packer build \
    -var 'aws_access_key=YOUR ACCESS KEY' \
    -var 'aws_secret_key=YOUR SECRET KEY' \
    example.json
```


### BUILDERS 

### Provisionners 

Provisioners use builtin and third-party software to install and configure the machine image after booting. Provisioners prepare the system for use, so common use cases for provisioners include:

    installing packages
    patching the kernel
    creating users
    downloading application code

Here's a little example : 

```
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
        "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
        "region":         "us-east-1"
    },
    "builders": [
        {
            "access_key": "{{user `aws_access_key`}}",
            "ami_name": "MYFunnyName-{{timestamp}}",
            "instance_type": "t2.micro",
            "region": "{{user `region`}}",
            "secret_key": "{{user `aws_secret_key`}}",
			"source_ami": "AMI-REF to use as starting point",

            #"source_ami_filter": {
            #  "filters": {
            #    "virtualization-type": "hvm",
            #    "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
            #    "root-device-type": "ebs"
            #  },
            #   "owners": ["XXXXXXXXXXXXXXXXXXXx"],
            #   "most_recent": true
            # },

			#If your provisioners need to access S3 data 
            "iam_instance_profile":"myBestProfile", 
            "ssh_username": "ubuntu",
            "type": "amazon-ebs",
            "tags": {
              "App": "aTAG",
              "Release": "AReleaseNumber",
              "BuildAt": "{{isotime \"02-Jan-06 03:04:05\"}}"
			},
            "ami_users": [
              "YYYYYYYYYYY",
              "ZZZZZZZZZZZ"
            ]
        }
    ],
    "provisioners": [
        {
            "type": "file",
            "source": "./welcome.txt",
            "destination": "/home/ubuntu/"
        },
        {
            "type": "shell",
            "inline":[
                "ls -al /home/ubuntu",
                "cat /home/ubuntu/welcome.txt"
            ]
        },
        {
            "type": "shell",
            "script": "./example.sh"
        }
    ]
}
```

### Architecture 

![Architecture overview](./images/architecture.png "Architecture Overview")




