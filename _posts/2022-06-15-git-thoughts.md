---
title: "Git thoughts"
subtitle: "Git things I do often enough to write about"
published: true
image_url: "featured/2022-06-15.png"
code_sample: false
---

# whoami

I’m by no means an expert at Git, but as a daily user for the last 5 years let me assure you that you don’t need to know a ton about the working internals to get by day-to-day. I decided I would write this post to call attention to some commands I use most regularly and how I think about them conceptually.

This is not a Git cheatsheet for when you monkey up like [Oh Shit, Git!?!](https://ohshitgit.com/) and it itsn’t really a [best practices recommendation](https://sethrobertson.github.io/GitBestPractices/) either. Just a dude who thinks it’s fun to explain how he thinks about Git.

![A sketch of the git logo from git-scm.com/](/assets/images/posts/2022-06-15/1.png)

# Local and remote

It was my second year of university when I was first required to install Git in order to submit code to a public repository for my professor to evaluate me on an assignment. As someone who was previously accustomed to emailing or uploading word documents for assignments, imagine my confusion when I was required to open a terminal and run commands to submit work. Not to mention the strange things they made me do! Cloning repos, branching, committing changes, pushing them. It felt like a lot of work for something I had previously done in *one step*.

I attended office hours with the university system administrator who helped me to understand the difference between my local repo and the remote repo. Where the files lived and my professors would access them. He also helped me understand how committing created a local log of my work that could help me backtrack and undo changes, like the history feature in Google Docs.

So my first tip for Git is to think about what actions apply to what Git is tracking on your local file system, and what will make changes on the remote**.** For example

- `git commit` creates a commit including your staged changes *on local only*
- `git push` takes your new commit history and pushes it *to the remote*
- `git fetch` brings in new branches and branchs’ update histories from the remote *to your local*

As an aside— once I was working on code professionally I eventually found [a talk from Linus Torvalds at Google about Git](https://www.youtube.com/watch?v=4XpnKHJAok8). At that point I had been working with Git regularly for over a year, but the video opened my mind to the idea of having no central location for source code, just shared history pushed and pulled from individual nodes in a network. This isn’t how any company I have worked for manages source code.

# Staging and stashing

Another thing that confused me when I was trying to submit assignments for my algorithms class was why I had to `git add` *and* `git commit` before pushing my changes to the remote.

It’s worthwhile taking the time to be intentional while you are staging changes, and Git sets you up to do that well. Selecting which changes you want in which commit, and what changes you want on your local at any time, is a lot easier to do using Git than it is to try to do, undo, and redo changes manually.

- First of all, use `git status` to see what changes you have and what’s staged
- `git add` files by directories, like `git add /dir` instead of always `git add .`
- `git rm --cached file` will remove a staged file
- Once you have committed just what you want, `git stash` is a quick way to throw out the other changes
- `git stash pop` restores the most recently thrown out changes

I don’t use `git stash` in a sophisticated way very often, but a colleague recently introduced me to the concept of “[Git stash refactoring](https://www.dotnetcodegeeks.com/2016/01/git-stash-driven-development.html)” as a kind of *thread management* technique for when you need to do a refactor that isn’t related to the task at hand. I find this comes up for me a lot in Test Driven Development and I could probably use it better.

One nice way to stage your changes selectively while looking at what you change is `git add --patch`, a [command](https://nuclearsquid.com/writings/git-add/) which goes through hunks of files you have changed one by one and asks you how you would like to handle the set of changes.

# Merging vs. rebasing

This wasn’t something I learned in my first two years of Git, but later. For the first two years, I relied a lot on `git pull origin {branch}`. I used this to do a simple thing. `branch` has code on it I want on my branch. So, this command:

1. Fetches the remote `branch` with the changes I want
2. Merges my new local copy of `branch` into my current working branch
3. Voila! Gimme that code 🙂

But what’s this? It also pops up an annoying vi window for you to write a sexy lil’ merge commit message. Don’t worry, nobody will be offended by the default message, they are used to seeing it.

There is another way to get code from another branch onto your working branch called [rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing). 

`git rebase {branch}`

This command plops the history of your working branch onto `branch`. Magic! Well, almost. Here are some times it’s not magic:

- You have tons of commits - Rebasing rewrites history for each commit as if it were applied one by one to `branch`, not to the working branch’s history. If there are conflicts, you resolve them per commit. So if you have been making tons of itsy bitsy commits in the fame files, you may have to resolve similar conflicts over and over before you can `rebase --continue`
- You have to force push - Rewriting history means you have to push that new version of history to the central location. I don’t think Linus Torvalds would approve, but if you can convince your team to always `pull -r` and you are careful, it works pretty well.

![Two documents with different edits](/assets/images/posts/2022-06-15/2.png)

# Merge conflicts

Speaking of merging, a merge conflict is Git’s way of knowing it can’t decide what merged histories should look like without your help. It writes scary lines into your code and tells you it can’t continue the merge until they are resolved. Eek!

My only thoughts on this are generally this happens less when your tream is organized about closing [feature branches](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) quickly or when you all agree to do [trunk based development](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) and rebase as much as possible (see Merging vs. Rebasing below).

If you have to deal with merge conflicts, use an IDE that is good at it. I like the [Jetbrains IDEs](https://www.jetbrains.com/help/idea/resolving-conflicts.html), and I think [VScode](https://linuxpip.org/fix-merge-conflicts-vscode/) lets you do it inline.

# Amending, squashing, and interactive rebase

On the note of rebasing and having too many commits, I went on way too long living with an ugly commit history because I was too lazy/embarrassed to learn better, but also way sloppy to keep things tidy inherently. This section of tips are some things I do to try to stay neat.

`git commit --amend` updates your most recent commit with newly staged changes if you missed something, it also opens up a vi window for another saucy addition to your commit message (if you wrote the perfect commit message the first time, `git commit --amend --no-edit`).

Squashing commits is a nice way to take a bunch of teeny-tiny commits and wrap ‘em up in one big-ol’ commit. So if you like to commit a lot along the way, but you don’t want your coworkers to judge you, squash it! Again, [Jetbrains IDEs](https://www.jetbrains.com/help/idea/edit-project-history.html) let you do this in a way that is probably too easy to be good for you. I worked for some time at a company that had a policy to squash on merge to the `main` branch. That meant one commit per delivered feature, plus you could dig into the commit for all of the smaller commit information. 

If using the IDE to squash for you isn’t hard core enough, I feel you. The purists out there will know about [interactive rebasing](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History). This allows you to edit history on a bunch of commits at once. So say you left a mess of 15 icky commits (ya nasty!), `git rebase -i HEAD~15` will open up (you guessed it) a cheeky little vi window with a log of your commits and some editable actions to be run against them. Come here to edit messages, squash commits, and more all at once.

It’s worth that you can’t squash regular commits into merge commits, yet another way merge commits make it harder to keep things tidy.

# Aliases

Next hot tip to level up your git status (no, not [that one](https://git-scm.com/docs/git-status)!) - add a bunch of [aliases](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases) to your Git to confuse your pair (if you [pair program](https://martinfowler.com/articles/on-pair-programming.html)) and make you look fast. I’m kidding, I think these can actually save time if you aren’t a curmudgeon like me who built habits around typing out the full commands and doesn’t want to change.

You can set aliases in your global Git config `$HOME/.gitconfig` like this:

```
[alias]
 lgmt = !git add . && git commit -m "who reads these?" && git push
```

One fun outcome of this is you start an internal monologue with a new set of vocabulary “let me `gri` (`git rebase -i`) that real quick”. It also translates hilariously to dialogue with a pair. I worked for a few months with a colleague who had me asking her to `gpsupf` (pronounce gip-sup-fuh) as a reminder to `git push --force-with-lease`.

That’s all for now.