## Introduction ##

Welcome! This guide will show you how to obtain the latest version of the Asuswrt-Merlin source code, along with a stable version of the code for patching and compilation.

> **HERE BE DRAGONS!** There are risks when compiling your own firmware. You do so at your own risk, and your compiled firmware cannot be supported by the firmware author should something go wrong. You should be aware and familiar with the Asus recovery utilities and how to access and flash firmware from recovery mode. Your firmware may not build, it may not flash, it may flash but cause the router but not boot, it may flash and boot but work incorrectly or in unexpected ways, your computer may spontaneously combust, you may spontaneously combust, or the world may end.

## Prerequesites ##

**Note: this guide assumes a basic working knowledge of a Linux distribution. You should know how to install and access your Linux distribution, access the terminal, and install packages from your distribution's package manager.**

**Note: this guide does not cover any prerequesites for building the firmware - extra packages may need to be installed depending on your distribution. Refer to the guides on building the firmware for any extra packages you may need.**

To download the source code for compilation purposes, you will need the following:

* A Linux-based operating environment (either a dedicated machine running Linux, or a virtual machine using [VMware](http://www.vmware.com) or [VirtualBox](https://www.virtualbox.org/)).
* git distributed version control system (usually available in the package manager of your Linux distribution)
* At least 1GB free hard drive space

## Downloading the Source Code ##

Open the terminal and `cd` to a directory of your choosing. Note that you don't need to create a new directory for Asuswrt-Merlin, as cloning the repository via git will create a directory for you.

Now, clone the repository: `git clone git://github.com/RMerl/asuswrt-merlin.git`.

*This step will take some time depending on your computer's internet connection. About 500MB of data will be downloaded. Take a break, grab a cup of coffee, stretch your legs out.*

Here's what the output of this command will look like:

```
build@asuswrt-merlin:~$ git://github.com/RMerl/asuswrt-merlin.git
Cloning into asuswrt-merlin...
remote: Counting objects: 76989, done.
remote: Compressing objects: 100% (54069/54069), done.
remote: Total 76989 (delta 20537), reused 76676 (delta 20231)
Receiving objects: 100% (76989/76989), 483.79 MiB | 3.10 MiB/s, done.
Resolving objects: 100% (20537/20537), done.
build@asuswrt-merlin:~$
```

Now, `cd` into the newly-created asuswrt-merlin directory: `cd asuswrt-merlin`

```
build@asuswrt-merlin:~$ cd asuswrt-merlin/
build@asuswrt-merlin:~/asuswrt-merlin$
```

At this point, we have cloned the *master* branch of the GitHub repository. The master branch contains the latest and greatest code that has been committed to git and pushed to GitHub. If you wish to work with this code, you may stop here and continue on to building the firmware.

**Note:** If you choose to build from the master branch, please keep in mind that work in progress features and code merges may be included at any given time. This means that the firmware may not build successfully, or the compiled firmware images may not work as expected if at all.

However, if you wish to work with and build from a more stable codebase - continue on!

Starting with the release of 3.0.0.4.270.24 BETA 3, releases on GitHub are 'tagged'. Tags are points in history considered important (releases). We can switch to tags to work with the codebase as it was at that point in time.

We can use git to tell us what tags are available: `git tag -l`

```
build@asuswrt-merlin:~/asuswrt-merlin$ git tag -l
3.0.0.4.270.24-BETA3
build@asuswrt-merlin:~/asuswrt-merlin$
```

Only build 3.0.0.4.270.24-BETA3 is available at the time of writing, so let's switch to that point in time. You should substitute 3.0.0.4.270.24-BETA3 in the following commands for the tag you wish to use: `git checkout 3.0.0.4.270.24-BETA3`

```
build@asuswrt-merlin:~/asuswrt-merlin$ git checkout 3.0.0.4.270.24-BETA3
Note: checking out '3.0.0.4.270.24-BETA3'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 3dc2b4c... populateCache() now handled by the lease XML parsing
build@asuswrt-merlin:~/asuswrt-merlin$
```

*Don't worry about the long message git gives in response to that command. It's simply telling you that you've gone back in time!*

At this point, you're now at the tag you'd like to work with, and can continue to patching and/or building the firmware.

We can switch back to the master branch at any time if we'd prefer to work with that instead: `git checkout master` (remember that the master branch is the latest and greatest code!)

```
build@asuswrt-merlin:~/asuswrt-merlin$ git checkout master
Previous HEAD position was 3dc2b4c... populateCache() now handled by the lease XML parsing
Switched to branch 'master'
build@asuswrt-merlin:~/asuswrt-merlin$
```

## Updating your local repository ##

As RMerlin continues to improve and update Asuswrt-Merlin, the code you previously downloaded will quickly become out of date. But don't worry! You don't have to do the entire cloning process all over again! Just use `git pull`:

*If you have made changes to the source code and then run `git pull`, you may find merge conflicts if the master branch has changes to a file you changed as well. These merge conflicts are out of the scope of this documentation. You can find documentation on how to resolve merge conflicts in the [official git documentation](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html#resolving-a-merge).*

```
build@asuswrt-merlin:~/asuswrt-merlin$ git pull
remote: Counting objects: 329, done.
remote: Compressing objects: 100% (258/258), done.
remote: Total 318 (delta 60), reused 312 (delta 54)
Receiving objects: 100% (318/318), 469.80 KiB | 455 KiB/s, done.
Resolving deltas: 100% (60/60), completed with 11 local objects.
From git://github.com/RMerl/asuswrt-merlin
 * [new branch]      nfs        -> origin/nfs
Already up-to-date.
build@asuswrt-merlin:~/asuswrt-merlin$
```

Now you're up to date! In this particular case, a new branch called 'nfs' was created between the time we cloned the repository and the time we performed our first `git pull`. You could switch to this branch if you'd like (`git checkout nfs`), and any new tags will also be retrieved from the repository during the `git pull` operation.

If your repository is already up to date with the repository on GitHub when you run `git pull`, git will simply tell you you're already up to date.

```
build@asuswrt-merlin:~/asuswrt-merlin$ git pull
Already up-to-date.
build@asuswrt-merlin:~/asuswrt-merlin$
```

## Conclusion ##

That's it! This guide should have helped you in obtaining the latest source code for Asuswrt-Merlin, as well as working with the code from a release. You can now refer to the other guides on the wiki for building the source code!