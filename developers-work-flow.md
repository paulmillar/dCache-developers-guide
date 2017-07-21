# The developer's work-flow

In this chapter, we follow a typical developer as he adds a new feature to dCache and fixes a bug.  This chapter gives concrete instructions to achieve the goals.  You don't need to follow these instructions exactly; however, you really need to know what you are doing if you do something different.

Throughout this chapter we assume that the developer has the official dCache repo as the `upstream` remote and the developer's own fork of dCache as the `origin` remote.  You can check this with the `git remote -v` command:

```
paul@celebrimbor:~/git/dCache (master)$ git remote -v 
origin  git@github.com:paulmillar/dcache.git (fetch)
origin  git@github.com:paulmillar/dcache.git (push)
upstream        git@github.com:dCache/dcache.git (fetch)
upstream        git@github.com:dCache/dcache.git (push)
paul@celebrimbor:~/git/dCache (master)$ 
```

If you are not a member of the dCache core team then you will not have commit access to the official dCache repo.  In this case, you can use the anonymous HTTP-based access:

```
paul@celebrimbor:~/git/dCache (master)$ git remote -v
origin  git@github.com:paulmillar/dcache.git (fetch)
origin  git@github.com:paulmillar/dcache.git (push)
upstream        https://github.com/dCache/dcache.git (fetch)
upstream        https://github.com/dCache/dcache.git (push)
paul@celebrimbor:~/git/dCache (master)$ 
```

The output from `git help remote` will hopefully let you modify your local dCache code-base to match one of these expectations.

## Some general comments

In general, developers only commit to `master` branch and only once the code as passed the code-review process.  Never push commits into any other branch in `upstream`.

Patches should be as small as possible, while achieving some admin- or user-noticable result.  Often, very large patches can be broken down into smaller patches, each of which builds on the previous patches and adds some meaningful functionality.  Large patches are harder to code-review and provide a hiding place for bugs.

Patches should not add dead code.  Code that is not used is dead-code, whether it has been commented out or whether the code is something that is indended for use in later patches. Dead-code is probematic because it isn't clear whether or not it is supposed to be enabled.  Likewise, adding dead code with a promise that some subsequent patch will make use of it is also problematic as it is difficult to code-review the complete picture.

Patches must not break dCache.  It is not allowed to commit a patch with a promise that some subsequent patch will current breakage.

Patches should have a specific focus.  They should not mix code-changes \(i.e., implementing or changing functionality\) with non-code changes \(e.g., formatting changes, white-space changes\).  This is because such a mixture makes it harder to understand the patch.

## Developing a new feature

In this section we follow the developer during the feature development process.  In general, new features are committed to the `master` branch, which will be the next version of dCache.  Therefore, the first thing is to bring the local clone up-to-date and create a branch for the development work:

```
paul@celebrimbor:~/git/dCache (master)$ git fetch upstream
paul@celebrimbor:~/git/dCache (master)$ git checkout -b development/webdav-add-mirror-option upstream/master
Branch development/webdav-add-mirror-option set up to track remote branch master from upstream.
Switched to a new branch 'development/webdav-add-mirror-option'
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ 
```

In this case, nothing has changed in the `upstream` remote, so the fetch command had no output.  The command will give some output if there has been activity from other developers committing patches, or the release-team have been busy.

The branch name follows a particular format: `development/<service>-<description>`. Although it only matters that the developer understands the branch's name, this particular format has proved useful.

The developer now updates dCache to implement this new feature.  Progress may be checked by running system-test and verifying the behaviour is working as expected.  You should also check that there are no stack-traces in the log file.

Once the new feature is working well the code may be committed.

### Initial commit message

Commit messages have a specific format, partly to achieve uniformity across many messages and partly to help the developer remember all the information that is necessary.  Here is an example commit message:

```
webdav: add support for the 'transform=mirror' option

Motivation:

Many users want the mirror-image version of an image.  Currently
they must download the image and create the mirror-immage version
of the image themselves.  This is both inconvenient and not all
users have sufficient knowledge of image manipulation software to
achieve this.

Modification:

Add support for parsing the 'transform=mirror' argument in the URL.
Add support in the webdav door for generating the mirror image
dynamically.

Future patches will address storing the generated images, likely by
using some dCache capacity as a cache.

Result:

Users can request mirror-image version of uploaded files.

Target: master
Require-notes: yes
Require-book: yes
```

There are five main sections: the title, the motivation, the modification, the result and metadata.

The title is a single line \(&lt; 80 characters\) that contains the affected service, a colon, then a very brief description of what has changed.

The motivation, modification, and result each have their own header.  The description after each title is largely free-form, but ideally each line is no longer than 80 characters.

The final section is various metadata key-value elements that provide succinct description of often used information.  In the commit before submitting to code-review, three items are present: `Target`, `Require-notes` and `Require-book`.  The Target is the git branch the developer is intending the patch.  This is almost always `master`.  The Require-notes line provides a hint to the release-team about whether the change is something that should be described in the release notes.  The Require-book line provides a hint on whether the dCache Book should be updated as a result of this patch.

### Code review

The code-review process is somewhat different, depending on whether you are a core-team developer \(with direct commit access to dCache code\), or an external contributer.

#### As core-team developer

For core-team development, we use [our ReviewBoard instance](https://rb.dcache.org/) for handling code review process.  Although a patch may be submitted directly to ReviewBoard, it's easier to use the review-board client directly:

```
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ rbt post -g -o --target-groups=all
Review request #10375 posted.

https://rb.dcache.org/r/10375/
https://rb.dcache.org/r/10375/diff/
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ 
```

The link is also opened in your web browser.  You should see your git commit message as the ReviewBoard title and description.  The Diff tab allows you to review the patch.  One you are happy with both, Submit the patch for code review.

The code-review process is described in more detail in the Code review chapter.

If, during the code review, you wish to make changes to the patch, you can either update the existing patch or add a new patch with the subsequent changes.  In either case, use the rbt tool to update the patch in ReviewBoard, using the RB request number:

```
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ rbt post -r 10375
Review request #10375 posted.

https://rb.dcache.org/r/10375/
https://rb.dcache.org/r/10375/diff/
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ 
```

Once the code-review phase is complete you can commit the patch into dCache.  To do this, you need to update the patch to include a reference to patch location and described which developers said "Ship It!" for the patch:

```
...

Target: master
Require-notes: yes
Require-book: yes
Patch: https://rb.dcache.org/r/10375/
Acked-by: Tigran Mkrtchyan
Acked-by: Gerd Behrmann
```

Then the patch may be pushed into dCache master; e.g.,

```
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ git push upstream HEAD:master
Counting objects: 13, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (13/13), done.
Writing objects: 100% (13/13), 22.37 KiB | 0 bytes/s, done.
Total 13 (delta 8), reused 0 (delta 0)
remote: Resolving deltas: 100% (8/8), completed with 8 local objects.
To git@github.com:dCache/dcache.git
   0b21d9d..2784157  HEAD -> master
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$
```

The new tip of master \(`2784157`\) is then added to ReviewBoard as an additional metadata field:

```
...

Target: master
Require-notes: yes
Require-book: yes
Committed: master@2784157
```

and the review request is closed.

#### As external developer

In general, external developers do not have access to our ReviewBoard.  Instead, we use the tools provided by github.

You must push your patch to your fork of dCache code, the `origin` remote:

```
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ git push origin HEAD
Counting objects: 575, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (468/468), done.
Writing objects: 100% (575/575), 104.90 KiB | 0 bytes/s, done.
Total 575 (delta 316), reused 79 (delta 20)
remote: Resolving deltas: 100% (316/316), completed with 91 local objects.
To git@github.com:paulmillar/dcache.git
 * [new branch]      HEAD -> development/webdav-add-mirror-option
paul@celebrimbor:~/git/dCache (development/webdav-add-mirror-option)$ 
```

Once the branch exists in your fork, you can create a pull-request into the `master` branch on dCache.

Code review will take place through this pull-request.

Once the code-review is complete, the pull-request will be accepted.

## Fixing a bug present in master and supported dCache versions

When fixing a bug, it is likely that the problem exists in the `master` branch, too.  Therefore we focus on that case, first.

The initial work is much the same as for developing a new feature: create a branch, change dCache to fix the problem, commit the change, go through code-review, commit the change to master.  The only differences \(at this stage\) is the branch name is `fix/<service>-<description>` , don't close the ReviewBoard request once the patch is committed to master, and the initial commit message should have `Request` metadata, describing into which dCache versions the patch should be back-ported; for example,

```
srm: fix problem with plain TLS connections

Motivation:

Some small number of client-connections fail during the TLS handshake
due to a race-condition when loading trusted certificates from the
trust-store.  In some clients, this results in the error message:

    FATAL: server rejected nonce, handshake NULL/NULL rejected.

Modification:

The problem is ultimately due to non-thread-safe code in an upstream library.
This patch introduces a work-around by synchronizing access to this library
but ideally the library is updated to fix this issue.

Result:

TLS handshake becomes reliable again.

Target: master
Require-notes: yes
Require-book: no
Request: 3.1
Request: 3.0
Request: 2.16
```

Once the bug is fixed in the `master`branch, you can create pull-requests for each of the supported branches.  The following procedure is not the only way of achieving this, but is likely results in the effort.

Create a branch from the current version of the branch you wish to fix, 3.1 in this example:

```
paul@celebrimbor:~/git/dCache (fix/srm-fix-tls-handshake)$ git checkout -b fix/3.1/rb10376 upstream/3.1
Branch fix/3.1/rb10376 set up to track remote branch 3.1 from upstream.
Switched to a new branch 'fix/3.1/rb10376'
paul@celebrimbor:~/git/dCache (fix/3.1/rb10376)$ 
```

Cherry-pick the patch, in this case from master:

```
paul@celebrimbor:~/git/dCache (fix/3.1/rb10376)$ git cherry-pick master
[fix/3.1/rb10376 9107534] srm: fix problem with plain TLS connections
 Date: Tue Jul 18 15:32:45 2017 +0200
 2 files changed, 45 insertions(+), 4 deletions(-)
paul@celebrimbor:~/git/dCache (fix/3.1/rb10376)$ 

```

and push this to your fork of dCache:

```
paul@celebrimbor:~/git/dCache (fix/3.1/rb10376)$ git push origin HEAD
Counting objects: 1, done.
Writing objects: 100% (1/1), 496 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:paulmillar/dcache.git
 * [new branch]      HEAD -> fix/3.1/rb10376
paul@celebrimbor:~/git/dCache (fix/3.1/rb10376)$ 
```

You can use the github web interface to create the pull request.  Note that the target branch \(`3.1` in this example\), is in the origin branch's name \(`fix/3.1/rb10376`\).  This makes creating the pull-request easier.

Once the pull-request is created, the ReviewBoard description is updated by placing the pull-request URL in as metadata; e.g.,

```
Target: master
Require-notes: yes
Require-book: no
Request: 3.1
Request: 3.0
Request: 2.16
Committed: master@46a38e
Pull-request: https://github.com/dCache/dcache/pull/3357
```

The process then repeats itself for each supported branch.  You may find it easier to cherry-pick from the previous pull-request branch, rather than always from master.  This is in case there were conflicts during the cherry-pick that needed resolving.

Once all the pull-requests are sent, you can close the ReviewBoard request.

## Fixing a bug that is not present in master

Fixing a bug that is present in a supported dCache version, but not present in the `master` branch is similar to when the bug is in master, except you do not commit the patch to master.  As a consequence, the commit message has no `Target` metadata and the ReviewBoard entry has no `Committed` metadata. Once the patch passes code-review, pull requests are created using the currently uncommitted patch.





