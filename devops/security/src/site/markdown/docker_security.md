# docker security

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Sources 

 * https://blog.sonatype.com/docker-image-security-for-devsecops
 * https://secludit.com/blog/securite-de-docker/
 * https://github.com/arminc/clair-scanner
 
 
 
## Docker Image Security

Docker. It seems like in this day-and-age you are either using Docker containers or you are going to use Docker containers. If you haven’t jumped on the bandwagon yet, check out a previous article, Docker: The New Ordinary. If you are on the wagon or are thinking about it but have concerns about their security, it’s time to read on.

Jose Manuel Ortega (jmortega.github.io ) is a software engineer and security researcher in Spain.  I recently watched his presentation online entitled, Testing Docker Images Security. He gave an overview of typical Docker deployments, explained the attack surface and threats, presented how to detect vulnerabilities, and outlined a couple of best practices. In short, his advice will help you learn how to better secure your Docker containers.

[New to Docker? Read this paragraph; all others skip ahead.]  If you aren’t sure what Docker is, Jose offers this explanation, 'Docker containers wrap a piece of software in a complete file system that contains everything it needs to run: code, runtime, system tools, system libraries - anything you can install on a server, regardless of the environment it is running in.'

That is, containers are isolated but share an operating system and, where appropriate, binaries and libraries. Docker provides an additional layer of isolation, making your infrastructure safer by default. This makes the application lifecycle faster and easier to configure, reducing risks in your applications.

For starters, Jose lays out Docker’s default mechanisms for security:

    Linux kernel namespaces
    Linux Control Groups (cgroups)
    The Docker daemon
    Linux capabilities (libcap)
    Linux security mechanisms like AppArmor or SELinux

 
![writablecontainer](./images/writablecontainer.png "Architecture Overview")


Jose walks through others tools, add-ons, best practices, etc. to increase Docker container security. I will cover most of them here.

Docker Inspect Tool. The Docker Inspect Tool is built into Docker. It provides information about the host name, the ID of the image, etc. and it comes up when you start Docker.

Docker Content Trust. It protects against untrusted images. It can enable signing checks on every managed host, guarantee integrity of your images when pulled, and provide trust from publisher to consumer.

![Docker Content Trust](./images/DockerContentTrust.png "Docker Content Trust")

Docker File Security. Docker files build Docker containers. They should not write secrets, such as users and passwords. You should remove unnecessary setuid and setgid permissions, download packages securely using GPG and certificates, and try to restrict an image or container to one service.

Container Security. Docker security is about limiting and controlling the attack surface on the kernel. Don’t run your applications as root in containers, and create specific users for testing and policing the Docker image. Run filesystems as read-only so attackers can not overwrite data or save malicious scripts to the image.

Jose provided a useful checklist to check the security of a Docker container, but it’s not a short one.  Remember, if you are going to deploy hundreds or thousands of these containers, you’ll want to ensure consistent handling of security concerns to keep the hackers at bay:

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

Jose explored these solutions and best practices in more detail and offered up technical implementation tips in his full talk, available for free here. If you are working on or interested in Docker security, his talk is worth your time.



## Docker security 2 

Par effet de mode, lobbying technologique ou besoin, les architectures logicielles evoluent. Ainsi, diverses architectures se sont succede :

    Monolithique.
    Composant / Objet (DCOM/CORBA)
    Service (SOA)
    Micro-services

Chaque mutation apporte son lot d’avantages tout comme son lot de nouveaux defis. Dans cet article, nous allons parcourir les defis lies a la securite des Micro-Services et plus particulierement la securite autour de Docker.
L’adoption des micro-services est nee d’une volonte d’accroître la modularite et l’agilite des applications mais ils complexifient la securisation de nos systemes d’informations.

Le challenge de la securisation des micro-services

Autrefois, il fallait assurer la securite d’une unique application. Aujourd’hui, il est necessaire de proteger une multitude de services ainsi que leurs implementations. Ils interagissent les uns avec les autres a travers differentes API et flux de donnees qui voient le jour aussi rapidement qu’ils disparaissent. Plus il y a des services, plus il y a d’elements exposes aux cyber-attaques ainsi qu’aux mauvaises utilisations.

L’article, ecrit par Graham Lea, fait office de reference et de livre blanc pour la securisation des micro-services car il etablit une liste exhaustive des meilleures pratiques a mettre en oeuvre. Neanmoins, l’article ne decrit pas comment et quels outils utiliser pour appliquer ces regles.

 
### Quelle politique de securite pour Docker ?

Pour repondre au developpement des microservices, l’Industrie Informatique s’est tournee vers l’utilisation des conteneurs et notamment, vers son gestionnaire le plus mediatique : Docker.

Cependant, on ne peut pas auditer un conteneur docker comme un serveur classique :

    par principe les conteneurs ne sont pas accessibles via ssh.
    l’isolation des conteneurs ne les rende pas visible depuis le serveur docker.

Il est donc inutile d’utiliser un scanner SAST traditionnel car il est impossible de faire un inventaire des librairies ou même d’avoir acces aux fichiers de configuration des logiciels utilises par le conteneur.

Neanmoins voici les principaux conseils de securite indispensables pour votre environnement docker :

 
### La securite des images Docker

La mise en place de conteneurs Docker passe par l’utilisation d’images. C’est le premier maillon de la chaîne qu’il va falloir valider, avec une petite revolution : une partie des tests de securite qui etaient consacres aux Ops ont pu être migres dans la chaîne d’integration continue des Devs. Ils sont menes lors des operations de build ou de post- des images.

Ces tests peuvent être executes par des services comme ‘Docker scanning service’ de Docker.com, ‘Twistlock Trust’ developpe par twistlock.com, ou en utilisant la solution opensource ‘Clair’ de Coreos.com.

Ces solutions d’inspection des conteneurs (Deep Container Inspection, DCI) ont pour but de s’assurer que les outils et logiciels embarques dans les images utilisees par les conteneurs ne sont pas vulnerables. L’exploitation de vulnerabilites pourrait amener une personne mal intentionnee a prendre le contrôle du conteneur. Elle pourrait alors tenter d’obtenir les droits root (une elevation de privilege) sur l’hôte du conteneur, et/ou tenter de prendre la main sur les conteneurs voisins.

 
### La securite des images dans Docker Hub

Par simplicite ou besoin de prototypage rapide, vous serez peut être amene a utiliser des images deja existantes issues de registry.
Il est faut être vigilant car le ‘Docker Hub Registry’ propose une multitude d’images telechargeables. Preferez toujours les images officielles proposees et validees par Docker. Elles sont mises a jour regulierement, bien documentees et auditees par le service ‘Docker scanning service’.

Si votre choix doit se porter sur l’utilisation d’une image non officielle, assurez-vous qu’elle ait ete suffisamment telechargee au prealable. Il faut egalement, consulter les commentaires et verifier le ‘Dockerfile’ sur github afin de s’assurer que l’image corresponde bien a ce qu’il est indique.

Si vous vous preparez a utiliser un registre tiers alors que l’image n’est pas disponible sur Docker Hub, verifiez sa reputation en vous assurant que la communication etablie avec ce nouveau registre se fera de maniere securisee.

 
### La securite du socle Docker Engine

Pour executer les conteneurs, il vous faut configurer correctement le socle ‘Docker Engine.’

security-dockerDes organismes comme Center for Internet Security (CIS) et Gotham Digital Science ont liste des recommandations a appliquer. Elles sont decrites dans les documents suivants :

    https://benchmarks.cisecurity.org/tools2/docker/CIS_Docker_1.11.0_Benchmark_v1.0.0.pdf
    https://github.com/GDSSecurity/Docker-Secure-Deployment-Guidelines

Le projet open source ‘docker-bench-security‘ reprend les recommandations du CIS sous forme d’un script a executer. Il permet de s’assurer de l’application des bonnes pratiques de securite sur la machine hôte executant les conteneurs. On retrouve ici les regles du hardening appliquees a Docker, a sa configuration et a la bonne mise en place des security profile (apparmor, seccomp, selinux).

Attention, toutefois car les outils n’automatisent pas l’integralite des recommandations. Certains tests ne peuvent être evalues que lors d’un audit en fonction de l’environnement deploye et des besoins a couvrir.

 
###  La securite du partage des donnees

Soyez egalement vigilant avec le partage des donnees entre vos conteneurs (Volume dans le langage docker), pour :
ne pas exposer plus d’informations que necessaire car utiliser des containers c’est aussi pouvoir compartimenter les informations ;
eviter de perdre de l’espace sur le disque du serveur hôte, par exemple l’image elasticsearch qui partage le repertoire /usr/share/elasticsearch/data*

*Ces informations sont disponible au travers des commandes ‘docker volume ls’ et ‘docker volume inspect’.

 
### La securite de l’ecosysteme Docker

Il ne faut pas non plus omettre la securisation de l’ecosysteme qui gravite autour de docker : swarm, kubernetes, mesos, nomad, traefik, flocker, etc. Cet ecosysteme grossit vite, (trop vite) mais heureusement la securite est devenue une priorite. A titre d’exemple, la version Docker 1.12 qui integrait nativement swarm, embarque desormais le mecanisme de chiffrement des echanges en facilitant la generation et le deploiement des certificats entre les noeuds du cluster.
Ainsi, apres chaque mise a jour de vos outils docker, faîtes attention au changelog, pour verifier que de nouvelles fonctionnalites touchant a la securite n’ont pas ete modifiees.

 
### Ameliorer la securite de Docker est possible

Decriee a ses debuts pour des failles de securite importantes (privileges trop importants, faiblesse du cloisonnement), l’engagement pris par Salomon Hykes et son equipe commence a porter ses fruits.

    'Au fur et a mesure que nous grandissons, nous allons poursuivre notre investissement pour renforcer notre equipe de securite a travers des contributions, des outils et des processus. Cet investissement fera de Docker un outil plus sur, l’aidant a devenir un partenaire sur et fiable pour nos utilisateurs', DEC.2014

Nous venons de vous presenter les premiers outils et quelques conseils pour vous assurer de la securite des infrastructures microservices,
Nous avons rassembles une partie de ces meilleures pratiques de securite dans Elastic Detector pour une automatisation et un deploiement rapides, faites le test et partagez vos commentaires.

Ressources :

 * http://www.grahamlea.com/2015/07/microservices-security-questions/
 * https://docs.docker.com/docker-cloud/builds/image-scan/
 * https://www.twistlock.com/
 * https://github.com/coreos/clair
 * http://rhelblog.redhat.com/2015/09/03/what-is-deep-container-inspection-dci-and-why-is-it-important/
 * https://hub.docker.com/explore/
 * https://github.com/docker/docker-bench-security
 * https://benchmarks.cisecurity.org/tools2/docker/CIS_Docker_1.11.0_Benchmark_v1.0.0.pdf
 * https://github.com/GDSSecurity/Docker-Secure-Deployment-Guidelines
 * http://www.cyberciti.biz/tips/linux-security.html
 * https://d3oypxn00j2a10.cloudfront.net/assets/img/Docker%20Security/WP_Intro_to_container_security_03.20.2015.pdf


## Clair scanner 


Docker containers vulnerability scan

When you work with containers (Docker) you are not only packaging your application but also part of the OS. It is crucial to know what kind of libraries might be vulnerable in your container. One way to find this information is to look at the Docker registry [Hub or Quay.io] security scan. This means your vulnerable image is already on the Docker registry.

What you want is a scan as a part of CI/CD pipeline that stops the Docker image push on vulnerabilities:

    Build and test your application
    Build the container
    Test the container for vulnerabilities
    Check the vulnerabilities against allowed ones, if everything is allowed then pass otherwise fail

This straightforward process is not that easy to achieve when using the services like Docker Hub or Quay.io. This is because they work asynchronously which makes it harder to do straightforward CI/CD pipeline.
Clair to the rescue

CoreOS has created an awesome container scan tool called Clair. Clair is also used by Quay.io. What clair does not have is a simple tool that scans your image and compares the vulnerabilities against a whitelist to see if they are approved or not.

This is where clair-scanner comes into place. The clair-scanner does the following:

    Scans an image against Clair server
    Compares the vulnerabilities against a whitelist
    Tells you if there are vulnerabilities that are not in the whitelist and fails
    If everything is fine it completes correctly

	
	
	