---
layout: post
title: Synchronisation d'un fork avec git
description: Comment synchroniser un dépôt git après un fork.
photo: 
categories: articles
tags: [git]
comments: true
---

Je viens de mettre à jour le thême du blog qui utilise [So Simple Theme](https://github.com/mmistakes/so-simple-theme) mais ayant fait quelques modifications il a fallu que je synchronise mon dépôt avec le dépôt original. Pour cela j'ai suivi un tuto créé par Github très clair.

J'en ai fait un article plus pour m'en rappeler que pour autre chose car même un anglophobe ne devrait pas avoir de mal à suivre [l'article original](https://help.github.com/articles/syncing-a-fork) ;)


## Mise en place ##

Avant de pouvoir sychroniser notre dépôt il faut ajouter le remote du dépôt original.

    $git remote -v
    # Liste les remotes actuels
    # origin  https://github.com/user/repo.git (fetch)
    # origin  https://github.com/user/repo.git (push)
    
    $git remote add upstream https://github.com/otheruser/repo.git
    # Ajout d'un nouveau remote
    
    $git remote -v
    # Vérification de l'ajout du nouveau remote
    # origin    https://github.com/user/repo.git (fetch)
    # origin    https://github.com/user/repo.git (push)
    # upstream  https://github.com/otheruser/repo.git (fetch)
    # upstream  https://github.com/otheruser/repo.git (push)

## Synchronisation ##

Deux étapes sont nécessaires afin de synchroniser votre dépôt avec le dépôt original: dans un premier temps il va falloir récupérer les nouvelles modifications (fetch), ensuite il faut fusionner (merge) les deux dépôts. 

### Récupération ###

    $git fetch upstream
    # Grab the upstream remote's branches
    # remote: Counting objects: 75, done.
    # remote: Compressing objects: 100% (53/53), done.
    # remote: Total 62 (delta 27), reused 44 (delta 9)
    # Unpacking objects: 100% (62/62), done.
    # From https://github.com/otheruser/repo
    #  * [new branch]      master     -> upstream/master

On a désormais la branche master distante stockée dans une branche locale, `upstream/master`.

    $git branch -va
    # List all local and remote-tracking branches
    # * master                  a422352 My local commit
    #   remotes/origin/HEAD     -> origin/master
    #   remotes/origin/master   a422352 My local commit
    #   remotes/upstream/master 5fdff0f Some upstream commit

### Fusion ###

Maintenant que l'on a récupéré les modifications on veux désormais fusionner les changements avec notre branche locale. 

    $git checkout master
    # Check out our local master branch
    # Switched to branch 'master'

    $git merge upstream/master
    # Merge upstream's master into our own
    # Updating a422352..5fdff0f
    # Fast-forward
    #  README                    |    9 -------
    #  README.md                 |    7 ++++++
    #  2 files changed, 7 insertions(+), 9 deletions(-)
    #  delete mode 100644 README
    #  create mode 100644 README.md

Si votre dépôt ne contenait pas de nouvelles modifications alors git va effectuer un "fast-forward":

    $git merge upstream/master
    # Updating 34e91da..16c56ad
    # Fast-forward
    #  README.md                 |    5 +++--
    #  1 file changed, 3 insertions(+), 2 deletions(-)

Et voilà c'était pas si compliqué :D (le truc qui peut par contre être emmerdant c'est la résolution des potentiels conflits...)
