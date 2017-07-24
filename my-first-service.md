# My first service

This chapter provides a tutorial in which we will build a simple service, step by step.  Each step builds on the previous ones and introduces a new aspect of dCache.  Together, this provides a whistle-stop tour of the different aspects of dCache you are likely to encounter as a developer.

## A service to say Hello World!

In this section we introduce services, batch files, dCache layout configuration and the domain log file.

A node or computer within the dCache cluster runs one or more Java Virtual Machines \(JVMs\), called domains.  Each domain \(i.e., each JVM\) may be started or stopped independently using the `dcache` script.  The `dcache start` and `dcache stop` commands take optional arguments: the names of the domains to start or stop \(respectively\).  Although it is common to end domain names with `Domain`\(e.g., `AdminDomain`\), there's no need to do so.

Each domain can do any amount of work.  In system-test, for example, everything is placed in a single domain called `dCacheDomain`.  In production environments, the functionality of dCache is split over many domains.  Each node in the dCache cluster must have at least one domain, so that it can participate in the dCache cluster.  Nodes can also have multiple domains.

Services are the smallest unit of deployable work with each domain hosts one or more service.  Each service has a type, which describes what kind of work it does.  For example a `pool` service is resposible for storing data, a `webdav` service provides access to dCache via HTTP and WebDAV protocols, and a `pnfsmanager` service is resposible for dCache interacting with the namespace.

The layout file is resposible for describing which domains should run on a node and which services are hosted by each domain.  Domains are defined by a line containing their name in square brackets; e.g.,` [dCacheDomain]` declares that the node should run a domain called `dCacheDomain`.  A service is declared by placing the service type after the domain name and a slash all within square brackets; e.g.,` [dCacheDomain/pool]` describes a `pool` service that should run within the `dCacheDomain` domain.  Service declarations must come after the domain declaration.  The layout file can also contain configuration \(lines that contain a `=`\), but we'll come to that in the next section.

Ultimately, a service is a list of operations to establish that particular service, written in a language called batch script.  Each service has its own batch script, each with the service name and `.batch` as the file name; for example, `pool.batch` is the batch file for the `pool` service.  The batch files are copied directly into whichever package is being build and are located in the `services` directory.  The exact location of this directory varies for different package types; for FHS compliant deployments, the directory is `/usr/share/dcache/services`.  Within the git repo, this directory is `skel/share/services`.

To create our new service, we must create a file `simple.batch` in the `skel/share/services` directory with the following content:

```
say -level=fsay Hello World!
```

In batch script language, the `say` command logs some information, with the arguments providing the message to be logged.  The `level` option describes at which log-level the message will be logged; `fsay` means the message is to be logged at error-level.  This is needed so the message will appear in the domain log file.

We also need to update system-test so that it runs our new service.  The source for the layout file for system-test is `packages/system-test/src/main/skel/etc/layouts/system-test.conf`.  Add the line `[dCacheDomain/simple]` immediately before some other `[dCacheDomain/<service>]` line; for example,

```
... some configuration here ...

[dCacheDomain]
# The following is defined for the domain to prevent that the CLI
# applications enable the debugging options.
dcache.java.options.extra=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=localhost:2299,suspend=n -XX:+TieredCompilation

[dCacheDomain/simple]
[dCacheDomain/admin]
admin.paths.history=${system-test.home}/var/admin/history

[dCacheDomain/zookeeper]
```

In this example, the `[dCacheDomain/simple]` line is the first service defined in the `dCacheDomain` domain, immediately before the `admin` service.

When starting a domain, dCache will start each service in the order listed in the layout file.  If our` simple` service as the first service,  it will be the first service to be started.

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
* `Hello World!` the message from the `simple` service.

## Being flexible: dCache configuration

Here, we start making our service more flexible by allowing administrators to modify how it behaves.

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



