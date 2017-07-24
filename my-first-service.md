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

Next, update our service's batch file \(\) to the following content:

```
#
#      Simple service
#

onerror shutdown

check -strong simple.greeting

say -level=fsay ${simple.greeting}
```

The `onerror shutdown`mean that dCache will stop during the start-up process if there is an error.  The `check` command returns with an error if the argument \(`simple.greeting`\) has not been defined; the `-strong` option means the command also fails if the value is empty.

Rebuilding system test \(`mvn -am -pl packages/system-test clean package -DskipTests -Pstart`\) yields the expected greeting:

```
2017-07-24 14:08:31 Launching /usr/lib/jvm/java-8-openjdk-[...]
Listening for transport dt_socket at address: 2299
INFO  - system-test.conf:1: Property system-test.home is not a standard property
INFO  - system-test.conf:160: Property frontend.net.internal is not a standard property
24 Jul 2017 14:08:33 (System) [] ZooKeeper connection to localhost/127.0.0.1:2181 failed (Connection refused), attempting reconnect.
24 Jul 2017 14:08:33 (System) [] Hello, world!
```

The difference is that we can now change this greeting without changing the batch file.

Edit the layout file in system-test \(packages/system-test/target/dcache/etc/layouts/system-test.conf\) and add the extra line immediately after the `[dCacheDomain/simple]` line:

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

Take care **NOT** to rebuild system-test after adding the `simple.greeting = Aloha Honua!` line to system-test.conf as you will loose that line.  If you want your changes to survive system-test being rebuild, modify `packages/system-test/src/main/skel/etc/layouts/system-test.conf`; however, then the changes only take affect after system test is rebuilt.

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

So far, we haven't used any Java code.  This changes now, as we start creating some simple code that dCache will run as part of a cell.

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



