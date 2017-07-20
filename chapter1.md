# Getting your feet wet: first steps with dCache

In this chapter, we identify some first steps in getting a working dCache, including obtaining the source code, compiling and running dCache in a test environment and some steps to verify that dCache is working correctly.

# Obtaining the source-code

We currently use git as our version control system and github service \(www.github.com\).  The source code is freely available, so anyone can download it.  Perhaps the easiest way of doing this is to use the git client directly \(the `git clone` command\) with the anonymous access URL.

Here's an example:

```
paul@celebrimbor:~/git$ git clone https://github.com/dCache/dcache dCache
Cloning into 'dCache'...
remote: Counting objects: 362938, done.
remote: Compressing objects: 100% (128/128), done.
remote: Total 362938 (delta 59), reused 107 (delta 20), pack-reused 362759
Receiving objects: 100% (362938/362938), 353.80 MiB | 25.21 MiB/s, done.
Resolving deltas: 100% (198554/198554), done.
Checking connectivity... done.
paul@celebrimbor:~/git$ 
```

To compile the code you will need a Java compiler and Maven.  Later chapters will go into the structure and layout of dCache code, but for now we just point out the major areas:

```
paul@celebrimbor:~/git$ ls -l dCache
total 88
drwxr-xr-x  3 paul paul  4096 Jul 20 16:31 archetypes
-rw-r--r--  1 paul paul  4492 Jul 20 16:31 BUILDING.md
drwxr-xr-x 48 paul paul  4096 Jul 20 16:31 modules
drwxr-xr-x  5 paul paul  4096 Jul 20 16:31 packages
drwxr-xr-x  3 paul paul  4096 Jul 20 16:31 plugins
-rw-r--r--  1 paul paul 54609 Jul 20 16:31 pom.xml
-rw-r--r--  1 paul paul  3716 Jul 20 16:31 README.md
drwxr-xr-x 10 paul paul  4096 Jul 20 16:31 skel
paul@celebrimbor:~/git$ 
```

There are five directories:

* `archetypes` holds coding templates, called archetypes, to make it easier for people to write plugins for dCache.
* `modules` holds the bulk of dCache code.  It is further subdivided into individual maven modules.
* `packages` holds various maven modules that are responsible for assembling dCache files for distribution.
* `plugins` holds various maven modules that compiles and is available via dCache plugin mechanisms.
* `skel` holds files that are copied into dCache packages with minimal changes.

Additionally, there are three files: the `pom.xml` provides maven with its initial set of instructions, and `BUILDING.md` and `README.md` are simple documentation files written in mark-down.

## Building and running dCache test instance

To obtain a complete dCache that may be run, you need to build one of the modules from the `packages` directory.  In this chapter, we will focus on system-test as this is the probably the quickest and easiest way to see a running dCache.

To build system-test, run a maven command like `mvn -am -pl packages/system-test clean package`.  The `clean` target is not needed for the first build, but should included subsequently to avoid stale state. There are also a couple of useful additional options: `-DskipTests` tells maven not to run the unit tests, which speeds up the process and `-Pstart` tells maven to start system-test once it is built.

Here is a shortened example showing this:

```
paul@celebrimbor:~/git/dCache (master)$ mvn -am -pl packages/system-test clean package -DskipTests -Pstart
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] Logback config for building and testing
[INFO] dCache parent
[INFO] Logback config for console output
[INFO] dCache common
[INFO] CLI framework
[INFO] dCache common authentication
[INFO] cells
[INFO] gPlazma 2
[INFO] gPlazma 2 Argus plugin
[INFO] gPlazma 2 grid plugins
[INFO] gPlazma 2 Kerberos plugin
[INFO] gPlazma 2 JAAS plugin
[INFO] gPlazma 2 Network Information Service plugin
[INFO] gPlazma 2 Name Service Switch plugin
[INFO] gPlazma 2 principal ban file plugin
[INFO] gPlazma 2 VOMS plugin
[INFO] gPlazma 2 KPWD plugin
[INFO] FTP client
...
[INFO] missing-files SEMsg plugin ......................... SUCCESS [  0.079 s]
[INFO] dCache plugins ..................................... SUCCESS [  0.001 s]
[INFO] HSQLDB plugin ...................................... SUCCESS [  0.729 s]
[INFO] dCache packaging base .............................. SUCCESS [  0.002 s]
[INFO] System tests ....................................... SUCCESS [ 28.000 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:23 min
[INFO] Finished at: 2017-07-20T17:43:27+02:00
[INFO] Final Memory: 186M/1600M
[INFO] ------------------------------------------------------------------------
paul@celebrimbor:~/git/dCache (master)$ 
```

Congratulations: you should now have a work dCache instance!

Almost all the files associated with this instance are located in `packages/system-test/target/dcache`. You can verify that dCache is running by using the `dcache` script.  This script also lets you start and stop dCache, show some information about dCache instance and  perform some basic admin operations.

Here are some examples of stopping and starting dCache, and querying dCache's current status:

```
paul@celebrimbor:~/git/dCache (master)$ packages/system-test/target/dcache/bin/dcache status
DOMAIN       STATUS                  PID  USER LOG                                                                               
dCacheDomain running (for 5 minutes) 4855 paul /home/paul/git/dCache/packages/system-test/target/dcache/var/log/dCacheDomain.log 
paul@celebrimbor:~/git/dCache (master)$ packages/system-test/target/dcache/bin/dcache stop
Stopping dCacheDomain 0 1 2 3 4 5 6 7 8 9 10 11 done
paul@celebrimbor:~/git/dCache (master)$ packages/system-test/target/dcache/bin/dcache status
DOMAIN       STATUS  PID USER                          LOG                                                                               
dCacheDomain stopped     [whoever runs "dcache start"] /home/paul/git/dCache/packages/system-test/target/dcache/var/log/dCacheDomain.log 
paul@celebrimbor:~/git/dCache (master)$ packages/system-test/target/dcache/bin/dcache start
Starting dCacheDomain done
paul@celebrimbor:~/git/dCache (master)$ packages/system-test/target/dcache/bin/dcache status
DOMAIN       STATUS                   PID  USER LOG                                                                               
dCacheDomain running (for 20 seconds) 6743 paul /home/paul/git/dCache/packages/system-test/target/dcache/var/log/dCacheDomain.log 
paul@celebrimbor:~/git/dCache (master)$ 
```

Without going into details, a domain is a Java Virtual Machine \(JVM\) instance.  A node within a dCache cluster can run multiple domains \(i.e., multiple JVM instances\) that are configured in a common configuration system.  The `dcache` script can start and stop the domains independently. Here, in system-test, dCache has been configured to run a single domain, called `dCacheDomain`.

The output from dcache status mentions the location of the log file for this domain.  This file contains the redirected standard out \(stdout\) and standard error \(stderr\) from the java process.

## Familiarising yourself with dCache

So, what can you do with this system-test dCache?  Here are a few exercises...

