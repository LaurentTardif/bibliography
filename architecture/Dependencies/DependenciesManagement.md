# Dependencies Management

## Challenges

Code projects can use numerous dependencies to accomplish the tasks the build process to produce the binaries requires. These dependencies may contain bugs or security holes. They may also evolve with each new version to add new features or correct flaws or bugs.

> In order to maintain applications correctly over time, it is essential to control the versions of the dependencies used.

In case of Java/Maven project, the dependencies:

* should be **necessary and sufficient**, no transitive dependencies explicitly define.
* should be **versionned**, do not let the system choose for you what's you are using.
* should **define the scope** (test, build, run), do not go to production with tests dependencies
* should be **updated regularly**, do not keep bug or security issues in your application
* avoid using **multiple dependencies** for the same functionality


## Reference documentation

* [Maven](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

## Maven's dependency resolution mechanism

When a dependency is present at several points in a project's dependency tree, the dependency that is "closest" is selected. (you may find the full details in "Dependency mediation" paragraph in the reference documentation mentioned above)

### Maven: Dependency mediation

* **Dependency mediation**: determines which version of an artifact will be chosen when several versions are encountered as dependencies. Maven chooses the "closest definition". In other words, it uses the version of the dependency closest to your project in the dependency tree. You can always guarantee a version by explicitly declaring it in your project's POM. Note that if two dependency versions are at the same depth in the dependency tree, the first declaration takes precedence.

* **"closest definition"** means that the version used will be the closest to your project in the dependency tree.

Consider this dependency tree:

```shell
 A
 ├── B
 │   └── C
 │       └── D 2.0
 └── E
    └── D 1.0
```

In the text, the dependencies for A, B and C are defined as A -> B -> C -> D 2.0 and A -> E -> D 1.0, so D 1.0 will be used when building A because the path from A to D to E is shorter. You can explicitly add a dependency on D 2.0 in A to force the use of D 2.0, as shown here :

```shell
  A
  ├── B
  │   └── C
  │       └── D 2.0
  ├── E
  │   └── D 1.0
  │
  └── D 2.0
```

We can see that this resolution mechanism has a weakness linked to the order in which dependencies are declared.

#### Maven: the fragility of dependency resolution

Consider the dependency tree defined in the project's pom.xml file:

```shell
  A
  ├── B
  │   └── C 1.0
  └── D
      └── C 2.0
```

In this case, the dependency that will actually be resolved is C 1.0.

If, however, during the life of the project, the order of declaration changes (which is likely to happen) and becomes:

```shell
  A
  ├── D
  │   └── C 2.0
  └── B
      └── C 1.0
```

In this case, the dependency that will actually be resolved is C 2.0. After such a change, the project's behavior may change: functionality may be altered, there may be an edge effect, or it may even be impossible to build.

This fragility (not necessarily known to everyone) shows that it's preferable to have a good grasp of a project's dependencies.

### Declare dependencies actually used

When the code of a maven project explicitly uses elements of a dependency (e.g. org.apache.commons.lang3.StringUtils#isNotBlank), then this dependency must be explicitly declared in the maven project's pom.xml file, rather than using this dependency transitively. This allows you to:

* avoid induced dependencies (which might otherwise be useless)
* eliminate dependencies that are no longer necessary.
* make construction impossible (ko compilation) following a change of dependency (the transitive dependency may disappear)
* understand the project's technical requirements, and avoid adding avoidable dependencies.

> For example, if you explicitly know that the project has a JSON manipulation library (e.g. com.fasterxml.jackson:jackson-databind), you're less likely to inadvertently add another JSON library (e.g. com.google.code.gson:gson), which is likely to be redundant.

## Transitive Dependencies Management

If an application A uses a dependency B and B uses a dependency C, then dependency C is a transitive dependency for application A. The C dependency should not be defined in the pom of your project. Maven will get it for you.

**Use only strictly necessary dependencies**

It is common to find unnecessary dependencies in maven projects. For example, if a batch uses the "spring-boot-starter-web" dependency, it's fair to ask: is the purpose of a batch to expose a REST API or to serve Web content?

Generally speaking, unnecessary dependencies requires unecessary work and generate risks:

* from a security point of view, they increase the attack surface.
* they can cause version conflicts with truly useful dependencies.
* from a resource point of view, they produce larger artifacts: more storage required, more memory needed, start-up time may be longer... etc.
* from a maintenance cost point of view: all project dependencies need to be managed throughout the life of the project.

## Strict versioning of dependencies

For efficient maintenance of a maven project, it is strongly recommended to activate the "dependency convergence" check as early as possible in the project's life (see below), and to cause the build to fail if this check is negative.

This ensures that all dependencies are unambiguously defined and that version conflicts are resolved as soon as they are detected.

### Resolution of a version conflict

When the "dependency convergence" check fails, divergent dependencies are indicated, along with their position in the project dependency tree. You must then choose which dependency to retain. There is no general rule, but there is one (fairly frequent) case where the choice is low-risk: provided that the versions of the dependencies follow the semver convention (major.minor.patch more details here: <https://semver.org/lang/fr/>), if the versions differ by only one patch or minor number (v.w.x vs v.w.y or v.w.x vs v.y.z), then the risk is theoretically null of choosing the most recent version.

However, in a situation where versions have distinct major numbers, the resolution can be much trickier: it's possible that choosing the most recent version will render the application dysfunctional: in some cases, the application will no longer compile: this is what happens if the project code uses the offending library.

The "obvious" solution is to adapt the project code to make it compatible with the most recent library.

 In other (pernicious) cases, problems are seen at runtime (typically java.lang.NoClassDefFoundError or java.lang.NoSuchMethodError exceptions). This problem is all the more pernicious in that it may only appear in certain cases that cannot be identified (unless you are very familiar with the inner workings of the dependencies at the origin of the version conflict). This kind of problem arises when two project dependencies each use an incompatible version of the conflicting dependency.

 There are several possible solutions:

* modify (when possible) the dependency that uses the oldest version of the conflicting dependency

* modify the versions of the user dependencies to find the version(s) where they each use a compatible version: but in general, this is to the detriment of functionality, as it will generally be necessary to lower the version number of one of the dependencies at the origin of the conflict.

* refactoring the project code so that it no longer depends on one of the libraries causing the version conflict

* if none of the above solutions is possible, create a "shaded" version of the dependency that uses the oldest conflicting dependency, using the appropriate maven plugin. This is an advanced technique that is relatively complex and time-consuming. It involves redefining the namespaces of the classes in the conflicting dependency and updating their usage in the user dependency.

For more information: <https://maven.apache.org/plugins/maven-shade-plugin/>. Once the "shaded" version has been created, the dependency causing the problem is replaced in the project's dependencies by its "shaded" version.

### Strict definition of a dependency's version

The "dependency-management" section of the project's pom.xml file allows you to unambiguously define the version of a dependency. This is where version conflicts are resolved.

### Using BOMs

Ensuring (and therefore testing) that different independent libraries work properly together can be a complex and time-consuming task. Some editors (such as Spring) provide Bill Of Materials (BOMs), which allow you to define sets of libraries that are compatible with each other. Importing (or inheriting via pom parent) these BOMs frees the developer from the need to manage versions of these libraries.

In the case of complex software projects, creating BOMs for libraries created for the needs of these projects can represent a real added value. By centralizing the management of library compatibility, you can significantly reduce the effort required to maintain the dependencies of each maven project. In this case, maven projects hardly need to define the versions of their dependencies.

In other words, the "dependencies" section of the pom.xml file should define virtually no versions (dependency sections do not include the "version" tag).

## Avoid using multiple dependencies for the same functionality

It is preferable to meet a specific project need with a single dependency. For example, avoid using two http clients (e.g. com.squareup.okhttp3:okhttp and org.apache.httpcomponents:httpclient) in the same project.

However, this recommendation is not always applicable: you may "suffer" from "redundant" transitive dependencies due to the use of libraries that have independently covered their own technical needs.

### Analyze existing (transitive) dependencies before adding a new one

When a project needs a new technical service, such as an http client, and there are several libraries that can provide this service, it's a good idea to look in the transitive dependencies to see if such a library isn't already present. This will avoid functional redundancy in the project's dependencies.

You should also check whether the jdk already provides the desired service. For example, there's no need to use the "joda-time" library, as the jdk has offered the date/time api (JSR 310) since Java 1.8.

### Beware of "phantom" dependencies

Some libraries have shaded some of their dependencies for their own purposes. This is the case, for example, with testcontainers, which has shaded (among others) the Jackson serialization library. For testcontainers, the namespace com.fasterxml.jackson is replaced by org.testcontainers.shaded.com.fasterxml.jackson. This means you can inadvertently declare an object mapper of type org.testcontainers.shaded.com.fasterxml.jackson.databind.ObjectMapper when you'd expect to use type com.fasterxml.jackson.databind.ObjectMapper.

## Pay attention to the scope of dependencies

It may happen, inadvertently, that test dependencies are included in production dependencies.

## Regular updating of dependencies

Updating a project's dependencies "as you go" is a commonly accepted best practice. In particular, it facilitates security-critical updates (e.g. log4shell). In the context of sensitive applications, this becomes imperative.

Tools such as 'dependabot', which integrate with Gitlab, are used to facilitate the process of regularly updating dependencies (the tool generates merge requests for dependency updates). Current practice is to perform a dependabot analysis, followed by a weekly update.

Updating dependencies can alter an application's behavior. Good test coverage helps address this risk.

## Tools

### Track dependency version conflicts: Dependency convergence

Here's an example of how to configure the "enforcer" maven plugin. This will cause the build to fail if version conflicts are detected

pom.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
...
    <build>
        <plugins>
            ...
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <configuration>
                            <rules>
                                <DependencyConvergence />
                            </rules>
                        </configuration>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            ...
        </plugins>
    </build>
...
</project>
```

Here's an example of a failed build:

Dependency convergence failure

``` shell
...
[WARNING]
Dependency convergence error for org.apache.xmlgraphics:xmlgraphics-commons:jar:2.3:compile paths to dependency are:
+-org.acme.xxx:yyy:jar:1.0.0-SNAPSHOT
  +-org.apache.xmlgraphics:fop:jar:2.3:compile
    +-org.apache.xmlgraphics:xmlgraphics-commons:jar:2.3:compile
and
+-org.acme.xxx:yyy:jar:1.0.0-SNAPSHOT
  +-org.apache.xmlgraphics:fop:jar:2.3:compile
    +-org.apache.xmlgraphics:batik-svg-dom:jar:1.10:compile
      +-org.apache.xmlgraphics:batik-css:jar:1.10:compile
        +-org.apache.xmlgraphics:xmlgraphics-commons:jar:2.2:compile
and
+-org.acme.xxx:yyy:jar:1.0.0-SNAPSHOT
  +-org.apache.xmlgraphics:fop:jar:2.3:compile
    +-org.apache.xmlgraphics:batik-bridge:jar:1.10:compile
      +-org.apache.xmlgraphics:xmlgraphics-commons:jar:2.2:compile
and
+-org.acme.xxx:yyy:jar:1.0.0-SNAPSHOT
  +-org.apache.xmlgraphics:fop:jar:2.3:compile
    +-org.apache.xmlgraphics:batik-awt-util:jar:1.10:compile
      +-org.apache.xmlgraphics:xmlgraphics-commons:jar:2.2:compile

[ERROR] Rule 0: org.apache.maven.plugins.enforcer.DependencyConvergence failed with message:
Failed while enforcing releasability. See above detailed error message.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
...
```

This result indicates that the _"org.apache.xmlgraphics:xmlgraphics-commons:jar"_ dependency is in version conflict, as it exists in versions 2.2 and 2.3 in the project. If you choose version 2.3, you can set this version in the "dependencyManagement" section of the pom.xml file.

### explicitly specifying the version of a dependency

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
...
    <dependencyManagement>
        <dependencies>
            ...
            <dependency>
                <groupId>org.apache.xmlgraphics</groupId>
                <artifactId>xmlgraphics-commons</artifactId>
                <version>2.3</version>
            </dependency>
             ...
         </dependencies>
    </dependencyManagement>
...
</project>
````

## Get an overview of dependencies

### Dependency analysis

The maven "dependency" plugin can be used to create an html report that provides information on a maven project:

* dependencies declared and actually used by the project code

* dependencies used by the project code but not declared (allows you to declare them). See paragraph Declaring dependencies actually used). However, you can choose not to take into account "common" spring dependencies (versions controlled by the editor via a BOM, supplied by spring boot starters, present in almost all spring modules).

* dependencies declared but not used by the code (allows only necessary dependencies to be used). Disregard spring boot starters, however, as they are merely containers for dependencies).

````shell
In the project repository:

mvn dependency:analyze-report
````

* produces the report ./target/dependency-analysis.html

Here is an example of the report produced

![Dependencies report](./images/dependencyReport.png "Dependencies Report")

### Find dependency updates

 Display dependency updates

The maven "versions" plugin lists the dependency updates available for a project. The documentation is [here :]( https://www.mojohaus.org/versions/versions-maven-plugin/display-dependency-updates-mojo.html)

versions:display-dependency-updates (exemple)

``` shell
# In the project repository

mvn versions:display-dependency-updates

# result (extract)
...
[INFO] The following dependencies in Dependency Management have newer versions:
[INFO]   antlr:antlr ........................................ 2.7.7 -> 20030911
[INFO]   ch.qos.logback:logback-access ....................... 1.2.12 -> 1.4.14
...
[INFO] The following dependencies in Dependencies have newer versions:
[INFO]   com.fasterxml.jackson.dataformat:jackson-dataformat-xml ...
[INFO]                                                         2.15.2 -> 2.17.0
[INFO]   com.google.zxing:core ................................. 3.5.0 -> 3.5.3
...
[INFO] The following dependencies in pluginManagement of plugins have newer versions:
[INFO]   org.springframework.boot:spring-boot-maven-plugin .... 2.7.16 -> 3.2.4
...
[INFO] The following dependencies in Plugin Dependencies have newer versions:
[INFO]   org.liquibase:liquibase-core ........................ 4.22.0 -> 4.26.0
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
