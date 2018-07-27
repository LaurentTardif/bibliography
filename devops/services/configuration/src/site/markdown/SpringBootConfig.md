# Spring boot config

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->

## Sources 

 * https://spring.io/guides/gs/centralized-configuration/
 * https://cloud.spring.io/spring-cloud-config/

## Configuring application

	You will setup a Config Server and then build a client that consumes the configuration on startup and then refreshes the configuration without restarting the client.

## Maven dependencies 

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>configuration-service</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Finchley.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
	    <repository>
	        <id>spring-milestones</id>
	        <name>Spring Milestones</name>
	        <url>https://repo.spring.io/libs-milestone</url>
	        <snapshots>
	            <enabled>false</enabled>
	        </snapshots>
	    </repository>
	</repositories>

	</project>
	

## Stand up a Config Server

You will first need a Config Service to act as a sort of intermediary between your Spring applications and a typically version-controlled repository of configuration files. You can use Spring Cloud’s @EnableConfigServer to standup a config server that other applications can talk to. This is a regular Spring Boot application with one annotation added to enable the config server.

	configuration-service/src/main/java/hello/ConfigServiceApplication.java

	package hello;

	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.cloud.config.server.EnableConfigServer;

	@EnableConfigServer
	@SpringBootApplication
	public class ConfigServiceApplication {

		public static void main(String[] args) {
			SpringApplication.run(ConfigServiceApplication.class, args);
		}
	}

The Config Server needs to know which repository to manage. There are several choices here, but we’ll use a Git-based filesystem repository. You could as easily point the Config Server to a Github or GitLab repository, as well. On the file system, create a new directory and git init it. Then add a file called a-bootiful-client.properties to the Git repository. Make sure to also git commit it, as well. Later, you will connect to the Config Server with a Spring Boot application whose spring.application.name property identifies it as a-bootiful-client to the Config Server. This is how the Config Server will know which set of configuration to send to a specific client. It will also send all the values from any file named application.properties or application.yml in the Git repository. Property keys in more specifically named files (like a-bootiful-client.properties) override those in application.properties or application.yml.

Add a simple property and value, message = Hello world, to the newly created a-bootiful-client.properties file and then git commit the change.

Specify the path to the Git repository by specifying the spring.cloud.config.server.git.uri property in configuration-service/src/main/resources/application.properties. Make sure to also specify a different server.port value to avoid port conflicts when you run both this server and another Spring Boot application on the same machine.

	configuration-service/src/main/resources/application.properties

	server.port=8888
	spring.cloud.config.server.git.uri=${HOME}/Desktop/config

In this example we are using a file-based git repository at ${HOME}/Desktop/config. You can create one easily by making a new directory and git committing properties and YAML files to it. E.g.

	$ cd ~/Desktop/config
	$ find .
	./.git
	...
	./application.yml

Or you could use a remote git repository, e.g. on github, if you change the configuration file in the application to point to that instead.
Reading Configuration from the Config Server using the Config Client

Now that we’ve stood up a Config Server, let’s stand up a new Spring Boot application that uses the Config Server to load its own configuration and that refreshes its configuration to reflect changes to the Config Server on-demand, without restarting the JVM. Add the org.springframework.cloud:spring-cloud-starter-config dependency in order to connect to the Config Server. Spring will see the configuration property files just like it would any property file loaded from application.properties or application.yml or any other PropertySource.

The properties to configure the Config Client must necessarily be read in before the rest of the application’s configuration is read from the Config Server, during the bootstrap phase. Specify the client’s spring.application.name as a-bootiful-client and the location of the Config Server spring.cloud.config.uri in configuration-client/src/main/resources/bootstrap.properties, where it will be loaded earlier than any other configuration.

	configuration-client/src/main/resources/bootstrap.properties

	spring.application.name=a-bootiful-client
	# N.B. this is the default:
	spring.cloud.config.uri=http://localhost:8888

We also want to enable the /refresh endpoint so that we can demonstrate dynamic configuration changes:

	configuration-client/src/main/resources/application.properties

	management.endpoints.web.exposure.include=*

The client may access any value in the Config Server using the traditional mechanisms (e.g. @ConfigurationProperties, @Value("${…​}") or through the Environment abstraction). Create a Spring MVC REST controller that returns the resolved message property’s value. Consult the Building a RESTful Web Service guide to learn more about building REST services with Spring MVC and Spring Boot.

By default, the configuration values are read on the client’s startup, and not again. You can force a bean to refresh its configuration - to pull updated values from the Config Server - by annotating the MessageRestController with the Spring Cloud Config @RefreshScope and then by triggering a refresh event.

	configuration-client/src/main/java/hello/ConfigClientApplication.java

	package hello;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.cloud.context.config.annotation.RefreshScope;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@SpringBootApplication
	public class ConfigClientApplication {

		public static void main(String[] args) {
			SpringApplication.run(ConfigClientApplication.class, args);
		}
	}

	@RefreshScope
	@RestController
	class MessageRestController {

		@Value("${message:Hello default}")
		private String message;

		@RequestMapping("/message")
		String getMessage() {
			return this.message;
		}
	}

## Test the application

Test the end-to-end result by starting the Config Service first and then, once loaded, starting the client. Visit the client app in the browser, http://localhost:8080/message. There, you should see the String Hello world reflected in the response.

Change the message key in the a-bootiful-client.properties file in the Git repository to something different (Hello Spring!, perhaps?). You can confirm that the Config Server sees the change by visiting http://localhost:8888/a-bootiful-client/default. You need to invoke the refresh Spring Boot Actuator endpoint in order to force the client to refresh itself and draw the new value in. Spring Boot’s Actuator exposes operational endpoints, like health checks and environment information, about an application. In order to use it you must add org.springframework.boot:spring-boot-starter-actuator to the client app’s CLASSPATH. You can invoke the refresh Actuator endpoint by sending an empty HTTP POST to the client’s refresh endpoint, http://localhost:8080/actuator/refresh, and then confirm it worked by reviewing the http://localhost:8080/message endpoint.

	$ curl localhost:8080/actuator/refresh -d {} -H "Content-Type: application/json"

	we set management.security.enabled=false in the client app to make this easy to test (by default since Spring Boot 1.5 the Actuator endpoints are secure by default). By default you can still access them over JMX if you don’t set the flag.

## To take into account

* Security
* who can access the configuration
* consider automatic configuration of application in order to reduce configuration management accross all environments.

