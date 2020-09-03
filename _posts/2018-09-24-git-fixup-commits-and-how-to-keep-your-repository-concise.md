---
layout:	"post"
title:	"Git fix-up commits and how to keep your repository concise"
---

* * *

> Have you ever wasted time looking at your repositories because of commits
that didn't add any value to the project's story? Some of the most common
normally start with "fix-up…", by avoiding these kinds of commits, your
repository will be easier to understand, because it'll focus on the story of
your project, not on smalls mistakes. Here git has to the rescue!

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_0.png)

Source: <https://git-scm.com/downloads/logos>

G[it](https://git-scm.com/) is an awesome and one the most popular Version
Control System (VCS) used around the world, it has a distributed structure,
it's fast (really fast), it has great tools like
[Bitbucket](https://bitbucket.org/) and [GitHub](https://github.com/), and
many famous projects are hosted on Git (for instance: the [Linux
kernel](https://github.com/torvalds/linux)).

There are a lot of excellent materials freely available that talks about Git,
for instance, the [Pro Git Book](https://git-scm.com/book/en/v2). Thus, I
won't focus on the basics, instead, I'll present a nice feature of Git that
can help you to avoid unnecessary commits that only pollute your code history,
adding no value to it. I'm talking about the combination of:

  1. Fix-up commits
  2. Rebase with autosquash

There aren't any mysteries regarding its usage but, unfortunately, these
features aren't well known, so I'll present them in this short article along
with an example. I hope that you enjoy!

#### A repository is a storyteller

A repository represents all the efforts that have been put on a project, like
features, bug-fixes, etc. It can be made by just one developer, by a company,
or by an entire community of fellow developers spread around the globe. Like a
typical story, a repository is a collection of facts sorted by time, every new
piece of feature represents a new fact, a discrete event, a snapshot of your
project at every point of time.

In Git, every discrete event is called _commit_ , a checkpoint on the project
that tells us that something has happened. By looking at the commits, you'll
be looking at the history of the project, how it has been changing over time.

Given that a series of commits tell us a story, and while writing a story the
writer will eventually make mistakes, we'll need some sort of "undo" feature
that will fix the mistake, but given that you haven't published the story yet,
the fix will be part of the original checkpoint and not a new one with little
or no value to the whole story. For example:

Suppose that you're working on a project, let's call it _the-coolest-project_
, specifically on the feature _the-feature_ and to implement this feature you
decided to break it on logical steps that run sequentially toward the goal,
thus you have the commits _the-feature-step1_ and _the-feature-step2_ on the
branch _the-feature_.

Hence, you type:

    
    
    git log --all --decorate --oneline --graph

And your repository can look like this:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_1.png)

But then you realized that you had done some mistake on _the-feature-step1_ ,
and you'd like to fix it, so you created another commit to fix the previous:

    
    
    git commit -m "Fix up a mistake at the-feature-step1"

Now, your repository looks like this:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_2.png)

But wait a minute! This fixing commit really doesn't make sense for the
repository's story, it's just part of a previous commit and doesn't tell us
anything more about the project than the previous commit did and shouldn't be
merged to _master_. Instead, we should squash the commit with its fix-up
before merging the branch to master.

So, what can we do now?

#### Fix-up commits and rebase with autosquash to the rescue

First, instead of creating the fix-up commit manually, let Git create it for
us!

For this, type:

    
    
    git commit --fixup 9d33fdb

Where _9d33fdb_ is the hash of the commit that you 'd like to be fixed.

Now our repository looks like this:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_3.png)

Not so different from our initial scenario, right? But be patient, things will
be clearer soon.

After you've finished the feature, you decided to push it, so your code can be
reviewed. Do you practice code review, right? You should!

But before, let's get rid of the fix-up commit that doesn't add any value to
our repository's story and shall not be merged to _master_.

Here comes rebase with the autosquash flag enabled to help us. So, Git will be
able to parse the fix-up commit and automatically squash it to the original
commit. For this, type:

    
    
    git rebase -i --autosquash master

Where:

  *  _-i_ means interactive, which will open a text editor before rebasing the commits, so you can act before (like changing order, editing messages, etc)
  *  _\--autosquash_ means that commits with _fixup!_ (or _squash!_ ) will be automatically squashed to their original commits

It'll open your text editor showing what the rebase will do, especially the
_fixup_ line, that will be already there in the right order, ready to be
squashed with the desired commit ( _the-feature-step1_ ).

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_4.png)

If everything is fine (and it is for our case), just close the text editor.

Now, your repository will look like this:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_5.png)

And the fix-up commit had been squashed with its originating commit before the
_the-feature_ branch was merged to _master_. Nice!

#### Automating: --autosquash as default

The material seen so far is enough for using _fixup_ commits and _autosquash_
, but Git allows us to go further by providing some ways to make this process
easier.

Given that interactive rebase only picks up commits with messages starting
with _fixup!_ (or _squash!_ ) and we always have the possibility to see what
Git will do by inspecting the text editor before the rebase takes place. A
possible improvement is to make _autosquash_ the default behavior for
interactive rebases.

For this purpose, you can enable the _rebase.autosquash_ flag by doing:

    
    
    git config --global rebase.autosquash true

Now, you can omit the _autosquash_ part of the rebase, like this:

    
    
    git rebase -i master

And the effect will be the same that we had previously seen. But with a
shorter syntax, cool!

#### Conclusion

Git is a fantastic tool that helps us on our daily programming tasks, making
it easier to distribute work among developers, keep track of the project's
history, etc. Some of its features are, unfortunately, not so well known by a
lot of developers.

Among these "not-well known" features, lies the combination of fix-up commits
with rebase and autosquash, which makes your repository clean of unimportant
commits, hence easier to track, and understand.

I really encourage you to give a chance to Git and explore some of its
features that can make your development process more manageable, easier and
funnier.

#### References

[1] Pro Git Book. <https://git-scm.com/book/en/v2>


***
*Originally published at https://medium.com/@rvarago*
