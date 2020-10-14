---
layout: "post"
title:  "Git fix-up Commits: How to Keep your Repository Concise"
tags:   git
---

> Have you ever wasted time looking at your repositories because of commits that didn't add any value to the project's story? Some of the most common start with "fix-up…". By avoiding these commits, the repository tends to become easier to understand, as we can then focus on the bigger story that the project is trying to tell, not small details.

* * *

|![Git.](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_0.png)|
|:--:| 
| *Git. Source: <https://git-scm.com/downloads/logos>.*|

[Git](https://git-scm.com/) is an awesome tool and one the most popular Version
Control System (VCS) ever, it has a distributed structure, it's fast, it's used by popular platforms like [Bitbucket](https://bitbucket.org/) and [GitHub](https://github.com/), and plenty of famous projects are tracked with Git (e.g. the [Linux kernel](https://github.com/torvalds/linux)).

There are a lot of excellent materials freely available that talks about Git, for instance, [Pro Git Book](https://git-scm.com/book/en/v2). Thus, I won't focus on the basics, and instead, I'll briefly describe a little nice feature provided by Git that might help you to avoid unnecessary commits, which pollute the code history. It's a combination of:

  1. Fix-up commits.
  2. Interactive rebase with autosquash.

There aren't mysteries regarding their usage. However, in my experience, these features aren't well-known, so I'll try to present them in this short post along with an example. I hope that you enjoy it!

# A repository tells a story

A repository contains the efforts that have been put on a project: features, bug-fixes, mistakes, etc. It could be managed by a single developer, by a company, or by an entire community of fellow developers spread all around the globe. As a typical story, a repository collects facts, sorted by time. And every new piece of is such a fact, a discrete event, a snapshot of the project.

In Git, every discrete event is reified as a _commit_, a checkpoint that tells us that something has happened and worth taking note. By looking at the commits, we are looking at the history of the project and how it has changed over time.

Given that a series of commits tell us a story, and while writing a story the writer eventually makes mistakes, we'll need to "undo" some features to fix these mistake. However, since we haven't published the story yet, the fix will be part of the original checkpoint and not a new one, with little or no value to the whole story.

Suppose that you're working on a project, let's call it _the-coolest-project_, specifically on the feature _the-feature_. To implement _the-feature_, you've decided to break it in a sequence of logical steps, thus you have commits: _the-feature-step1_ and _the-feature-step2_ tracked by the branch _the-feature_.

Therefore, you may type:
    
    git log --all --decorate --oneline --graph

And the outcome may look like this:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_1.png)

You then suddenly realize that you made a mistake on _the-feature-step1_, and you'd like to fix it up, so you make another commit to fix it:
        
    git commit -m "Fix up a mistake at the-feature-step1"

Now, your repository looks like this:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_2.png)

Hang on! This fix-up commit doesn't make sense that much of a sense to the repository's story, it's just part of a previous commit and doesn't tell us anything more about the project than what the previous commit did, it hasn't been pushed into the remote branch and so there's no need to have it ending up into _master_. Instead, we might squash the commit into its fix-up before merging our branch into _master_.

So, what can we do then?

## Fix-up commits and rebase with autosquash to the rescue

First off, rather than manually creating the fix-up commit, let Git do that for us:  
    
    git commit --fixup 9d33fdb

Where _9d33fdb_ is the hash of the commit that you want to fix.

Now, the repository looks like this:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_3.png)

Notice the `fixup!` prefix, that will be used by Git to properly clean up our repository later on.

After you've deemed the feature as done, you decided to push it, so the code review process can start (you do code reviews, right? No? Perhaps you should consider it). It's about time to get rid of the fix-up commit.

That's when interactive rebase + autosquash help us. So, Git will use the `fixup!` prefix to automatically arrange and squash the fix-up commit with the original commit:   
    
    git rebase -i --autosquash master

Where:

  *  `-i` stands for interactive, which will open a text editor before rebasing the commits, so you can influence the rebasing process, should you need it.
  *  `--autosquash` ensures that commits prefixed with `fixup!` (or `squash!`) will be automatically squashed into the commits that they fix.

It'll open your text editor showing what the rebase will look like. Pay special attention to the line starting with `fixup`, it should already be there and in the right order, ready to be squashed with the desired commit (`the-feature-step1`):

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_4.png)

If everything looks correct, and it is in our case, you can then close the text editor.

Now, the repository should look similar to:

![](/assets/img/2018-09-24-git-fixup-commits-and-how-to-keep-your-repository-concise_5.png)

The fix-up commit has been squashed with its original commit before `the-feature` branch was merged into `master`. Just as we intended!

# Using --autosquash by default

Git allows us to go even further by making the process simpler.

Given that interactive rebase only selects commits with messages starting with `fixup!` (or `squash!`) and we always have the possibility of seeing what Git will do with the text editor before rebasing, we can make `autosquash` be the default behaviour of interactive rebases.

For that to happen, we can enable the `rebase.autosquash` flag:
    
    git config --global rebase.autosquash true

Now, we can omit the `autosquash`flag when starting interactive rebases:
    
    git rebase -i master

The result should be the same as we had before.

# Conclusion

Git is a fantastic tool that helps us on our daily programming tasks, making it easier to share work with fellow developers, keeping track of the project's history, etc.

Some of its features are still not so well-known yet, for example, fix-up commits combined with interactive rebase and autosquash, which tidies your repository, hence making it easier to understand, so that we can be amused by the story that it's trying to tell you.


# References

[1] Pro Git Book. <https://git-scm.com/book/en/v2>

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
