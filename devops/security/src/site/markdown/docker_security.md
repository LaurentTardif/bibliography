# docker security

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Sources 

 * https://blog.sonatype.com/docker-image-security-for-devsecops
 * https://secludit.com/blog/securite-de-docker/
 * https://github.com/arminc/clair-scanner
 
 
 
## Docker Image Security

Docker. It seems like in this day-and-age you are either using Docker containers or you are going to use Docker containers. If you haven’t jumped on the bandwagon yet, check out a previous article, Docker: The New Ordinary. If you are on the wagon or are thinking about it but have concerns about their security, it’s time to read on.

José Manuel Ortega (jmortega.github.io ) is a software engineer and security researcher in Spain.  I recently watched his presentation online entitled, Testing Docker Images Security. He gave an overview of typical Docker deployments, explained the attack surface and threats, presented how to detect vulnerabilities, and outlined a couple of best practices. In short, his advice will help you learn how to better secure your Docker containers.

[New to Docker? Read this paragraph; all others skip ahead.]  If you aren’t sure what Docker is, José offers this explanation, “Docker containers wrap a piece of software in a complete file system that contains everything it needs to run: code, runtime, system tools, system libraries - anything you can install on a server, regardless of the environment it is running in.”

That is, containers are isolated but share an operating system and, where appropriate, binaries and libraries. Docker provides an additional layer of isolation, making your infrastructure safer by default. This makes the application lifecycle faster and easier to configure, reducing risks in your applications.

For starters, José lays out Docker’s default mechanisms for security:

    Linux kernel namespaces
    Linux Control Groups (cgroups)
    The Docker daemon
    Linux capabilities (libcap)
    Linux security mechanisms like AppArmor or SELinux

 
![writablecontainer](./images/writablecontainer.png "Architecture Overview")


José walks through others tools, add-ons, best practices, etc. to increase Docker container security. I will cover most of them here.

Docker Inspect Tool. The Docker Inspect Tool is built into Docker. It provides information about the host name, the ID of the image, etc. and it comes up when you start Docker.

Docker Content Trust. It protects against untrusted images. It can enable signing checks on every managed host, guarantee integrity of your images when pulled, and provide trust from publisher to consumer.

Screen Shot 2018-04-08 at 7.19.01 AM

Docker File Security. Docker files build Docker containers. They should not write secrets, such as users and passwords. You should remove unnecessary setuid and setgid permissions, download packages securely using GPG and certificates, and try to restrict an image or container to one service.

Container Security. Docker security is about limiting and controlling the attack surface on the kernel. Don’t run your applications as root in containers, and create specific users for testing and policing the Docker image. Run filesystems as read-only so attackers can not overwrite data or save malicious scripts to the image.

José provided a useful checklist to check the security of a Docker container, but it’s not a short one.  Remember, if you are going to deploy hundreds or thousands of these containers, you’ll want to ensure consistent handling of security concerns to keep the hackers at bay:

    Do not write secrets to Docker files
    Create a user
    Follow version pinning for base images, packages, etc.
    Remove unnecessary setuid, setgid permissions
    Do not write any kind of update instructions alone in a Docker file

    Download packages securely
    Do not download unnecessary packages
    Use COPY instead of ADD
    Use the HEALTHCHECK command
    Use gosu instead of sudo whenever possible
    Use-no-cache (if applicable) when building

    Enable Docker Content Trust
    Ensure images are free from known vulnerabilities
    Ensure images are scanned frequently throughout your DevOps pipeline
    Ensure your images, packages are up-to-date
    Use file monitoring solutions for image layers (if required)


Auditing Docker Images. You can scan your images for known vulnerabilities with a wide variety of commercial and open source tools such as:

    Docker’s native Security Scanning
    Sonatype’s Nexus Lifecycle
    Twistlock,
    Docker Bench Security
    CoreOS Clair
    Dagda
    Aqua
    Tenable
    Anchore

 

All of these solutions can be integrated in one element of your CI/CD pipelines -- some can be integrated in multiple places.

José explored these solutions and best practices in more detail and offered up technical implementation tips in his full talk, available for free here. If you are working on or interested in Docker security, his talk is worth your time.

