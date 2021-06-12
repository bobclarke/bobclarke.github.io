---
layout: post
title: 
date: 2021-06-11 13:05
category: 
author: 
tags: []
summary: 
---

Let's say I'm contributing to Github projects at work and I'm also contributing to Github projects that are not work related. In all likelyhood I'll do this using different Github accounts, however I may be working on the same machine for both work and personal projects for convenience. With this in mind, I need a way for my git cli to automatically detect which SSH key to use depending on the project.

For the pusposes of this example we'll use the following fake Github projects
```
https://github.com/my-work-org/my-work-project
https://github.com/my-personal-org/my-perosnal-project
```

For these projects, I'm using 2 different SSH keys, one for each project
```
/Users/clarkeb/.ssh/id_rsa_work.pub
/Users/clarkeb/.ssh/id_rsa_perosnal.pub
```

To ensure that my git cli uses the right key, I have to do two things...

First, I setup mhy  ~/.ssh/config like this:
```
Host github.com-work
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_work
  IdentitiesOnly yes
    StrictHostKeyChecking=no
    UserKnownHostsFile=/dev/null

Host github.com-personal
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_personal
  IdentitiesOnly yes
    StrictHostKeyChecking=no
    UserKnownHostsFile=/dev/null
```
This basically says... for git remotes with a host alias of This basically says... 
* for git remotes with a host alias of github.com-work, the git cli should use the ~/.ssh/id_rsa_work SSH private key
* for git remotes with a host alias of github.com-personal, the git cli should use the ~/.ssh/id_rsa_personal SSH private key

So what's a git remote alias?... You can find this out by running **git remote -v** in any local git directory. For example 
```
git remote -v
origin  git@github.com-personal:bobclarke/bobclarke.github.io.git (fetch)
origin  git@github.com-perosnal:bobclarke/bobclarke.github.io.git (push)

```

This (for me) was a bit confusing at first because the **github.com-personal** part wasn't a proper hostname. Then I relaised it didn't need to be, it just needed to match the **Host** field in ~/.ssh/config, where it is matched to the actuall Github hostname with the **HostName** field. 




