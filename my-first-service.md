# My first service

This chapter provides a tutorial in which we will build a simple service, step by step.  Each step builds on the previous ones and introduces a new aspect of dCache.  Together, this provides a whistle-stop tour of the different aspects of dCache you are likely to encounter as a developer.

## A service to say Hello World!

In this section we introduce services, batch files, dCache layout configuration and the domain log file.

A node or computer within the dCache cluster runs one or more Java Virtual Machines \(JVMs\), called domains.  Each domain \(i.e., each JVM\) may be started or stopped independently using the `dcache` script.  The `dcache start` and `dcache stop` commands take optional arguments: the names of the domains to start or stop \(respectively\).  Although it is common to end domain names with `Domain`\(e.g., `AdminDomain`\), there's no need to do so.

Each domain can do any amount of work.  In system-test, for example, everything is placed in a single domain called `dCacheDomain`.  In production environments, the functionality of dCache is split over many domains.  Each node in the dCache cluster must have at least one domain, so that it can participate in the dCache cluster.  Nodes can also have multiple domains.

Services are the smallest unit of deployable work with each domain hosts one or more service.  Each service has a type, which describes what kind of work it does.  For example a `pool` service is resposible for storing data, a `webdav` service provides access to dCache via HTTP and WebDAV protocols, and a `pnfsmanager` service is resposible for dCache interacting with the namespace.

The layout file is resposible for describing which domains should run on a node and which services are hosted by each domain.  Domains are defined by a line containing their name in square brackets; e.g.,`[dCacheDomain]` declares that the node should run a domain called `dCacheDomain`.  A service is declared by placing the service type after the domain name and a slash all within square brackets; e.g.,`[dCacheDomain/pool]` describes a `pool` service that should run within the `dCacheDomain` domain.  Service declarations must come after the domain declaration.  The layout file can also contain configuration \(lines that contain a `=`\), but we'll come to that in the next section.

Ultimately, a service is a list of operations to establish that particular service, written in a language called batch script.  Each service has its own batch script, each with the service name and `.batch` as the file name; for example, `pool.batch` is the batch file for the `pool` service.  The batch files are copied directly into whichever package is being build and are located in the `services` directory.  The exact location of this directory varies for different package types; for FHS compliant deployments, the directory is `/usr/share/dcache/services`.  Within the git repo, this directory is `skel/share/services`.

To create our new service, we must create a file `simple.batch` in the `skel/share/services` directory with the following content:

```
#
#      Simple service
#

say -level=fsay Hello World!
```

In batch script language, the `say` command logs some information, with the arguments providing the message to be logged.  The `level` option describes at which log-level the message will be logged; `fsay` means the message is to be logged at error-level.  This is needed so the message will appear in the domain log file.

We also need to update system-test so that it runs our new service.  The source for the layout file for system-test is `packages/system-test/src/main/skel/etc/layouts/system-test.conf`.  Add the line `[dCacheDomain/simple]` immediately before some other `[dCacheDomain/<service>]` line; for example,

```
...

[dCacheDomain]
# The following is defined for the domain to prevent that the CLI
# applications enable the debugging options.
dcache.java.options.extra=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=localhost:2299,suspend=n -XX:+TieredCompilation

[dCacheDomain/simple]
[dCacheDomain/admin]
admin.paths.history=${system-test.home}/var/admin/history

[dCacheDomain/zookeeper]
...
```

In this example, the `[dCacheDomain/simple]` line is the first service defined in the `dCacheDomain` domain, immediately before the `admin` service.

When starting a domain, dCache will start each service in the order listed in the layout file.  If our`simple` service as the first service,  it will be the first service to be started.

Rebuild system-test using the command `mvn -am -pl packages/system-test clean package -DskipTests -Pstart`.  Once this command completes successfully, dCache is starting in the background.  The `dcache status` command shows the current status of each domain configured for this node.  For system-test, this script is available as `packages/system-test/target/dcache/bin/dcache`; for example,

```
paul@celebrimbor:~/git/dCache (tutorial/step-1)$ packages/system-test/target/dcache/bin/dcache status
DOMAIN       STATUS                   PID   USER LOG                                                                               
dCacheDomain running (for 16 minutes) 12950 paul /home/paul/git/dCache/packages/system-test/target/dcache/var/log/dCacheDomain.log 
paul@celebrimbor:~/git/dCache (tutorial/step-1)$
```

The output shows some simple statistics for each domain, including the location of the domain log file.  By default, the domain log filename is  the domain name with `.log`; e.g., `dCacheDomain.log`.  With FHS packages, this is located in the `/var/log/dcache directory`and  for system-test log files are written into the `packages/system-test/target/dcache/var/log` directory.

The `simple` service, when started, will log a simple message.  As this is the first service, the message will appear near the top of the log file:

```
paul@celebrimbor:~/git/dCache (tutorial/step-1)$ head packages/system-test/target/dcache/var/log/dCacheDomain.log

2017-07-24 10:47:08 Launching /usr/lib/jvm/java-8-openjdk-amd64/bin/java -server -Xmx1024m -XX:MaxDirectMemorySize=256m -Dsun.net.inetaddr.ttl=1800 -Dorg.globus.tcp.port.range=23000,25000 -Dorg.dcache.dcap.port=0 -Dorg.dcache.net.tcp.portrange=33115:33145 -Djava.security.krb5.realm= -Djava.security.krb5.kdc= -Djavax.security.auth.useSubjectCredsOnly=false -Djava.security.auth.login.config=/home/paul/git/dCache/packages/system-test/target/dcache/etc/jgss.conf -Dzookeeper.sasl.client=false -Dcurator-dont-log-connection-problems=true -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/paul/git/dCache/packages/system-test/target/dcache/var/log/dCacheDomain-oom.hprof -XX:+UseCompressedOops -javaagent:/home/paul/git/dCache/packages/system-test/target/dcache/share/classes/aspectjweaver-1.8.10.jar -Djava.awt.headless=true -DwantLog4jSetup=n -Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=localhost:2299,suspend=n -XX:+TieredCompilation -Ddcache.home=/home/paul/git/dCache/packages/system-test/target/dcache -Ddcache.paths.defaults=/home/paul/git/dCache/packages/system-test/target/dcache/share/defaults org.dcache.boot.BootLoader start dCacheDomain
Listening for transport dt_socket at address: 2299
INFO  - system-test.conf:1: Property system-test.home is not a standard property
INFO  - system-test.conf:161: Property frontend.net.internal is not a standard property
24 Jul 2017 10:47:11 (System) [] ZooKeeper connection to localhost/0:0:0:0:0:0:0:1:2181 failed (Connection refused), attempting reconnect.
24 Jul 2017 10:47:11 (System) [] Hello World! 
24 Jul 2017 10:47:11 (System) [] ZooKeeper connection to localhost/127.0.0.1:2181 failed (Connection refused), attempting reconnect.
24 Jul 2017 10:47:12 (System) [] ZooKeeper connection to localhost/0:0:0:0:0:0:0:1:2181 failed (Connection refused), attempting reconnect.
24 Jul 2017 10:47:12 (System) [] ZooKeeper connection to localhost/127.0.0.1:2181 failed (Connection refused), attempting reconnect.
paul@celebrimbor:~/git/dCache (tutorial/step-1)$
```

Here is a brief explanation of each of these lines:

* `2017-07-24 10:47:08 Launching` describes the command used to start this domain JVM.
* `Listening for transport` this line comes from OpenJDK's built-in debugger interface, which system-test enables.
* `INFO  - system-test.conf:` these lines come from dCache's built-in configuration checking.  It is currently not powerful enough to detect all problems, so this information is provided as a hint.
* `24 Jul 2017 10:47:11 (System) []` These lines are regular dCache logging lines.  The timestamps are from the system current clock; the value in parentheses is the name of the cell logging this information \(we'll learn more about cells in a bit\), the square brackets contains contextual information and the rest is the logged message.
* `ZooKeeper connection to` these messages are from the dCache ZooKeeper integration as it attempts to connect to the ZooKeeper instance.  In system-test, dCache starts a single embedded ZooKeeper server \(the `zookeeper` service\) as the ZooKeeper instance, so these messages appear until that service has started.
* `Hello World!` the message from our `simple` service.

## Being flexible: dCache configuration

Before we make our service configuration, here's a quick recap on dCache configuration.

dCache has a sophisticated configuration system that has been developed based on feedback from admins over the years. It includes advanced features, such as a configuration property lifecycle \(being able to retire configuration in a controlled fashion\), clear default values, value restrictions; features that many other configuration systems lack.  Developers are _strongly_ encouraged to use dCache configuration when making their contribution flexible, even when that configuration adjusts third-party libraries.

Configuration property assignments are name-value pairs separated by an equals sign \(`=`\).  The can be optional white-space on either side of the equals sign, so `foo=bar`, `foo= bar` and `foo = bar` are equivalent assignments.

In an assignment, the property name is the part before the equals sign.  Names are all lower-case and form a dot-separated hierachy.  This hierachy is nominal \(one cannot usefully specify a 'branch' in the configuration\), but it helps control the property names.  The first element in the hierachy describes which service the property affects.  If the property affects multiple services then `dcache` is used; for example, the `ftp.net.listen` describes the network interface the `ftp` service uses, while `dcache.net.listen` describes the network interface services \(including `ftp`\) will use if no service-specific configuration is specified.

Although it is possible to include spaces in property names, it requires escaping and it therefore clumsy to use and should be avoided.  A dash character should be used instead of spaces; for example,`ftp.net.port-range`is the "port range" property, part of the network \(`net`\) 'branch' of the properties affecting the`ftp`service.

In an assigment, the value is the part after the equals sign with any leading or trailing white space removed and property references resolved.  A property reference is written `${<name>}`, where `<name>` is the configuration property name.  This allows configuration properties to inherit default values and even have different default values depending on other property values.

Property assignment can appear in several places: the defaults files, the dcache.conf file and the layout file.

Default values appear in the `defaults` directory \(`/usr/share/dcache/defaults` in FHS packages; `skel/share/defaults` in git\).  The properties that affect only some specific service are located in a file in this directory called `<service>.properties`; for example, the properties that only affect the `ftp` service is located in the `ftp.properties` file \(`skel/share/defaults/ftp.properties` in git\).  The default assignments for properties that affect multiple services go in the `dcache.properties` file. Therefore, all properties with names that start `ftp.` have default values in the `ftp.properties` file.

Although modifying files in the `defaults` directory will result in modified dCache behaviour, any upgrade will lose those changes and admins are advised not to do this.

The first file to place configuration is in the `dcache.conf` file.  Typically, this file is common to all nodes in the dCache cluster, so should contain any configuration that is the same for all nodes.

The layout file defines configuration, domains and services.  It also allows configuration that is specific to this node \(before the first domain declaration\), to a particular domain \(after the domain but before the first service\), or a particular service \(after that service.

So, let's make our service configurable.  To do this, we need a file in the `defaults` directory, called `simple.properties`, that contains the default property values for our `simple` service, and update our service to make use of that configurable property.

First, create the `skel/share/defaults/simple.properties` file with the following content:

```
#  -----------------------------------------------------------------------
#     Default values for the simple service
#  -----------------------------------------------------------------------
@DEFAULTS_HEADER@

#  The greeting text to use
#
simple.greeting = Hello World!
```

During the build process, the `@DEFAULTS_HEADER@` is repaced with a standard set of useful information.

Next, update our service's batch file \(`skel/share/services/simple.batch`\) to the following content:

```
#
#      Simple service
#

onerror shutdown

check -strong simple.greeting

say -level=fsay ${simple.greeting}
```

The `onerror shutdown`mean that dCache will stop during the start-up process if there is an error.  The `check` command returns with an error if the argument \(`simple.greeting`\) has not been defined; the `-strong` option means the command also fails if the value is empty.

Rebuilding system test \(`mvn -am -pl packages/system-test clean package -DskipTests -Pstart`\) yields the expected greeting in `packages/system-test/target/dcache/var/log/dCacheDomain.log`:

```
2017-07-24 14:08:31 Launching /usr/lib/jvm/java-8-openjdk-[...]
Listening for transport dt_socket at address: 2299
INFO  - system-test.conf:1: Property system-test.home is not a standard property
INFO  - system-test.conf:160: Property frontend.net.internal is not a standard property
24 Jul 2017 14:08:33 (System) [] ZooKeeper connection to localhost/127.0.0.1:2181 failed (Connection refused), attempting reconnect.
24 Jul 2017 14:08:33 (System) [] Hello, world!
```

The difference is that we can now change this greeting without changing the batch file.

Edit the layout file in system-test \(`packages/system-test/target/dcache/etc/layouts/system-test.conf`\) and add the extra line immediately after the `[dCacheDomain/simple]` line:

```
...

[dCacheDomain]
# The following is defined for the domain to prevent that the CLI
# applications enable the debugging options.
dcache.java.options.extra=-Xdebug -Xrunjdwp:[...]

[dCacheDomain/simple]
simple.greeting = Aloha Honua!
[dCacheDomain/admin]
...
```

Take care **NOT** to rebuild system-test after adding the `simple.greeting = Aloha Honua!` line to system-test.conf as you will loose that edit when dCache rebuilds system-test.  If you want your changes to survive system-test being rebuild, modify `packages/system-test/src/main/skel/etc/layouts/system-test.conf`; however, then the changes only take affect after system test is rebuilt.

To see the effect of our new configuration, we simply restart dCache  with the `dcache restart` command:

```
paul@celebrimbor:~/git/dCache (tutorial/step-2)$ packages/system-test/target/dcache/bin/dcache restart
Stopping dCacheDomain 0 1 2 3 4 5 6 7 8 9 10 done
Starting dCacheDomain done
paul@celebrimbor:~/git/dCache (tutorial/step-2)$
```

You will see the new greeting in the domain log file: `packages/system-test/target/dcache/var/log/dCacheDomain.log`

```
2017-07-24 14:19:25 Launching /usr/bin/java -server [...]
Listening for transport dt_socket at address: 2299
INFO  - system-test.conf:1: Property system-test.home is not a standard property
INFO  - system-test.conf:161: Property frontend.net.internal is not a standard property
24 Jul 2017 14:19:28 (System) [] ZooKeeper connection to localhost/127.0.0.1:2181 failed (Connection refused), attempting reconnect.
24 Jul 2017 14:19:28 (System) [] Aloha Honua!
```

The new greeting won't be at the top of the file because \(by default\) dCache does not start a new file with each domain restart.

One last remark: the more observant reader will have noticed that dCache packages the batch scripts.  Therefore, to change the greeting we could have edited the batch file for the `simple` service \(`packages/system-test/target/dcache/share/services/simple.batch`\).  While this will work, we don't intend admins to modify batch scripts and upgrading dCache will overwrite any changes.

## Introducing spring: bringing Java into the game

So far, we haven't used any Java code.  We'll change this now by replacing the `say` batch command with some simple Java code that does the same thing.

Before that, we need a short introduction to cells in dCache.  Cells are dCache components that have a specific task and achieve this by communicating with other cells using messages.  A domain typically hosts several cells, with most services creating a single cell.  A cell has a domain-unique identity, can send and receive messages from other cells, uses a separate threads from other cells, and can have different logging configuration.

So, in this section we are going to create a very simple new cell that simply greets the new world.

Most services in dCache have their own maven module, which produces a jar file.  We'll follow that convension here.

### Creating a new module

In git, the majority of the maven modules are in the `modules` directory, each module with its own subdirectory.

Create the a directory `modules/dcache-simple`, and create a new `pom.xml` file with the following content:

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.dcache</groupId>
        <artifactId>dcache-parent</artifactId>
        <version>3.2.0-SNAPSHOT</version>
        <relativePath>../../pom.xml</relativePath>
    </parent>

    <artifactId>dcache-simple</artifactId>
    <packaging>jar</packaging>

    <name>dCache simple service</name>
</project>
```

The pom.xml file controls how maven will build the `dcache-simple.jar` file.

The first five lines are basically boiler-plate for a maven `pom.xml` file.  The details within the `parent` element indicates from which maven module certain values are derived.  The `packaging` element describes what this module produces; in this case, a jar file.  The `artifactId` identifies this module within maven; it is also used to generate the jar filename.  Finally, the `name` element is used when generating human-consumable output.

We also need two more changes: to tell our parent project of this new module and the packaging modules that there's another jar file to include.

To add our new module in the parent module \(which has an artifactId of `dcache-parent`\), update the  `pom.xml` file located in the root of the git checkout. The `modules` element lists all modules. You need to add an additional line to it:  `<module>modules/dcache-simple</module>`.  Focusing on only that part of the file, the result should look like:

```
...
        <module>modules/dcache-webadmin</module>
        <module>modules/dcache-restful-api</module>
        <module>modules/dcache-nfs</module>
        <module>modules/dcache-simple</module>
        <module>modules/dcache-srm</module>
        <module>modules/dcache-info</module>
...
```

Finally, we need to include this new jar file in the different dCache packaging.  To do this, update the `packages/pom.xml` file to include an additional dependency.  This involves adding a new stanza to the `dependencies` element, like:

```
    <dependency>
        <groupId>org.dcache</groupId>
        <artifactId>dcache-simple</artifactId>
        <version>${project.version}</version>
    </dependency>
```

With these changes, maven will build the dcache-simple.jar file.  Maven shows a summary of the modules it built after it completes; this will now be a `dCache simple service` line in that output:

```
[INFO] dCache Info Service ................................ SUCCESS [  0.620 s]
[INFO] webadmin ........................................... SUCCESS [  1.425 s]
[INFO] dCache NFSv4.1/pNFS ................................ SUCCESS [  0.318 s]
[INFO] dCache simple service .............................. SUCCESS [  0.011 s]
[INFO] missing-files SEMsg plugin ......................... SUCCESS [  0.078 s]
```

There will also be a jar file in system-test:

```
paul@celebrimbor:~/git/dCache (master)$ ls packages/system-test/target/dcache/share/classes/dcache-simple-*
packages/system-test/target/dcache/share/classes/dcache-simple-3.2.0-SNAPSHOT.jar
paul@celebrimbor:~/git/dCache (master)$
```

### Creating the cell

Now that we have the new dcache-simple module, which generates the `dcache-simple-*.jar`file, we can now create our simple cell.

We need to define the default name for our cell.  Following the convention from other cells, this configuration property is called `simple.cell.name` and should be found in the `skel/share/defaults/simple.properties` file.  Let's update this file now by adding the definition of the `simple.cell.name` property:

```
# -----------------------------------------------------------------------
# Default values for the simple service
# -----------------------------------------------------------------------
@DEFAULTS_HEADER@

# The greeting text to use
#
simple.greeting = Hello, world!

# ---- Cell names
#
simple.cell.name = simple
```

Now, we can update `skel/share/services/simple.batch` by replacing the `say` command with a `create` command.  The batch file should look like:

```
#
#      Simple service
#

onerror shutdown

check -strong simple.greeting

create org.dcache.cells.UniversalSpringCell ${simple.cell.name} \
        "classpath:org/dcache/simple/simple.xml"
```

This `create` command is resposible for starting a new cell using the named class \(first argument\) and giving it the name from the second argument.  The meaning of the third argument is specific to the cell: for UniversalSpringCell, for now we only need one argument: the location of the Spring XML file it will use to build the cell.

Let's now create the Java class that will generate the log message.  Create the file `modules/dcache-simple/src/main/java/org/dcache/simple/Greeter.java` with the following content:

```java
/*
 * dCache - http://www.dcache.org/
 *
 * Copyright (C) 2017 Deutsches Elektronen-Synchrotron
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU Affero General Public License as
 * published by the Free Software Foundation, either version 3 of the
 * License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU Affero General Public License for more details.
 *
 * You should have received a copy of the GNU Affero General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 */
package org.dcache.simple;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Required;

import static java.util.Objects.requireNonNull;

/**
 * This class emits a customisable message.
 */
public class Greeter
{
    private static final Logger LOG = LoggerFactory.getLogger(Greeter.class);
    private String _message;

    @Required
    public void setMessage(String message)
    {
        _message = requireNonNull(message);
    }

    public String getMessage()
    {
        return _message;
    }

    public void greet()
    {
        LOG.warn("{}", _message);
    }
}
```

A few points to note with this code:

* It has a copyright notice at the top: all new code should include such a notice.
* The code is using Simple Logging Facade for Java \(slf4j\).  This is the standard logging framework in dCache.
* The `@Required` annotation comes from Spring.  It means that, if this value is not set, Spring will fail when creating the object, which will ultimately cause dCache to shutdown during the startup procedule.  This follows the fail-fast principle.

Adding this Java code breaks compilation of the `dcache-simple` module because it uses the slf4j and Spring beans libraries, but maven does not know about these dependencies.  To fix this, update `modules/dcache-simple/pom.xml` and add the `dependencies` element underneath the `name` element.  The last lines of the file should now look like:

```
...
    <name>dCache simple service</name>

    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
    </dependency>
    </dependencies>
</project>
```

We're nearly there.  The last part is the Spring configuration that will describe how UniveralSpringCell can create the new cell.

Create the file `modules/dcache-simple/src/main/resources/org/dcache/simple/simple.xml` with the following content:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder/>

    <bean class="org.dcache.simple.Greeter" init-method="greet">
        <description>Greet the new world</description>
        <property name="message" value="${simple.greeting}"/>
    </bean>
</beans>
```

The first eight lines are boiler-plate.  The `<context:property-placeholder/>` line allows access to dCache configuration properties; for example, the value `${simple.greeting}`is replaced with the value of the `simple.greeting` configuration property.  The `bean` element describes how to build a Java object: `class` says which Java class to construct, `description` is used by Spring internally when generating human-readable output, `property` describes which setter methods to call, and `init-method` describes a method to call after the object has been properly configured.

With all these changes, we can now rebuild dCache  and see our familiar greeting in the `packages/system-test/target/dcache/var/log/dCacheDomain.log` file:

```
2017-07-27 12:49:50 Launching /usr/lib/jvm/java-8-openjdk-[...]
Listening for transport dt_socket at address: 2299
INFO  - system-test.conf:1: Property system-test.home is not a standard property
INFO  - system-test.conf:160: Property frontend.net.internal is not a standard property
27 Jul 2017 12:49:53 (System) [] ZooKeeper connection to localhost/127.0.0.1:2181 failed (Connection refused), attempting reconnect.
27 Jul 2017 12:49:53 (System) [] ZooKeeper connection to localhost/0:0:0:0:0:0:0:1:2181 failed (Connection refused), attempting reconnect.
27 Jul 2017 12:49:53 (simple) [] Hello, world!
```

There is a difference from earlier, though!  The output now has `simple` in parentheses, rather than `System`.  This is because the log entry is generated by our cell, which is called `simple` by default.

## The admin interface: on-line inspection and configuration

The admin interface is a powerful mechanism within dCache for adjusting a live system.

## Logging: for admins and developers

Here we have a look at adding some basic logging and see how to get the most from it.

## Bringing users into the fore: adding network ports

So far, the service hasn't had any users.  This changes as we add the possibility of users interacting with our service.

## dCache communication 101

dCache is a heterogeneous service with different cells sending messages to synchronise behaviour.  In this section we look some of the possibilities.

## Logging in with gPlazma

Following on from the previous section, here we add the ability to authenticate our users and discover information about them.

## Handling the namespace

In this section adds some basic interaction with the namespace.

## Handling content: pools and movers

A brief introduction on how content is handled and how different protocols are supported.

## 

## 



