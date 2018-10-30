---
layout: post
title: Manage local gits with repo
date: 2018-10-30
categories: 
- git repo
tags: 
- git
author: Kai Fan
excerpt: Try to manage my local git repos with git-repo tool
---


# Would like to use repo becasue of Uh&#x2026; Vim 8.0

There are at least two copy of VIMs lives in my laptop. One was
manually installed gvim 8.1, the other is bundled vim by msysgit.

As of vim 8.x there's a new package management system released. One
may manage all vim plgin package without any thirdpart plugin
manager. As described in this famous [blog](https://shapeshed.com/vim-packages/) post.

    git submodule add https://github.com/vim-airline/vim-airline.git pack/shapeshed/start/vim-airline # 
    git add .gitmodules pack/shapeshed/start/vim-airline

Anyway, I have no confidence in the git submodule command. Also the
solution try to initial one git repo group for each dot vim. I don't
like to iterate over all dot vim directories one by one when doing
plugin updates.

Alternative way is to make use of git-repo tool to handle all the git
repositories (vim plugins) as a whole&#x2026;


# Repo init

Error executing repo init&#x2026; Because of msys path and win32 native
path conflict. As my python interpreter comes with emacs, which is
compiled to work as native programe. And its os.path.expanduser api
will return windows native path.

    $ repo init -m workspace/xml/manifest.xml
    gpg: keyblock resource '/c/opt/ekaifan/C:\opt\ekaifan/.repoconfig/gnupg/pubring.kbx': No such file or directory
    gpg: no writable keyring found: Not found
    gpg: error reading '[stdin]': General error
    gpg: import from '[stdin]' failed: General error
    gpg: Total number processed: 0
    fatal: registering repo maintainer keys failed


# Repo manifest file and local manifest

Manifest file is the configuration(repository information database)
for git-repo tool. It is in xml format, DTD can be found in its
offical document.

Refer to [Learn about the repo tool , manifests and local manifests and
5 important tips ! by Red Devil](https://forum.xda-developers.com/showthread.php?t=2329228)

Also the [offical git-repo document](https://gerrit.googlesource.com/git-repo/+/HEAD/docs/manifest-format.md)


## The XML and DTD and nxml

Lesson learn from DTD to rng converting:

1.  The \`<!DOCTYPE foobar [' tag is for inline DTD, for external
    DTDs this tag along with its conterpart \`]>' should be removed
    for validity.
2.  XSD to RNG,RNC,DTD converting is almost impossible, as XSD is
    too strict comparing with other xml validating format.


## convert DTD to RNG with jing-trang

    cd ~/workspace
    mkdir xml
    cat > repo-manifest.dtd
    # comment out first and last line in repo-manifest.dtd
    # to form a valid external dtd file


# To-Be-Continued

The repo tool is not offically supported on windows. And I find the
issue of my environment is also in [esrlabs clone](https://github.com/esrlabs/git-repo).

I don't have too much time working on repo tool, leave it there. 

Would like to see if [depottools](https://dev.chromium.org/developers/how-tos/depottools) gclient can helps.

