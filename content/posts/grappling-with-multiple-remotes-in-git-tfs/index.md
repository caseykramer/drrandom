---
url: "/2011/11/16/grappling-with-multiple-remotes-in-git-tfs/"
title: "Grappling with multiple remotes in git-tfs"
date: 2011-11-16T06:53:00.000Z
slug: "grappling-with-multiple-remotes-in-git-tfs"
tags: 
  - "git"
  - "tips"
  - "tools"
  - "version-control"
---

If you happen to be one of the many people in the unfortunate situation to be stuck working with TFS source control on a daily basis and gaze longingly at the folks using Git or Mercurial wishing you could have some of that distributed goodness for your very own self, I am here to tell you that all is not lost.  There are a couple ways you can work with a distributed version control system along side TFS and try and reduce the pain associated with TFS.  One way I wrote about here as an answer to a [question on StackOverflow](http://stackoverflow.com/questions/2331636/real-world-use-of-mercurial-with-a-team-foundation-server/2351734#2351734).  This technique worked fairly well for me dealing with a small codebase with only a few branches.  However, it became unmanageable once I started working in an environment which had a large TFS repo with several different branches that I needed to switch between on a regular basis.  You can read about some of the issues I ran into within the updated section of the answer, but overall things got messy quickly.

The solution I have arrived at, and one which seems to be working out reasonably well so far, is to use Git and [git-tfs](https://github.com/spraints/git-tfs) rather than Mercurial.  This could actually apply equally to Mercurial if such a thing as hg-tfs existed, but alas no such animal can be found.

#### Quick introduction to git-tfs

The [git-tfs project](https://github.com/spraints/git-tfs) can be found out on Github (of course), and is based on the also very awesome git-svn project.  It takes advantage of the fact that git allows for custom commands by looking for anything on the path that has the form `git-<command>` and executing it when you type in `git <command>`.  The project is a C# project that compiles into an EXE named `git-tfs.exe`.  You drop this in your path and you’re off to the races.

Under the covers git-tfs will tag commits that come from TFS with the repository information and changeset ID, and then when you want to put your changes into TFS (as a shelfset or a commit) it creates a new workspace, adds changes not associated with a changeset from your git repo, and then when it is all done the workspace gets deleted.

Rather than go through the details of getting git-tfs installed and your repository set up I’m going to refer you to the [documentation](https://github.com/spraints/git-tfs/wiki), which is not too bad, and get to the more involved scenario that this post is all about.

#### TFS Branches and Remotes

If you take a look at the config information on your git-tfs repo you are likely to see a section that looks something like

```
[tfs-remote "default"]
    url = http://mytfsserver:8080/tfs
    repository = $/TFS/repo
    fetch = refs/remotes/default/master
```

As you might expect this defines your TFS server and branch information. You can edit your configuration and add as many of these as you want, as long as they have unique fetch paths. The problem is that there really isn't any documentation that talks about how this works, and so that is what I'm hoping to illuminate. For starters, I’m going to assume you have just the “default” tfs-remote configured.

#### Working with a new branch

The first thing to do is to create a new remote in your git config.  As you might guess that simply means creating a new `tfs-remote` section with a unique name.  Lets add something like:

```
[tfs-remote "release"]
    url = http://mytfsserver:8080/tfs
    repository = $/TFS/release
    fetch = refs/remotes/release/master
```

This creates a new remote name "release". The next step is to set up a git branch for this TFS branch. To do that simply create the branch the way you would any git branch:

```
git checkout -b release
```

Now comes the good part, we want to pull in the changes from our TFS branch into this git branch.   Git-tfs includes a “pull” command, but it has a fairly big limitation when working in this particular scenario.  It does not allow you to specify a merge strategy.  That means that there is no way to say “pull in everything and use the versions of the files on TFS if there is a conflict”, which is what we want to do.  To accomplish this, we’ll want to deconstruct the “pull” command into it’s discrete steps.  The first step is to do a get-tfs fetch against the new remote.  You do that using this command:

```
git tfs fetch -i release
```

This will pull in all of the changesets from the TFS branch and apply them to the object tree that sits under the covers in git. Once that is done we'll want to merge those changes into our git branch by doing:

```
git merge -X theirs refs/remotes/tfs/release
```

You can actually specify whichever merge strategy you want, but this is the one I generally choose (merge recursive, take the remote changes for any conflicts). Pay special attention to where we are merging from. This is the remote location that points to the HEAD of the commit tree that we just pulled from TFS. The general format of this is: `refs/remotes/tfs/<tfs-remote name>` Once you have done the initial fetch/merge you will have better luck using the get-tfs pull command, though you can still run into merge conflicts if there are a lot of changes between pulls.

The way I have been working, and a way that seems to work well, is to have a primary branch that mirrors each of your TFS branches that you may be working in, with topic branches created from there. That gives you at least one git branch for every TFS branch that is "pure", and that you can potentially mess up without feeling too bad, since you can always fetch everything from TFS again (this assumes you have some time to kill, since fetching a lot of changes from TFS takes a while).

By the way, when you’re working with multiple TFS branches in git-tfs, the `–i <remote-name>` switch will become your constant companion.  You will need to use it on any command which may have an ambiguous parent….meaning when git-tfs goes through you’re history to find out which TFS remote to use, it’s going to look at those git-tfs-id tags on the commits, and if there is more than one source it will ask you which one you want.  When you have several branches you’ve created that represent different TFS branches, you are most likely going to have the history of the original branch lurking somewhere, so you will be needing the switch.

#### Committing your work back to TFS

Now that you have branches in git you can work with, you can use whatever topic-branching strategy makes sense when your doing your day-to-day work.  Once you are ready to commit things back to TFS, the simplest way to do that is to check your changes into git, and then issue a git-tfs check-in command (There are three, “checkin”, '”rcheckin” and “ct”.  I personally like the “ct” command, which brings up the TFS dialog that lists changes and allows you to specify commit messages).  When you do this, git-tfs will look through your commit history, and find the last commit with a git-tfs-id tag, and then take every commit from that point and apply it to a workspace that it creates behind the scenes. Once you check-in git-tfs then does a pull/merge from TFS (to get any changes since your last fetch), and leaves your branch with your commit on top.

Notice I said that it goes through the commit history, and looks for the _first_ commit with a git-tfs-id tag?  This is important if your working in a scenario where you have a reasonably large change you’re working on, and you are doing periodic tfs fetches and merges into your topic branch to keep up to date with what your colleges are doing. 

Lets say you’ve been working on something for 5 hours, doing periodic commits as your go, and one of your teammates tells you they’ve made a change to one of the common libraries that you are using.  At this point you’ll want to switch back to your TFS tracking branch, get the latest changes, and them merge those in to your working branch.  Doing this, though, means that git-tfs is not going to see the changes you made over the last 5 hours.  So what do you do?

You remember that you’re using git :).  Lets continue with our scenario and say you work for another two hours and complete your feature making commits along the way.  Now, you’re ready to check in to TFS.  What I’ve found works best for me is to go back to my TFS tracking branch, make sure it’s up to date, and then create a _new_ branch for my pending check-in.  Then I do a merge into the new branch from the topic branch I had been working in, and throw a –squash on there (this is not strictly necessary, but in my environment it’s handy to have a change in a single changeset that is associated with a single change request, that way roll-back is easier).  Once the merge is done, I can issue my `git tfs ct –i release` command and all is good.  I’ve got all of my changes in a nice neat little package.

Another option available is to use the git-tfs rcheckin command, which essentially does a rebase against your TFS branch.  I’ve actually not done this, since it doesn’t fit in with how changes are handled in my organization.

#### What about Shelf-Sets?

I’m glad you asked.  We actually use shelf-sets for code-reviews (which are required before code goes to production).  Fortunately git-tfs has nice support for shelfsets. For starters un-shelving changes puts them in a topic branch, which is exactly what I would hope for.  To do this use the following command:

```
git tfs unshelve -i <remote> [-u <username>] <shelfset name> <branch>
```

(Just a quick note, the master branch of the git-tfs github repo does not contain support for the –I argument at this point.  I’ve got a pull-request in place to add it, since it was just a one-liner.  You can track it [here](https://github.com/spraints/git-tfs/pull/105) if you need to.  You can also grab my fork of the git-tfs project [here](https://github.com/caseykramer/git-tfs) (there is a branch called UnshelveTweak that has the change) Looks like the pull request has been merged into the main git-tfs project.  If you clone/fork and do a build you should be good to go)

Hopefully this is pretty self-explanatory. The `-u` option is not required if your pulling from one of your own shelfsets. If you’re shelfset has a name with spaces in it, you should surround it with double quotes.

Shelving changes is equally easy, you just issue the shelve command:

```
git tfs shelve -i <remote> <shelfset name>
```

Again, if you've got a shelfset name with spaces be sure to put quotes around it.

#### A few more tips

I’ve found on occasion that things get a little out of whack when shelving or committing changes back to TFS, and some files are included that shouldn’t be.  This usually happens when things are not merged 100% after a git tfs fetch, and there are some files that need to be committed separately.  So how do you deal with this?  Well, you rely on our friend the get-tfs-id tag.  The simplest process is to do a `git log` to see your list of changes, and copy the line that starts with `git-tfs-id:`. Then, if you've not committed your changes yet, add that to a new line at the end of your commit message. If you have committed you can use the `git commit --amend` command to update the last commit. Once you've done this you will only get new changes checked in or shelved to TFS

Another useful bit of kit is a script I whipped together that sets up a git bash shell with all of the .Net/Visual Studio 2010 tools on the path, so I can build my projects from within the git-tfs bash shell. The script looks like (I’m running a 64-bit OS, so you may need to adjust the paths):

```
@echo off
call "C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\vcvarsall.bat" x86
"C:\Program Files (x86)\Git\bin\sh.exe" --login -i
```

I use the most excellent [Console2](http://sourceforge.net/projects/console/) project, so I can just save that script to a .bat or .cmd file and add a new tab that has that file as the shell.  Now I can see which branch I’m working in while on the command line, and still build everything.

So there you have it.  It’s not 100% hassle free, but it is a lot less hassle than dealing with TFS all of the time.  I probably save several hours a week simply by not needing to wait on TFS fetches and branch changes.
