# Questions 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


##  Common questions when evaluating a registry

### step 1 

    Workflow (build/deployment, collaboration, ease of use and visibility)
    Authentication and authorization
    Availability and performance
    Pricing

	In this article I compared a few hosted docker registry options. There are obviously pros and cons to all of these tools, and they are changing at a very fast pace right now. I think Quay is currently an excellent option for many organizations. It is inexpensive, offers unparalleled insight into docker registry usage, and can seamlessly integrate with your build and deployment pipelines.

Both Google Container Registry and Artifactory are targeted towards large enterprises with focus on high availability. They also integrate with LDAP to sync with organization’s user directory and groups. I think Google container registry can be a viable option if you are already invested in their cloud platform. In a similar vein to Google, Artifactory can be a great option for teams already using it for centralized artifact management. However, it can get expensive if you only plan to use it as a hosted docker registry.

Enterprise DockerHub is already a very good platform, and should be getting better with the recent announcement of a new version, which should be coming out any day now. Once that is available, I’ll dig in and look at it further.

### Step 2 

The main variables between the different registry offerings include what type of deployment environment they support (hosted, on-premise or both); how fine-tuned their access control options are; and how much additional security they provide for container registries. Choosing the right registry for your needs, of course, will depend on how these features align with your priorities. But with so many choices, it’s not difficult to find a registry that delivers the perfect balance for a given organization’s needs.