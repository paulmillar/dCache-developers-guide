Releasing a new dCache Series
=============================

This document describes the organisational and technical aspects of releasing a
new dCache series. Our current (as of 2019) release model is \*time boxed\*,
with a new series being released every three months. See the [release
graph](https://www.dcache.org/downloads/1.9/timeline-dCache.svg).


Three weeks prior to the release date: Prepare Infrastructure
-------------------------------------------------------------

This part is very much specific to the DESY infrastructure.

### Setting up the release machine

1.  Contact it-virtualization and request a new Xen VM. Generally, we use 4
    cores, 8 GB RAM, 40 GB disk, CentOS latest.

2.  Set up the machine as follows, taking care to use the latest software
    versions available at the time of setup:

3.  Verify EPEL repo is enabled on the machine.

    Install necessary standard software:

	    # yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel openssh-clients httpd-tools tcpdump wget redhat-lsb-core unzip git rpm-build

    ... as well as diagnostics / admin tools:

	    # yum install ncdu htop atop

4.  Install / enable Postgres 10:

        # rpm -Uvh https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm
        # yum install postgresql10-server postgresql10

    Initialize Database using /usr/pgsql-10/bin/postgresql-10-setup initdb

    Edit /var/lib/pgsql/10.0/data/pg_hba.conf, check/ensure local connections are *trust*ed.

    Enable and start Postgres service

        # systemctl start postgresql-9.4.service
        # systemctl enable postgresql-9.4.service

5. Configure a grid-style X.509 infrastructure

    * copy EGI-trustanchors.repo into /etc/yum.repos.d/. From that repo, we'll need to
    
            # yum install ca_policy_igtf-classic fetch-crl && fetch-crl

    * Enable and start fetch-crl services for regular and boot-time checks:

            # systemctl start fetch-crl-cron
            # systemctl enable fetch-crl-cron
            # systemctl enable fetch-crl-boot

    * Install the IGTF CA bundle

            # yum install ca_policy_igtf-classic

    * Install dCache CA:

            # yum localinstall ca_dCacheORG-2.2-1.sl6.rpm
	    
    * Create Hostkeys / Hostcert using GridKA's interface or locally, using autoca:

            #wget https://raw.githubusercontent.com/kofemann/autoca/master/pyclient/autoca-client
            #chmod +x autoca-client
            #autoca-client -n -k /etc/grid-security/hostkey.pem -c /etc/grid-security/hostcert.pem https://dcache-infra03:8083/

    * Ensure that

            # chown dcache:dcache /etc/grid-security/host\*
            # chmod 600 /etc/grid-security/hostkey.pem
            # chmod 644 /etc/grid-security/hostcert.pem

    * Mark local VOMS server as trusted:

            # mkdir -p /etc/grid-security/vomsdir/desy

    * create a file /etc/grid-security/vomsdir/desy/grid-voms.desy.de.lsc with the following content:

            /C=DE/O=GermanGrid/OU=DESY/CN=host/grid-voms.desy.de
            /C=DE/O=GermanGrid/CN=GridKa-CA

6. Install jenkins-slave

	<small>nb: there is no formal release process for the jenkins-slave yet. Ask Jürgen or Paul how to get the latest binaries</small>

 	Copy local RPMs dir from a suitable machine in the cluster

	    # yum localinstall jenkins-slave-0.21-1.noarch.rpm

	Edit /etc/jenkins-slave.conf, setting JENKINS_USERS to "jenkins root" 

	    # chkconfig jenkins-slave on
        # service jenkins-slave start

7. Provide JaCoCo Agent at /usr/share/java/jacocoagent.jar (source:http://www.jacoco.org/jacoco/trunk/index.html)

8. Clone the latest series’ jobs in Jenkins.
 

### Prepare release notes

In our release-notes repo:

1.  Copy the release note skeleton from an older release.

2.  Create sections for every single *service* (i.e. file in
    dcache/skel/share/services)

3.  Distribute work: Have every person on the team research and fill two or
    three sections

4.  Remove empty sections

 

### Prepare upgrade guide

Create upgrade guide on the website. Ask around the team for any upgrade steps
that require manual intervention from the admins.

 

Two weeks prior to release: Feature freeze and testing period
-------------------------------------------------------------

At this point, we create a new branch and tag the current state of master to
become the new series. Afterwards, that release gets built as a release
candidate and tested on the developers’ home institutes’ test clusters.

Branching and tagging
---------------------

1.  Use the Maven Release plugin to change version numbers in preparation for
    (in this example) 5.0:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    mvn release:branch -DbranchName=5.0 -DdryRun=true -DdevelopmentVersion=5.1.0-SNAPSHOT
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    That should result in an unchanged (!) pom.xml, a pom.xml.branch with the
    upcoming branch version number and a pom.xml.next with the upcoming
    development version number.

2.  Make the changes permanent using

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    mvn branch:clean && 
    mvn release:branch -DbranchName=5.0 -DdevelopmentVersion=5.1.0-SNAPSHOT
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    This creates the new branch, but does not yet create a tag. Check the git
    status / diff, commit and push upstream.

3.  Now change to the release-utils directory and edit bin/tag-releases so that
    the script supports the new series, if necessary. Push changes to the
    release-utils repo upstream.

4.  *CAVEAT*: Commit 3bbe49e032 introduces a bug with the docker image. We
    currently revert the commit *immediately after creating a new branch* in
    order to avoid the following error:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    [INFO] [ERROR] Failed to execute goal on project dcache-docker: Could not resolve dependencies for project org.dcache:dcache-docker:pom:5.0.0: Could not find artifact org.dcache:dcache-tar:jar:5.0.0 in dcache.repository (https://download.dcache.org/nexus/content/groups/public), try downloading from https://download.dcache.org/nexus/content/repositories/releases/ -> [Help 1]
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    This is an ugly pseudo-fix, only required until we understand why this
    happens on new series.

5.  Change to the dCache repo, make sure you are on the upcoming branch, 
    `git pull` once to ensure everything is local. Tag the release and trigger
    package building like this:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ../release-utils/bin/tag-releases 5.0
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    (expect a prompt requiring you to do a `git clean -df` here!)

### Testing period

Distribute the RC rpm to participating institutes, asking them to test it.
During this time (two weeks to one week before the release date), the only pull
requests accepted are error corrections and last-minute fixes. Work of the
entire team should focus on quality control of the RC during that time.

 

One week prior to release: Finish testing and release notes
-----------------------------------------------------------

The last week before the release is dedicated to any last-minute fixes that may
become necessary in the light of the tests done during the previous week.

### Final update to the release notes

1.  Create list of changes using 

        git log 5.1..5.2 --no-merges --format='[%h](https://github.com/dcache/dcache/commit/%H)%n:   %s%n'

    Push the release notes upstream.

    Two tricks: Get a list of all contributors for the acknowledgements list:
  
        git shortlog 5.1..5.2 --no-merges
    
    Find out what changed in a given service:

        git log 5.1..5.2 --no-merges -i --grep="pinmanager"  
  
Verify that the release notes are complete, correct and releaseable.
 

Releasing and Publishing
------------------------

1.  Verify that Jenkins created the correct packages on the ftp server

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    /pnfs/desy.de/desy/dcache.org/5.0/
        srm-client-5.0.0.tar.gz
        dcache-5.0.0-1_all.deb
        dcache-srmclient-5.0.0-1.noarch.rpm
        dcache-5.0.0.tar.gz
        dcache-5.0.0-1.noarch.rpm
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

2.  Update the releases.xml file manually to include a new \<series\> tag
    containing the following:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    <series id="server-5.0">
      <title>server 5.0.x</title>

      <description>
        <p>dCache 5.0 is a Feature Release [...]</p>
        <ul>
          <li>Major feature</li>
        </ul>
        <p>dCache v5.0 requires a JVM supporting Java 8 or Java 11.</p>
      </description>

      <releases>
        <name>dCache VERSION</name>
        <download-url-prefix>repo/5.0/dcache</download-url-prefix>
        <notes-url>release-notes-5.0.shtml</notes-url>
        <version-prefix>5.0.</version-prefix>
      </releases>

    </series>
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

3.  cd to release-notes dir and run `update-releases 5.0.0` like normally

4.  Manually edit the website's headers to include the new book issue.

5.  Ask Paul to update the release graph

6.  Send mail to users announcing the new release as well as discontinuing old
    releases
