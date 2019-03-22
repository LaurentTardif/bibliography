# Registry 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Bibliography

 * https://rancher.com/comparing-four-hosted-docker-registries/
 * https://rancher.com/container-registries-might-missed/ 
 * https://www.slant.co/topics/2436/~best-docker-image-private-registries
 * https://blog.codeship.com/overview-of-docker-registries/

## Registry (Docker and non docker)

### Docker Concepts

* **Index**: An index tracks name-spaces and docker repositories. For example, Docker Hub is a centralized index for public repositories.
* **Registry**: A docker registry is responsible for storing images and repository graphs. A registry typically hosts multiple docker repositories.
* **Repository**: A docker repository is a logical collection of tags for an individual docker image, and is quite similar to a git repository. Like the git counterpart, a docker repository is identified by a URI and can either be public or private. The URI looks like: ** {registry_address}/{name-space}/{repository_name}:{tag}**

**For example** : 	quay.io/techtraits/image_service

There are, however, a couple of exceptions to the above rule. First, the registry address can be omitted for repositories hosted with Docker Hub, e.g., techtraits/image_service. This works because the docker client defaults to Docker Hub if the registry address is not specified. Second, Docker Hub has a concept of official verified repositories and for such repositories, the URI is just the repository name. For example, the URI for the official Ubuntu repository is simply \“ubuntu\“.

### Working with docker registries

To use a docker registry with the command line tool, you must first login. Open up a terminal and type:

 docker login index.docker.io

Enter your username, password and email address for your Docker Hub account to login. Upon successful login, Docker client will add the credentials to your docker config (~/.dockercfg) and each subsequent request for a private image hosted by that registry will be authenticated using these credentials.

Similar to the command line tool, Rancher provides an easy way to connect with your registries for managing your docker deployments. To use a registry with Rancher, navigate to settings and select registries. Next, put in your registry credentials and click create to add a new registry. You can follow the same process for adding multiple registries.


## 1- Docker Registry

**URL**: https://docs.docker.com/registry/

**Top Con**

 * Only supports docker images
 * Setting it up may require some work
 * Setting up the new Docker Registry 2.0 requires some more setup than usual. Especially dependencies need to be installed beforehand. 
 * **proxy** : Can mirror Official docker registries and be used as a proxy
 
**Top Pro**

 * Simple to set up , https://docs.docker.com/registry/#basic-commands
 * Maintains several storage drivers to allow for different models of image retention, Currently includes in-memory, local filesystem, Azure, and S3.

 
## Coreos entreprise registry  
 
**Top Pro**

 * Web based UI,CoreOS Enterprise Registry also features a web based UI for managing containers.
 * Powerful auditing logs, CoreOS Enterprise Registry logs every Docker repo access. The name of the item, the action performed and the authorization of the user who made the action are all stored. Logs can also be exported in JSON format.
 * Both self-hosted and hosted versions exist, CoreOS has given teams the option to either host CoreOS Enterprise Registry on their internal servers, or use the hosted version maintained by CoreOS themselves.
 * Has LDAP support, The Lightweight Directory Access Protocol is an application protocol for accessing and maintaining distributed directory information services over an Internet Protocol (IP) network.
 * Based on Quay.io, Base on the hosted private docker made by Quay.io which was actually acquired by CoreOS in August. Basically Enterprise Registry is Quay.io running behind a firewall.
 * Powerful team based user management, CoreOS features a powerful team based user management system, a lot like GitHub enterprise's which allows teams to be created and each user has their own level of permission. Credentials and passwords for each team member are stored safely within Docker containers. All these features allow teams to work safely and securely behind a firewall.
 * **GEO replication**
 * **security scan** with **clair** 
 
**Top Con**
 
 * Pricing may change in the future, For now, the price for Enterprise Registry is set to starting at $10/month which is rather cheap compared to the service they are offering. But CoreOS has announced that they will notify their users in advance if the price model is set to change which suggests that they may be planning to change it or at least that they are taking a trial-and-error approach to their pricing and general business plan.
 * can be used as mirror ? 
 
 
## JFROG Artifactory

**Top Pro**

 * Universal binary repository manager for Docker and any other package type, One centralized repository for any package type.
 * Can run as a Docker container, A very simple setup for trial and production deployment using Docker.
 * Run it locally or get it as a service, You can install Artifactory locally in your data center or get it as a service from JFrog on one of several cloud providers (currently AWS, GCP and Azure).
 * Integration with JFrog Xray, JFrog Xray enables scanning of Docker images for known vulnerabilities, license compliance issues, providing a full component graph and analysis tool.
 * Support high availability set up

**Top Con**

 * Docker registry support is not available on their OSS version

Artifactory is an open source repository manager for binary artifacts. Unlike some of the other registry services in our list, Artifactory focuses on artifact management, high availability and security; for instance, it leaves the builds to your build servers. Similar to Quay and Docker Hub, Artifactory comes in two flavours: an on-premise solution and Artifactory cloud. However, unlike Docker Hub and Quay, Artifactory is relatively expensive if you’re only looking to use it as a docker registry. It starts out at \$98/month for 8GB of storage and 400GB network bandwidth per month with per GB charges for additional usage.

Repository or a registry: For a new user, it can be a bit confusing to get started. First, you’ll notice that a docker registry is just a repository in Artifactory. Then we have to select between a local repository or a remote repository. The distinction between the two is that a local repository is hosted by Artifactory whereas a remote repository is an external repository for which Artifactory acts as a proxy. I think remote repositories can be quite useful for ensuring availability, where Artifactory provides a unified interface to all your artifacts while also acting as a cache layer for remote artifacts.

To create a docker registry, select “repositories” in the Admin view and create a new local repository. Make sure you select docker as the package type for the new repository: **artifactory\_docker\_repo**

The address for your docker registry will look like: **{account-name}-docker-{repo-name}.artifactoryonline.com**

For example, I created a “docker-local” repository in my techtraits account and got the following registry address::**techtraits-docker-docker-local.artifactoryonline.com**

Except for an awkwardly long address, your docker commands would work the same with Artifactory because it fully supports the docker registry API. For instance, I used the following commands to login, tag, push and pull images with the docker client:

	docker login techtraits-docker-docker-local.artifactoryonline.com
	docker tag ubuntu techtraits-docker-docker-local.artifactoryonline.com/ubuntu
	docker {push|pull} techtraits-docker-docker-local.artifactoryonline.com/ubuntu

Rancher doesn’t have out of the box support for Artifactory but it can be added using the “custom” registry option. Just set your email address, credentials and the repository address and you are all done. You should now be able to use Rancher to run containers from your private images hosted in an Artifactory registry.

	rancher\_artifactory\_registry

**Advanced security**: Artifactory supports a suite of authentication options including LDAP, SAML and Atlassian’s Crowd. For fine-grained access control, there is support for user and group level permissions. You can also specify which group(s) a new user is added to by default. The permissions can further be controlled on a per-repository level for granular access control. For instance, in the screenshot below I have three groups, each with different access to my private repository:

	artifactory\_image\_view

**Summary:**

*Workflow:*

	+ all-in-one artifact management model
	– learning curve for new users
	– limited insight into registry usage

*Authentication and authorization:*

 + comprehensive authentication capabilities
 + fine-grained access control

*Availability and performance:*

 + remote repositories for high availability

*Pricing:*

 – steep pricing 
 
 
 
## Sonatype Nexus

**Pro**

 * Easy to setup, It just takes seconds to spin-up the docker image. It supports persistent volumes and ldap.
 * Is very easy to setup, and supports multiple types of repositories including , npm and maven.
 * OSS version provides Docker registry and proxying support

**Top Con**

 * Requires POSIX file system for volumes, This will not work on Windows based CIFS volumes, only POSIX compatible volume stores.
 * Their docker image lacks customization, There isn't much customization that can be don on the docker image that they have. Specifically presetting some environment settings such as HTTP proxy authentication.

 
## Gitlab Container Registry 

## JFrog Bintray

**Top Pro**

 * Full control and security, Exercise fine-grained access control over who can view, upload to or download from your private repositories. Maintain any degree of control through a variety of means, such as IP and geographical restrictions, EULA acceptance and more. Automatically provision your organization users via API, or have them silently sign in with SAML authentication to your existing identity provider.
 * Statistics and dashboards, Great statistics on downloads of your Docker images according to tags, geo-location and more..
 * Universal solution, One distribution platform that supports all technologies. JFrog Bintray natively supports all major package formats, which allows you to work seamlessly with industry standard development, build and deployment tools. With support for massive scalability and worldwide coverage, this gives you the best native repository distribution available.
 * Open for automation, JFrog Bintray easily integrates with your existing DevOps ecosystem, such as your continuous integration pipeline and your internal repositories. A rich REST API allows you to control every aspect of your software distribution, manage who has access to your content, collect logs and analytics, and much more - all with the full automation expected of a modern software distribution platform. 

## Google registry

**Top Pro **

 * Charged only for the storage
 * Reliable and consistent uptime, Since it's hosted on Google Cloud Platform then it's very reliable in both the uptime and the security. 


 Google is one of the key innovators in the containerization space and so, not surprisingly, it also offers a hosted registry service. If you’re not already using Google cloud services, you can get started by creating a new Google API project (with billing enabled) in Google’s developer console.

Unlike some of the other services, there are some caveats when working with the registry service. Therefore, I’m going to trace the steps for working with it.
First, you’d need to install Google cloud client SDK. Make sure you are able to run docker without sudo if you are installing gcloud as a non-root user. Next, we need to authenticate with Google using the client sdk:

	gcloud auth login

To push and pull images, we can invoke the docker command through gcloud:

	gcloud preview docker push|pull gcr.io/<PROJECT_ID>/ubuntu

Note that your registry address is fixed for each project and looks like:

	gcr.io/<PROJECT_ID>

Google command line tool will add multiple entries for various regions in your docker config when gcloud’s docker commands are invoked for the first time. A config entry looks like:

	"https://gcr.io": {
		"email": "not@val.id",
		"auth": "<AUTH-TOKEN>"
	}

Note that Google doesn’t store a valid email address in the config because it relies on temporary tokens for authorization. This approach enhances security; however, it does so at the expense of compatibility with the docker client and registry APIs. As it turns out, this additional dependency on Google client SDK and incompatibility with docker is one of the main reasons why Rancher doesn’t support it.

	google\_registry

Behind the scenes, your images are simply stored in a Google Cloud Storage (GCS) bucket and you are only charged for GCS storage and network egress. This makes Google registry a relatively inexpensive service. High availability and fast access are other out-of-the-box benefits that you get from a GCS-based image store.

**Workflow:** 

  While I like the idea of using GCS for storing images, the overall support for a docker based workflow is quite minimal. For instance, the registry service doesn’t support webhooks or notifications for when repositories get updated. It also doesn’t provide any insight into your registry usage. This is one area where I think Google’s docker registry service needs quite a bit of work.

**Security and access control:**

  Google takes a number of steps to ensure secure access to your docker images. First, it uses short-lived tokens when talking to the registry service. Second, all images are encrypted before they are written to GCS. Third, access to individual images and repositories can be restricted through GCS support for object and bucket level ACLs. Lastly, you can sync up your organization’s user directories with the Google project using Google Apps Directory Service.

**Summary:**
**Workflow:**

	– incompatibility with docker client
	– minimal support for integration with build and deployment pipelines
	– limited visibility into registry usage

**Authentication and authorization:**

	+ fine-grained access control with GCS ACLs
	+ enhanced security through short-lived temporary auth tokens
	+ support for LDAP sync

**Performance and availability:**

	+ uses the highly available Google cloud storage layer

**Pricing:**

	+ low cost 
 
 
## Quay.io

**Top Pro** 

 * Acquired company, CoreOS bought Quay and redirected their energies to the CoreOS Enterprise Registry, so this product might not be here for the duration.
 * Web based UI, Quay.io has a beutiful and easy to understand web based UI.
 * Most prominent alternative to Docker Hub, Quay was started by two Google engineers to serve as an alternative to Docker Hub. It was later acquired by CoreOS. 

 
 Next, we’ll look at Quay.io by CoreOS. It’s a free service for public repositories and there are multiple pricing tiers to choose from, based on the number of private docker repositories you are looking to host. Like Docker Hub, there is no additional charge for network bandwidth and storage.
 Authentication and access control: In Quay we can create organizations and teams where each team can have its own permissions. There are three permission levels to chose from: read (view and pull only), write (view, read and write) and admin. Users added to a team inherit their team’s permissions by default. This allows for more control over access to individual repositories. As for authentication, in addition to email based login, Quay supports OAuth integrations but lacks support for SAML, LDAP and Active directory.

 Automated builds done right: Workflow is one area where Quay excels. Not only does it have an intuitive and streamlined interface, it does a number of small things right to seamlessly integrate with your software delivery pipeline. A few highlights:

    Support for custom git repositories for build triggers using public keys
    Regular expression support for mapping branches and docker tags. This is quite useful for integrating with your git workflow (e.g. git flow) to automatically generate new versions of docker images for each new version of your application.
    Robot accounts for build servers
    Rich notification options including email, HipChat and webhooks for repository events

	quay\_builds

Visualizations and more visualizations: Quay has some great visualizations for your docker images, which effectively capture the evolution of an image. Visualizations don’t end there though, as Quay also offers detailed audit trail, build logs (shown below) and usage visualizations for visibility into all aspects of your docker registry.

	quay\_build\_log

While working with Quay, I didn’t notice any performance deteriorations, which is a good sign. But since I haven’t used it for large deployments, I don’t have any further insights into how it would behave under load.

**Summary:**
**Workflow:**

	+ clean, intuitive and streamlined interface
	+ automated builds done right
	+ great visibility into your registry usage
	+ webhooks and rich event notifications

**Authentication and authorization:**

	+ ability to create organizations and teams
	+ fine-grained access control through permissions

	– limited OAuth-only authentication support

**Pricing:**

	+ relatively inexpensive with usage-independent pricing 
 
 
## Amazon EC2 container registry

**TOP CON** 

 * The access token expires after 12 hours, You have to build a more complex deployment script in order to compensate for the AWS token expiring after 12 hours.

**Top Pro**

	* Amazon ECR is integrated with Amazon EC2 Container Service (ECS), simplifying your development to production workflow.
	* Free tier, Amazon ECR, like other AWS tools has a free tier for beginners of 500MB-month storage for one year. 
 
You probably already know that Amazon offers a hosted container service called Amazon EC2 Container Service (ECS). But the registry that Amazon provides to complete ECS tends to receive less attention. That registry, called Amazon EC2 Container Registry (ECR), is a hosted Docker container registry. It integrates with ECS. Introduced in December 2015, it is a somewhat newer registry option than most of the better-known registries, explaining why some users may not be familiar with it. ECS is not the only container registry that is compatible with ECR. ECS supports external registries, too. However, the main advantage of ECR is that it is a fully hosted and managed registry, which simplifies deployment and management. ECR also is as scalable as the rest of the ECS infrastructure -- which means it is very, very scalable. Best Use Cases: If you are a heavy user of AWS services, or plan to be, and are starting to look for a place to host private images, then ECR makes perfect sense to use. It is also a good choice if you have a large registry deployment or expect your registry to expand significantly over time; in that case, you’ll benefit from the virtually unlimited scalability of ECR.

## FlawCheck Private Registry

FlawCheck Private Registry (which was recently acquired, along with the rest of FlawCheck’s business, by security vendor Tenable) is a security-focused registry option. It offers integrated vulnerability scanning and malware detection for container images. While there is no magic bullet for keeping your container images free of malicious code, or preventing the insertion of malicious images into your registry, FlawCheck’s scanning features can help mitigate the risks. Best Use Case: For security-conscious companies out there, this is a really great option. I foresee a lot of adoption for this registry in heavily regulated industries. 
 
## GITLAB registry
 
 which can run as a hosted or on-premises registry, is GitLab’s solution for hosting container images. It’s built into GitLab and completely compatible with the rest of GitLab’s tools, which means it can integrate directly into your GitLab delivery pipeline. That’s an advantage if your team is seeking to adopt a seamless, DevOps workflow with as few moving parts as possible. Best Use Case: Some developers will find it convenient to store their Docker images on the same platform as their source code. If you use GitLab for your source code, then you’ll likely find the GitLab Container Registry handy. Otherwise, however, GitLab Container Registry doesn’t offer any killer features unavailable from most other registries.
 
 
## Portus by SUSE

Portus is not technically a registry, but it provides a front-end that replaces the native UI for on-premises deployments of Docker Registry. Portus is designed to add value to Docker Registry by providing extra access control options. These include the ability to configure “Teams” or registry users, with different access levels established for each Team. (In many ways, this feature is similar to user groups on Unix-like systems.) Portus also supports registry namespaces, which make it possible to configure the types of modifications individual users, as well as teams of users, can make to different repositories on a granular basis. Also notable is that Portus provides a user-friendly Web interface for configuring registry settings and access controls. (A CLI configuration tool, portusctl, is available as well.) Best Use Case: If you like Docker Registry but need extra security controls, or have other reasons to use fine-grained access control, Portus is a strong solution.

## VMware Harbor Registry

You might not think of VMware as a major player in the Docker ecosystem, but the company certainly has its toes in the water. Harbor Registry is VMware’s answer for hosting Docker images. This registry is built on the foundation of Docker Distribution, but it adds security and identity-management features. It also supports multiple registries on a single host. Best Use Case: Because of Harbor’s focus on security and user management, this option offers some valuable registry features that enterprises might seek, which are not available from all other registries. It’s a good choice in the enterprise. It’s worth noting, too, that because Harbor runs as Docker containers, it is easy to install on any server that has a Docker environment -- and the developers even offer an offline installer, which could be handy in situations where security considerations or other factors mean that a connection to the public Internet is not available.

 
## Docker HUB
 
**Top Pro**
 
	* Shares user accounts with the dominant public registry, Docker Hub

**Top Con**

	* Gives no metadata about image tags beyond their name, No information about when the image was created, pushed, what Dockerfile it came from, what user(s) pushed it, etc.
	* Default to public makes it dangerous, Since by default your account will create new repositories publicly, you could fairly easily leak sensitive images with one bad push.
	* Poor user interface
 

Docker Hub is an obvious choice for hosted private repositories. To get started with Docker Hub, sign up here and create a new private repository. It is very well-documented and straightforward to work with, and so I’ll skip the details. Every user is entitled to one free private repository and you can select between multiple tiers based on the number of private repositories you need. I really like that the pricing is usage independent, i.e., you are not charged for the network bandwidth used for pulling (and pushing) images.

Collaboration and access control: If you are familiar with Github, you will feel right at home with Docker Hub. The collaboration model adopted by Docker Hub is very similar to Github where individual collaborators can be added for each repository. You can also create organizations in Docker Hub. There is also support for creating groups within organizations; however, there is no way of assigning permissions to groups. What this means is that all users added to an organization share the same level of access to a repository. The coarse-grained access control may be suitable for smaller teams, but not for larger teams looking for finer control for individual repositories. Docker Hub also doesn’t support any of LDAP, Active Directory and SAML, which means that you can’t rely on your organization’s internal user directory and groups for restricting access to your repositories.

Docker Hub and your software delivery pipeline: Docker Hub aims to be more than just a repository for hosting your docker images. It seamlessly integrates with Github and Bitbucket to automatically build new images from Dockerfile. When setting up an automated build, individual git branches and tags can be associated with docker tags, which is quite useful for building multiple versions of a docker image.

 docker\_hub\_build\_repo

For your automated builds in Docker Hub, you can link repositories to create build pipelines where an update to one repository triggers a build for the linked repository. In addition to linked repositories, Docker Hub lets you configure webhooks which are triggered upon successful updates to a repository. For instance, webhooks can be used to build an automated test pipeline for your docker images.

Although useful, automated build support by Docker Hub does have a few rough edges. First, the visibility into your registry is limited; for instance, there are no audit logs. Second, there is no way of converting an image repository into an automated build repository. This somewhat artificial dichotomy between repository types means that built images can’t be pushed to an automated build repository. Another limitation of Docker Hub is that it only supports Github and BitBucket. You are out of luck if your git repositories are hosted elsewhere, e.g., Atlassian’s Stash.

Overall, Docker Hub works well for managing private images and automated image builds; however, on a number of occasions I noticed performance degradation when transferring images. Unfortunately without SLAs, I would be a bit concerned about using Docker Hub as the backbone for large-scale production deployments.

**Summary:**

**Workflow:**

	+ seamless integration with Github and BitBucket
	+ familiar Github-esq collaboration model
	+ repository links and webhooks for build automation
	+ well-documented and easy to use

	– build repositories lack support for custom git repos
	– limited insight into registry usage

**Authentication and authorization:**

	+ ability to create organizations

	– lacks fine-grained access control
	– lacks support for external authentication providers , e.g. LDAP, SAML and OAuth

**Availability and performance:**

	– uneven performance

**Pricing:**

	+ inexpensive and usage-independent pricing 
 
 
## Docker Hub Entreprise


**Top Pro**
 
	* Supports LDAP
	* Comes with enterprise support for Docker Engine 1.6
 
 
##  Microsoft Container Registry
 
 The last one from microsoft, will host windows image, instead of docker hub. 
 November 2018.
 
 
 
 
 
 