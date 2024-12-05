---
title: "Git workflow"
date: 2023-11-15T12:00:00+00:00
# weight: 1
# aliases: ["/first"]
tags: ["git", "workflow"]
author: "Jozef Cajkovic, [Eduard Bargues](https://github.com/lucky-edu)"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/jcajkovic.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


The post about my git workflow.

In an organisation I work for, there are some rules how the git commit should look like. A lot comes from general conventional commits standard to support automatic versioning of merged code stacked with rules like mandatory Jira ticket in every single commit message. All this paired with automatic creation of CHANGELOG.md file in the repository.

Today I would like to show you my flow working with the git and fixing or integrating new features into our code base.

It is important to mention I work mainly on Platform development, like terraform modules for AWS, GitHub, Azure AD, PagerDuty and similar. Important part of my work are GitHub workflows and actions our team make available for the rest of DevOps teams.

Let's get into it.

**Checkout**

First when I start to work on new feature I will naturally create the git branch in repository I work in. For this task I have defined following git alias it .gitconfig file:

```toml
co-from = "!f(){ git stash; git checkout $1; git fetch; git reset --hard origin/$1; git checkout -b $2-$3; git stash pop; }; f"
```

The usage of this alias is as follows:

```bash
git co-from main PIXEL-101 fix-nasty-bug
```

The output of this command is new branch which respects all the new changes in *main* branch and the name of the branch is *PIXEL-101-fix-nasty-bug*, not only that all the changes I have done in the code were stashed and than popped out of the stash after the pull of newest changes from origin were done.

**Push**

After I'm satisfied with all the changes I've done into the code I need to push everything to the branch and more importantly to the remote repository in the form organisation expect me to for code security analysis and for continuous integration workflows to process the code.

```toml
cm-rb-puu = "!f(){ git add .; git commit -m \"$1($2): $3 $4\"; git fetch; export GIT_EDITOR=true; git rebase -i --autosquash origin/$5; git push -f -u origin \"$(git branch --show-current)\"; }; f"
```

Usage:

```bash
git cm-rb-puu fix 'src' PIXEL-101 'the nasty bug is fixed by a humble developer with help of github copilot'
```

This command in fact squash and push all the changes I did and of course it will create a link for me to follow up on to create GitHub Pull Request. Something like this:

```bash
On branch PIXEL-101-fix-nasty-bug
nothing to commit, working tree clean
fatal: invalid upstream 'origin/'
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'PIXEL-101-fix-nasty-bug' on GitHub by visiting:
remote:      https://github.com/pixel-systems/example-repository/pull/new/PIXEL-101-fix-nasty-bug
remote:
To github.com:pixel-systems/example-repository.git
 * [new branch]      PIXEL-101-fix-nasty-bug -> PIXEL-101-fix-nasty-bug
branch 'PIXEL-101-fix-nasty-bug' set up to track 'origin/PIXEL-101-fix-nasty-bug'.
```

the Pull Request in GitHub will look like this:
![[Pasted image 20231115135008.png]]

**Fixing**

Of course the problems will rise when you will commit faulty code. There I have two options. Go with the previous cycle until my code is fixed and will pass all mandatory checks or I will do some sort of FixUp.

```
fu-rb-puu = "!f(){ git add .; git commit --fixup \"$(git rev-parse HEAD)\"; git fetch; export GIT_EDITOR=true; git rebase -i --autosquash origin/$1; git push -f -u origin \"$(git branch --show-current)\"; }; f"
```

I prefer to do git --fixup with respect to the changes done in main branch, this will not create a mess in my git history so the PR than looks very clean for my colleagues to review and it looks like I do not do mistakes ;-). 

Usage of this alias is:

```
git fu-rb-puu main
```

and of course the last part if I need to rebase this is the command to be used:

```
rb = "!f(){ git fetch; export GIT_EDITOR=true; git rebase -i --autosquash origin/$1; }; f"
```

usage:

```
git rb main
```

the rest of the my .gitconfig looks pretty boring but it is integral part of the whole workflow:

```toml
[core]
	editor = code --wait

[filter "lfs"]
	required = true
	clean = git-lfs clean -- %f
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process

[diff]
	tool = vscode-difftool
	
[difftool "vscode-diftool"]
	cmd = code --wait -d $LOCAL $REMOTE

[merge]
	tool = vscode-mergetool

[mergetool "vscode-mergetool"]
	cmd = code --wait $MERGED

[color]
	ui = true

[color "status"]
	changed = yellow bold
	untracked = red bold
	added = green bold

[help]
	autocorrect = 20
```
**Disclaimer:**

All this I've learned from my former colleague [Eduard Bargués Junyent](https://www.linkedin.com/in/eduardbargues), check his [github](https://github.com/lucky-edu) too.
