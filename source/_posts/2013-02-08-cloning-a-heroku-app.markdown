---
layout: post
title: "Cloning a Heroku app with git"
date: 2013-02-08 14:30
comments: true
categories: heroku git git-clone
---

Today I wanted to change a few things on my last post, but wasn't on my main computer, so I tried to clone my blog, hosted in Heroku, to make these changes.

I thought *hey, I'll certainly do this again in the future and I don't want to forget how to do it*, so here it is for posterity.

First of all, if you have never used Heroku in the computer you are in, you have to 

    gem install heroku
{:lang="text"}

Then, you have to input your Heroku credentials, and add your computer's key to your Heroku account :

    heroku login
    heroku keys:add [your key file]
{:lang="text"}

The computer's key is usually `~/.ssh/id_rsa.pub`. If you don't have a key, you can generate one using the command

    ssh-keygen
{:lang="text"}

Finally, to clone your repo, you use

    git clone git@heroku.com:my-awesome-repo.git my-directory
{:lang="text"}

That's it!
